# 🛠️ Guia Definitivo: Flipper no Ubuntu 25.04 (Correção OpenSSL 3.x)

Este guia resolve a incompatibilidade crítica entre o **Flipper** e o **OpenSSL 3.x** no Ubuntu 25.04, além de corrigir o erro de travamento do **Watchman**.

> **Atenção:** A conexão com dispositivos de produção (ex: Xiaomi Poco X3) pode falhar devido à restrição de root. Após seguir todos os passos, utilize um **Emulador Android (AVD)** para garantir funcionamento total.

---

## 1. Instalação e Configuração do OpenSSL 1.1.1 (Desktop)

Essencial para resolver erros de certificado.

1. **Instale dependências e compile o OpenSSL 1.1.1w:**

   ```bash
   sudo apt update
   sudo apt install build-essential checkinstall zlib1g-dev

   wget https://www.openssl.org/source/openssl-1.1.1w.tar.gz
   tar -zxf openssl-1.1.1w.tar.gz
   cd openssl-1.1.1w

   # Use $HOME para garantir o caminho correto
   ./config --prefix=$HOME/openssl-1.1.1 shared
   make
   sudo make install
   ```

2. **Configure as variáveis de ambiente (`~/.bashrc`):**
   Adicione ao final do arquivo:

   ```bash
   # Corrige compatibilidade Flipper/OpenSSL 3.x
   export SSL_ROOT=$HOME/openssl-1.1.1/bin
   export SSL_LIB=$HOME/openssl-1.1.1/lib

   export PATH=$SSL_ROOT:$PATH
   export LD_LIBRARY_PATH=$SSL_LIB:$LD_LIBRARY_PATH
   ```

   Após salvar, execute:

   ```bash
   source ~/.bashrc
   ```

---

## 2. Modificações no Flipper e Watchman

Essas alterações forçam o uso do OpenSSL correto e evitam o crash do Watchman.

1. **Clone o Flipper e instale dependências:**

   ```bash
   git clone https://github.com/facebook/flipper.git
   cd flipper/desktop
   yarn install
   ```

2. **Corrija a chamada do OpenSSL (`openssl-wrapper`):**
   Edite `./node_modules/openssl-wrapper/lib/index.js` (linha ~76) e altere:

   ```javascript
   // Troque 'openssl' pelo caminho absoluto:
   var openssl = (0, _child_process.spawn)(
     "/home/SEU_USUARIO/openssl-1.1.1/bin/openssl",
     params
   );
   ```

   > Substitua `SEU_USUARIO` pelo seu nome de usuário.

3. **Corrija o crash do Watchman (`watchman.tsx`):**
   Edite `./pkg-lib/src/watchman.tsx` (linha ~33):

   ```typescript
   const onError = (err: Error) => {
     // Adicione verificação de nulidade
     this.client?.removeAllListeners("error");
     reject(err);
     this.client?.end();
     delete this.client;
   };
   ```

---

## 3. Conexão

1. **Execute o Flipper e conecte ao AVD:**
   Inicie o Flipper e seu app Android em um **Emulador (AVD)** para garantir instalação do certificado sem restrições de segurança:
   ```bash
   yarn start
   ```

---

## Dicas Finais

- Sempre utilize o emulador para testes, evitando limitações de dispositivos físicos.
- Se encontrar erros, verifique se as variáveis de ambiente estão corretas e se o caminho do OpenSSL está ajustado para seu usuário.

---
