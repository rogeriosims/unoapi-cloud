# 💡 Exemplos Práticos de Uso

Este guia oferece exemplos de código para cenários de uso comuns da Unoapi Cloud, ajudando você a integrar a API em seus projetos rapidamente.

---

## 1. Enviando uma Mensagem de Texto Simples

Este é o caso de uso mais básico: enviar uma notificação ou uma resposta de texto para um usuário.

```javascript
// Exemplo usando Node.js com a biblioteca `node-fetch`

const fetch = require('node-fetch');

const API_URL = 'http://localhost:3000';
const SENDER_PHONE = '5511999999999'; // Seu número conectado
const RECIPIENT_PHONE = '5511888888888'; // Número do destinatário
const ACCESS_TOKEN = 'SEU_TOKEN_DE_ACESSO';

async function sendTextMessage(message) {
  try {
    const response = await fetch(`${API_URL}/v1/${SENDER_PHONE}/messages`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${ACCESS_TOKEN}`
      },
      body: JSON.stringify({
        messaging_product: "whatsapp",
        to: RECIPIENT_PHONE,
        type: "text",
        text: { body: message }
      })
    });

    if (!response.ok) {
      throw new Error(`Erro na API: ${response.statusText}`);
    }

    const data = await response.json();
    console.log('Mensagem enviada com sucesso:', data.messages[0].id);
  } catch (error) {
    console.error('Falha ao enviar mensagem:', error.message);
  }
}

sendTextMessage('Olá! Bem-vindo à nossa plataforma.');
```

---

## 2. Enviando uma Imagem

Para enviar uma mídia, você primeiro precisa fazer o upload dela para o WhatsApp para obter um ID de mídia e, em seguida, enviar esse ID para o destinatário.

> **Nota:** A API atualmente não expõe um endpoint direto para upload. Este exemplo assume que você obteve um `media_id` de outra forma (por exemplo, recebendo uma imagem e reutilizando seu ID). Uma alternativa seria usar um link para a imagem.

### Enviando Imagem com Link

```javascript
async function sendImageWithLink(imageUrl, caption) {
  const response = await fetch(`${API_URL}/v1/${SENDER_PHONE}/messages`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${ACCESS_TOKEN}` },
    body: JSON.stringify({
      messaging_product: "whatsapp",
      to: RECIPIENT_PHONE,
      type: "image",
      image: {
        link: imageUrl,
        caption: caption
      }
    })
  });
  // ... tratamento de resposta
}

sendImageWithLink('https://example.com/image.jpg', 'Olha esta imagem!');
```

---

## 3. Enviando Mensagem com Botões de Resposta Rápida

Botões incentivam a interação do usuário. Eles são fáceis de usar e ótimos para respostas padronizadas.

```javascript
async function sendMessageWithButtons() {
  const response = await fetch(`${API_URL}/v1/${SENDER_PHONE}/messages`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${ACCESS_TOKEN}` },
    body: JSON.stringify({
      messaging_product: "whatsapp",
      to: RECIPIENT_PHONE,
      type: "interactive",
      interactive: {
        type: "button",
        body: {
          text: "Olá! Gostaria de continuar?"
        },
        action: {
          buttons: [
            { type: "reply", reply: { id: "sim_continuar", title: "Sim" } },
            { type: "reply", reply: { id: "nao_parar", title: "Não" } }
          ]
        }
      }
    })
  });
  // ... tratamento de resposta
}

sendMessageWithButtons();
```

---

## 4. Recebendo e Respondendo a Mensagens (Webhook)

Para criar um bot interativo, você precisa configurar um webhook para receber mensagens e respondê-las.

Este exemplo usa **Express.js** para criar um servidor simples que atua como seu webhook.

**a. Crie seu servidor de webhook:**

```javascript
// webhook-server.js
const express = require('express');
const app = express();
app.use(express.json());

const PORT = 4000;

// Endpoint que o WhatsApp chamará
app.post('/webhook', (req, res) => {
  const messageData = req.body.entry?.[0]?.changes?.[0]?.value?.messages?.[0];

  if (messageData) {
    const from = messageData.from;
    const messageText = messageData.text.body;

    console.log(`Mensagem recebida de ${from}: "${messageText}"`);

    // Lógica do seu bot: responder "Olá" com "Oi, tudo bem?"
    if (messageText.toLowerCase() === 'olá') {
      sendTextMessage(from, 'Oi, tudo bem?');
    }
  }

  res.sendStatus(200); // Responda sempre com 200 OK
});

app.listen(PORT, () => console.log(`Servidor de webhook ouvindo na porta ${PORT}`));

// Função para enviar mensagem (reutilize a do exemplo 1)
// async function sendTextMessage(recipient, message) { ... }
```

**b. Exponha seu webhook publicamente:**

Seu servidor local (ex: `http://localhost:4000/webhook`) precisa estar acessível pela internet. Ferramentas como o **[ngrok](https://ngrok.com/)** são ótimas para isso durante o desenvolvimento.

**c. Configure a URL no painel da Meta:**

Com a URL pública em mãos (ex: `https://seusite.ngrok.io/webhook`), configure-a como o endpoint de webhook na sua conta do WhatsApp Business.