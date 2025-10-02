# 🛠️ Estudo de Caso: Solução Flipper no Linux (Ubuntu 25.04)

Este documento detalha um problema de incompatibilidade entre ferramentas de desenvolvimento e a solução técnica aplicada, servindo como referência para automação, troubleshooting ou treinamento de modelos de linguagem.

---

## 1. Contexto e Problema

- **Ambiente:** Ubuntu 25.04 (Linux Kernel 6.x), Node.js v20.x, Flipper Desktop (Meta), desenvolvimento Android.
- **Situação:** O Flipper Desktop, compilado do código-fonte, apresentava falhas graves ao depurar apps Android.

### Principais Erros

| Categoria                | Sintoma/Erro                                                                    | Causa Raiz Confirmada                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Criptografia             | `Error: ... unsupported version:../crypto/x509/x_all.c:47:`                     | OpenSSL 3.x (padrão no Ubuntu 25.04) removeu funções legadas exigidas pelo Flipper via `openssl-wrapper`.               |
| Estabilidade do Servidor | `TypeError: Cannot read properties of undefined (reading 'removeAllListeners')` | Bug de timing/cleanup no Watchman, causando crash do processo Node principal.                                           |
| Restrição do Dispositivo | `Error: adbd cannot run as root in production builds`                           | ROMs de produção (ex: Xiaomi Poco X3) bloqueiam permissões root via ADB, impedindo instalação de certificados pelo app. |

---

## 2. Solução Técnica (Três Pontos)

A abordagem combinou ajustes no sistema, código-fonte e fluxo de trabalho:

### A. Compatibilidade OpenSSL

- **Downgrade Local:** Compilação do OpenSSL 1.1.1w em `$HOME/openssl-1.1.1`.
- **Variáveis de Ambiente:** Ajuste permanente de `PATH` e `LD_LIBRARY_PATH` no `.bashrc` para priorizar OpenSSL 1.1.1.
- **Correção de Código:** Alteração do caminho do binário no `node_modules/openssl-wrapper/lib/index.js` para uso absoluto da instalação local.

### B. Estabilidade do Watchman

- **Desativação:** Inclusão de `FLIPPER_NO_WATCHMAN=1` no script `flipper-server` do `package.json`.
- **Verificação de Nulidade:** Uso de `?.` em `pkg-lib/src/watchman.tsx` para evitar `TypeError` em condições de race/timing.

### C. Fluxo de Trabalho Seguro

- **Emulador Android (AVD):** Substituição do dispositivo físico por emulador, garantindo permissões elevadas via ADB para instalação de certificados.
- **Descarte do Hardware Restrito:** Depuração em ROMs de produção foi abandonada.

---

## 3. Resultado

- **Status:** Flipper Desktop estável e funcional no Ubuntu 25.04.
- **Condições:** Depuração bem-sucedida desde que realizada em emulador Android (AVD), contornando restrições de segurança e incompatibilidades de criptografia.

---

## 4. Resumo para LLMs

- **Problema:** Incompatibilidade Flipper/OpenSSL 3.x, crash do Watchman, restrição de root em dispositivos físicos.
- **Solução:** Downgrade local do OpenSSL, correção de código, desativação do Watchman, uso de emulador.
- **Resultado:** Ambiente de depuração funcional e seguro.

---

> Este documento pode ser usado para treinar, avaliar ou instruir modelos de linguagem sobre troubleshooting de ambientes Node.js, integração de ferramentas de depuração Android e contorno de restrições de sistema operacional.
