# 🔌 Guia de Conexão com o WhatsApp

A Unoapi Cloud oferece flexibilidade para se conectar ao WhatsApp de duas maneiras principais. A escolha depende do seu caso de uso, escala e se você possui uma conta na Plataforma do WhatsApp Business.

---

## Opção 1: Conexão via WhatsApp Web (Comunidade)

Esta abordagem utiliza a biblioteca **[Baileys](https://github.com/adiwajshing/Baileys)**, que automatiza o WhatsApp Web em um servidor. É ideal para projetos pessoais, testes, e aplicações de pequeno a médio porte.

### Como Funciona?

A API cria uma sessão que se conecta ao WhatsApp da mesma forma que você faria no seu navegador. A autenticação é feita escaneando um QR Code gerado pela API.

### ✔️ Vantagens
- **Gratuito:** Não há custos diretos com o WhatsApp.
- **Rápido de configurar:** Você só precisa de um número de WhatsApp funcional.
- **Flexível:** Permite o uso de contas pessoais do WhatsApp.

### ❌ Desvantagens
- **Menos estável:** A conexão depende do seu celular estar online e conectado à internet.
- **Risco de bloqueio:** O uso de automação em contas pessoais viola os termos de serviço do WhatsApp e pode levar ao bloqueio do número. **Use por sua conta e risco.**
- **Sem suporte oficial:** Por não ser uma solução oficial, não há garantia de funcionamento contínuo.

### Passo a Passo para Conectar

1.  **Inicie a API:** Certifique-se de que sua instância da Unoapi Cloud esteja em execução (`npm run dev`).

2.  **Acesse a Rota de Conexão:** Abra seu navegador e acesse a rota `/connect/{phone}`, substituindo `{phone}` pelo número que você deseja conectar (ex: `http://localhost:3000/connect/5511999999999`).

3.  **Escaneie o QR Code:** Uma página com um QR Code será exibida.
    - No seu celular, abra o WhatsApp.
    - Vá para **Configurações > Aparelhos Conectados > Conectar um Aparelho**.
    - Escaneie o QR Code exibido no navegador.

4.  **Conexão Estabelecida:** Após escanear, a sessão será salva e a API estará pronta para enviar e receber mensagens através deste número.

### 🤯 Troubleshooting Comum

- **QR Code não aparece ou expira rápido:** Tente atualizar a página. Se o problema persistir, verifique o console da API por erros.
- **Desconectando com frequência:** Garanta que seu celular tenha uma conexão estável com a internet e que o modo de economia de bateria esteja desativado para o WhatsApp.

---

## Opção 2: Conexão via API Oficial do WhatsApp Business

Esta é a solução oficial e robusta, projetada para médias e grandes empresas que precisam de escalabilidade, estabilidade e suporte.

### Como Funciona?

Você configura a Unoapi Cloud para atuar como um *wrapper* para a **Plataforma Cloud da Meta**. Em vez de gerar um QR Code, a API encaminha as requisições para os servidores da Meta, usando as credenciais da sua conta Business.

### ✔️ Vantagens
- **Oficial e estável:** Solução suportada pela Meta, garantindo alta disponibilidade.
- **Escalável:** Projetada para um grande volume de mensagens.
- **Seguro:** Sem risco de bloqueio por violação dos termos de serviço.
- **Recursos avançados:** Acesso a templates de mensagem, análises e outros recursos de negócios.

### ❌ Desvantagens
- **Custos:** A Meta cobra por conversas. Consulte os [preços oficiais](https://developers.facebook.com/docs/whatsapp/pricing).
- **Processo de aprovação:** Requer a criação de uma conta no Gerenciador de Negócios da Meta e a verificação da empresa.
- **Restrições:** O envio de mensagens é limitado a templates pré-aprovados, a menos que o cliente inicie a conversa.

### Passo a Passo para Configurar

1.  **Crie uma Conta na Plataforma da Meta:**
    - Siga o [guia oficial da Meta](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started) para criar sua conta, configurar um número de teste e obter suas credenciais.

2.  **Configure as Variáveis de Ambiente:**
    - No arquivo `.env` da sua Unoapi Cloud, configure as variáveis relacionadas à API da Meta:
      - `GRAPH_API_VERSION`: A versão da API da Graph (ex: `v18.0`).
      - `ACCESS_TOKEN`: O token de acesso permanente gerado na sua conta da Meta.
      - `PHONE_NUMBER_ID`: O ID do número de telefone que você configurou na plataforma.

3.  **Configure o Webhook:**
    - Na sua conta da Meta, você precisará configurar um webhook para receber mensagens e atualizações.
    - A URL do webhook será a rota `/webhooks/whatsapp/{phone}` da sua API (ex: `https://sua-api.com/webhooks/whatsapp/5511999999999`).
    - Você também precisará de um **Token de Verificação**, que pode ser qualquer string segura que você definir. Configure-a na variável `VERIFY_TOKEN` no seu arquivo `.env`.

4.  **Use os Endpoints da API:**
    - Com a configuração concluída, você pode usar os endpoints da Unoapi Cloud (como `/messages`) que, por sua vez, se comunicarão com a API oficial da Meta.