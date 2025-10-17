# 🤯 Guia de Solução de Problemas (Troubleshooting)

Este guia centraliza os erros e problemas mais comuns que você pode encontrar ao instalar, configurar e usar a Unoapi Cloud, junto com suas possíveis soluções.

---

## 🔧 Problemas de Instalação

### 1. Erro `ERESOLVE` ou Conflito de Dependências no `npm install`

- **Sintoma:** O comando `npm install` falha com uma mensagem `ERESOLVE` ou sobre `peer dependency conflict`.
- **Causa:** Algumas dependências do projeto têm requisitos de versão que entram em conflito entre si.
- **Solução:** Use a flag `--legacy-peer-deps` para que o npm ignore esses conflitos e use a versão da dependência que está definida no `package.json`.
  ```bash
  npm install --legacy-peer-deps
  ```

### 2. Erro `node-gyp` durante a instalação

- **Sintoma:** A instalação falha com erros relacionados ao `node-gyp`.
- **Causa:** O `node-gyp` é uma ferramenta que compila módulos nativos do Node.js. O erro significa que as ferramentas de compilação necessárias (como compiladores de C++) não estão instaladas no seu sistema.
- **Solução:**
    - **Windows:** Abra o PowerShell como Administrador e execute:
      ```bash
      npm install --global --production windows-build-tools
      ```
    - **macOS:** Instale as ferramentas de linha de comando do Xcode:
      ```bash
      xcode-select --install
      ```
    - **Linux (Debian/Ubuntu):** Instale o pacote `build-essential`:
      ```bash
      sudo apt-get install -y build-essential
      ```

---

## 🔌 Problemas de Conexão (WhatsApp Web / Baileys)

### 1. QR Code Não Carrega ou Expira Muito Rápido

- **Sintoma:** Ao acessar a rota `/connect/{phone}`, o QR Code não aparece, ou desaparece antes que você consiga escaneá-lo.
- **Causa:** Pode ser um problema de performance do servidor onde a API está rodando ou uma instabilidade na conexão com os servidores do WhatsApp.
- **Soluções:**
    1.  **Atualize a Página:** A primeira tentativa é simplesmente atualizar a página (`F5` ou `Cmd+R`).
    2.  **Verifique os Logs:** Verifique o console onde a API está rodando (`npm run dev` ou `docker-compose logs -f`) para ver se há alguma mensagem de erro.
    3.  **Use o Terminal:** Em alguns casos, o QR Code pode ser exibido diretamente no terminal se a renderização no navegador falhar. Verifique os logs da aplicação.

### 2. Desconexões Frequentes da Sessão

- **Sintoma:** A API funciona por um tempo, mas depois a sessão conectada "cai" e para de responder.
- **Causa:** A conexão via Baileys depende 100% do seu celular estar conectado à internet e com o WhatsApp ativo.
- **Soluções:**
    - **Conexão Estável:** Garanta que seu celular esteja conectado a uma rede Wi-Fi estável.
    - **Economia de Bateria:** Desative o modo de economia de bateria para o aplicativo do WhatsApp no seu celular. Muitos sistemas operacionais (Android e iOS) limitam a atividade em segundo plano de aplicativos para economizar energia, o que pode desconectar a sessão.
    - **WhatsApp Aberto:** Mantenha o aplicativo do WhatsApp aberto em segundo plano no seu celular.

### 3. Risco de Bloqueio do Número

- **Sintoma:** Seu número do WhatsApp é banido ou bloqueado.
- **Causa:** A automação de contas pessoais viola os Termos de Serviço do WhatsApp. O uso de bibliotecas como o Baileys carrega um risco inerente de bloqueio.
- **Solução:**
    - **Use com Moderação:** Evite enviar spam ou um volume muito alto de mensagens em um curto período.
    - **"Aqueça" o Número:** Se o número for novo, use-o manualmente por um tempo antes de conectá-lo à automação.
    - **Use a API Oficial:** Para casos de uso comerciais, críticos ou de larga escala, a única solução 100% segura é usar a **API Oficial do WhatsApp Business**.

---

## ⚙️ Erros em Tempo de Execução

### 1. Erro `DecryptError` nos Logs ou nos Testes

- **Sintoma:** Aparecem mensagens de erro contendo `DecryptError` ou "The message could not be read" nos logs da aplicação, especialmente ao receber mensagens.
- **Causa:** Este erro geralmente acontece quando a API recebe uma mensagem do WhatsApp que ela não consegue descriptografar. Isso pode ocorrer por vários motivos, como uma dessincronização das chaves de criptografia entre a sua sessão e os servidores do WhatsApp.
- **Solução:**
    - **Reinicie a Sessão:** A solução mais comum é desconectar e reconectar a sessão. Delete o arquivo da sessão (se estiver usando `STORE_TYPE=file`) e gere um novo QR Code para conectar novamente.
    - **Ignore o Erro:** Em muitos casos, este erro é temporário e a própria biblioteca Baileys consegue se recuperar. Se a API continuar funcionando para outras mensagens, você pode monitorar o erro, mas nenhuma ação imediata pode ser necessária.
    - **Atualize as Dependências:** Certifique-se de que você está usando a versão mais recente do projeto, pois correções para esses problemas são frequentemente lançadas.