#  Unoapi Cloud  unofficial

![Unoapi Cloud](https://unoapi.cloud/assets/images/logo.png)

**Um gateway de API de código aberto para o WhatsApp, permitindo a conexão tanto pela API oficial da Meta quanto pela abordagem da comunidade (WhatsApp Web).**

---

## 🇧🇷 Bem-vindo à Documentação em Português!

Esta documentação foi criada para fornecer um guia completo e acessível para desenvolvedores brasileiros que desejam integrar o WhatsApp em suas aplicações usando a Unoapi Cloud.

## 📚 Estrutura da Documentação

Para facilitar a navegação, a documentação foi dividida em vários arquivos:

- **[🚀 Guia de Instalação](./INSTALACAO.md)**: Passo a passo detalhado para configurar o projeto em seu ambiente local.
- **[🔌 Guia de Conexão com o WhatsApp](./CONEXAO_WHATSAPP.md)**: Explica como conectar seu número usando a API da Comunidade (QR Code) ou a API Oficial da Meta.
- **[📖 Referência Completa de Endpoints](./ENDPOINTS.md)**: Detalhes técnicos de cada endpoint da API, com exemplos de requisição e resposta.
- **[💡 Exemplos Práticos de Uso](./EXEMPLOS.md)**: Receitas de código para cenários comuns, como enviar mensagens de texto, imagens e usar botões.
- **[📝 Análise de Lacunas](./ANALISE_LACUNAS.md)**: Relatório sobre os endpoints que foram implementados mas não documentados anteriormente.
- **[📄 Especificação OpenAPI](./openapi.yaml)**: Arquivo `openapi.yaml` para integração com ferramentas de API como Postman e Swagger UI.

---

## ⚡ Início Rápido

### 1. Instalação
Siga o **[Guia de Instalação](./INSTALACAO.md)** para deixar o ambiente pronto.

### 2. Conexão
Escolha seu método de conexão preferido no **[Guia de Conexão](./CONEXAO_WHATSAPP.md)**. Para um teste rápido, a conexão via QR Code é a mais simples.

### 3. Envie sua Primeira Mensagem
Depois de conectar um número, use o endpoint de envio de mensagens para testar.

**Exemplo com `curl`:**
```bash
curl -X POST \
  'http://localhost:3000/v1/SEU_NUMERO_REMETENTE/messages' \
  -H 'Authorization: Bearer SEU_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "messaging_product": "whatsapp",
    "to": "NUMERO_DESTINATARIO",
    "type": "text",
    "text": {
      "body": "Olá, mundo, da Unoapi Cloud!"
    }
  }'
```

---

## 🤝 Contribuições

Este é um projeto de código aberto. Contribuições são bem-vindas! Sinta-se à vontade para abrir *issues* ou enviar *pull requests*.

## 📄 Licença

Este projeto é licenciado sob a [GPL-3.0 license](./license.txt).