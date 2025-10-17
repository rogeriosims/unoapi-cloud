# 📖 Referência Completa de Endpoints

Este documento detalha todos os endpoints disponíveis na Unoapi Cloud.

---

## 🔌 Gestão de Sessões

Endpoints para conectar, verificar e gerenciar as sessões do WhatsApp.

### **▶️ Iniciar Conexão (QR Code)**

#### **Para que serve?**
Gera uma página com um QR Code para que um novo número de WhatsApp possa ser conectado à API. Este endpoint é usado na abordagem de conexão via WhatsApp Web (Baileys).

#### **Como usar?**
- **Método:** `GET`
- **URL:** `/connect/{phone}`

**Parâmetros de URL:**
- `{phone}`: O número de telefone que você deseja registrar para a sessão (ex: `5511999999999`).

#### **O que enviar?**
Nenhum corpo de requisição é necessário.

#### **O que receber?**
Uma página HTML com o QR Code para ser escaneado pelo aplicativo WhatsApp.

#### **Exemplo Prático (Acesso via Navegador)**
Abra o seguinte URL no seu navegador:
```
http://localhost:3000/connect/5511999999999
```

#### **Possíveis Erros**
- `404 - Not Found`: Se a rota estiver incorreta.
- `500 - Internal Server Error`: Se houver um problema ao gerar o QR Code.

---

### **📊 Verificar Status da Sessão**

#### **Para que serve?**
Obtém informações detalhadas sobre uma sessão específica, como status da conexão, informações do aparelho, etc.

#### **Como usar?**
- **Método:** `GET`
- **URL:** `/sessions/{phone}`

**Parâmetros de URL:**
- `{phone}`: O número da sessão que você deseja consultar.

#### **O que enviar?**
Nenhum corpo de requisição é necessário.

#### **O que receber?**
```json
{
  "success": true,
  "data": {
    "phone": "5511999999999",
    "status": "CONNECTED",
    "platform": "iPhone",
    "pushname": "Jules"
  },
  "message": "Sessão encontrada."
}
```

#### **Exemplo Prático (JavaScript)**
```javascript
async function getSessionStatus(phone) {
  const response = await fetch(`/sessions/${phone}`, {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer SEU_TOKEN_DE_ACESSO'
    }
  });
  const data = await response.json();
  console.log(data);
}
```

#### **Possíveis Erros**
- `401 - Unauthorized`: Token de acesso inválido ou não fornecido.
- `404 - Not Found`: Nenhuma sessão encontrada para o número de telefone fornecido.

---

## 🎣 Webhooks

Endpoints para receber dados do WhatsApp em tempo real.

### **📢 Receber Eventos do WhatsApp**

#### **Para que serve?**
É o endpoint principal que a plataforma do WhatsApp (ou o Baileys) usará para enviar eventos, como novas mensagens, confirmações de entrega, atualizações de status, etc.

#### **Como usar?**
- **Método:** `POST`
- **URL:** `/webhooks/whatsapp/{phone}`

**Parâmetros de URL:**
- `{phone}`: O número de telefone que está recebendo o evento.

#### **O que enviar?**
O corpo da requisição (payload) é enviado pela Meta/WhatsApp e varia dependendo do evento. Um exemplo para uma mensagem de texto recebida é:

```json
{
  "object": "whatsapp_business_account",
  "entry": [{
    "id": "BUSINESS_ACCOUNT_ID",
    "changes": [{
      "value": {
        "messaging_product": "whatsapp",
        "metadata": {
          "display_phone_number": "5511999999999",
          "phone_number_id": "PHONE_NUMBER_ID"
        },
        "contacts": [{ "profile": { "name": "Nome do Contato" }, "wa_id": "5511888888888" }],
        "messages": [{
          "from": "5511888888888",
          "id": "wamid.ID_DA_MENSAGEM",
          "timestamp": "1678886400",
          "text": { "body": "Olá, mundo!" },
          "type": "text"
        }]
      },
      "field": "messages"
    }]
  }]
}
```

#### **O que receber?**
A API deve responder com um status `200 OK` para confirmar o recebimento do evento.

#### **Exemplo Prático (Configuração na Meta)**
Você deve configurar esta URL no painel de desenvolvedores da Meta, na seção de webhooks do seu aplicativo do WhatsApp.

#### **Possíveis Erros**
- A lógica de tratamento de erros para este endpoint deve ser implementada no seu lado do webhook para processar os dados recebidos.

---

## 💬 Envio de Mensagens

Endpoints para enviar diferentes tipos de mensagens.

### **✉️ Enviar Mensagem de Texto**

#### **Para que serve?**
Envia uma mensagem de texto simples para um contato ou grupo.

#### **Como usar?**
- **Método:** `POST`
- **URL:** `/:version/{phone}/messages`

**Parâmetros de URL:**
- `{version}`: A versão da API (ex: `v1`).
- `{phone}`: O número de telefone da sua conta que enviará a mensagem.

#### **O que enviar?**
```json
{
  "messaging_product": "whatsapp",
  "to": "5511888888888",
  "type": "text",
  "text": {
    "preview_url": false,
    "body": "Olá! Esta é uma mensagem de teste."
  }
}
```

#### **O que receber?**
```json
{
  "messaging_product": "whatsapp",
  "contacts": [{
    "input": "5511888888888",
    "wa_id": "5511888888888"
  }],
  "messages": [{
    "id": "wamid.ID_DA_MENSAGEM_ENVIADA"
  }]
}
```

#### **Exemplo Prático (JavaScript)**
```javascript
async function sendMessage(senderPhone, recipientPhone, message) {
  const response = await fetch(`/v1/${senderPhone}/messages`, {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer SEU_TOKEN_DE_ACESSO',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      messaging_product: "whatsapp",
      to: recipientPhone,
      type: "text",
      text: { body: message }
    })
  });
  const data = await response.json();
  console.log(data);
}
```

#### **Possíveis Erros**
- `400 - Bad Request`: O corpo da requisição está mal formatado ou faltam campos obrigatórios.
- `401 - Unauthorized`: Token de acesso inválido.
- `404 - Not Found`: O número de telefone remetente não está conectado.

---

## 👥 Gestão de Contatos

Endpoints para interagir com contatos e grupos.

### **✅ Verificar Contatos no WhatsApp**

#### **Para que serve?**
Verifica se uma lista de números de telefone possui contas ativas no WhatsApp.

#### **Como usar?**
- **Método:** `POST`
- **URL:** `/{phone}/contacts`

**Parâmetros de URL:**
- `{phone}`: O número de telefone da sua conta que fará a verificação.

#### **O que enviar?**
```json
{
  "contacts": [
    "+5511888888888",
    "+14155552671"
  ]
}
```

#### **O que receber?**
```json
{
  "success": true,
  "data": [
    { "input": "+5511888888888", "status": "valid", "wa_id": "5511888888888" },
    { "input": "+14155552671", "status": "invalid" }
  ],
  "message": "Contatos verificados."
}
```

#### **Exemplo Prático (JavaScript)**
```javascript
async function checkContacts(senderPhone, contactsToCheck) {
  const response = await fetch(`/${senderPhone}/contacts`, {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer SEU_TOKEN_DE_ACESSO',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      contacts: contactsToCheck
    })
  });
  const data = await response.json();
  console.log(data);
}
```

#### **Possíveis Erros**
- `400 - Bad Request`: A lista de contatos está vazia ou mal formatada.
- `401 - Unauthorized`: Token de acesso inválido.

---

## 🖼️ Gestão de Mídia

Endpoints para obter e fazer download de mídias.

### **ℹ️ Obter Informações da Mídia**

#### **Para que serve?**
Recupera informações sobre um arquivo de mídia (imagem, vídeo, documento) a partir do seu ID.

#### **Como usar?**
- **Método:** `GET`
- **URL:** `/:version/{phone}/{media_id}`

**Parâmetros de URL:**
- `{version}`: A versão da API (ex: `v1`).
- `{phone}`: O número de telefone da sua conta.
- `{media_id}`: O ID da mídia que você deseja consultar.

#### **O que enviar?**
Nenhum corpo de requisição é necessário.

#### **O que receber?**
```json
{
  "messaging_product": "whatsapp",
  "url": "URL_PARA_DOWNLOAD_DA_MIDIA",
  "mime_type": "image/jpeg",
  "sha256": "HASH_DO_ARQUIVO",
  "file_size": 123456,
  "id": "ID_DA_MIDIA"
}
```

#### **Exemplo Prático (JavaScript)**
```javascript
async function getMediaInfo(phone, mediaId) {
  const response = await fetch(`/v1/${phone}/${mediaId}`, {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer SEU_TOKEN_DE_ACESSO'
    }
  });
  const data = await response.json();
  console.log(data); // Contém a URL para o download
}
```

#### **Possíveis Erros**
- `401 - Unauthorized`: Token de acesso inválido.
- `404 - Not Found`: Mídia não encontrada para o ID fornecido.

### **📥 Download de Mídia**

#### **Para que serve?**
Faz o download direto de um arquivo de mídia.

#### **Como usar?**
- **Método:** `GET`
- **URL:** `/:version/download/{phone}/{file}`

**Parâmetros de URL:**
- `{version}`: A versão da API (ex: `v1`).
- `{phone}`: O número de telefone da sua conta.
- `{file}`: O nome do arquivo ou identificador para download.

#### **O que receber?**
O arquivo de mídia binário.

---

## 📝 Gestão de Templates

Endpoints para listar e enviar mensagens baseadas em templates.

### **📋 Listar Templates de Mensagem**

#### **Para que serve?**
Recupera a lista de templates de mensagem pré-aprovados associados à sua conta do WhatsApp Business.

#### **Como usar?**
- **Método:** `GET`
- **URL:** `/:version/{phone}/message_templates`

**Parâmetros de URL:**
- `{version}`: A versão da API (ex: `v1`).
- `{phone}`: O número de telefone da sua conta.

#### **O que receber?**
```json
{
  "data": [
    {
      "name": "hello_world",
      "components": [
        { "type": "BODY", "text": "Hello {{1}}" }
      ],
      "language": "en_US",
      "status": "APPROVED",
      "id": "TEMPLATE_ID"
    }
  ]
}
```

#### **Exemplo Prático (JavaScript)**
```javascript
async function listTemplates(phone) {
  const response = await fetch(`/v1/${phone}/message_templates`, {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer SEU_TOKEN_DE_ACESSO'
    }
  });
  const data = await response.json();
  console.log(data);
}
```

#### **Possíveis Erros**
- `401 - Unauthorized`: Token de acesso inválido.
- `403 - Forbidden`: A conta não tem permissão para acessar templates.

---

## 🛠️ Utilitários

Endpoints com funcionalidades diversas e de suporte.

### **PING**

#### **Para que serve?**
Verifica a saúde e a disponibilidade da API.

#### **Como usar?**
- **Método:** `GET`
- **URL:** `/ping`

#### **O que receber?**
Uma string simples: `pong`.

---

### **Atualizar Blacklist**

#### **Para que serve?**
Adiciona ou remove um número de telefone da lista de bloqueio (blacklist) para um webhook específico, impedindo que mensagens desse número sejam processadas.

#### **Como usar?**
- **Método:** `POST`
- **URL:** `/{phone}/blacklist/{webhook_id}`

**Parâmetros de URL:**
- `{phone}`: O número de telefone a ser adicionado/removido.
- `{webhook_id}`: O ID do webhook ao qual a regra se aplica.

---

### **Solicitar Código de Pareamento**

#### **Para que serve?**
Inicia o processo de registro de um número de telefone na API Oficial do WhatsApp Business, solicitando um código de pareamento.

#### **Como usar?**
- **Método:** `POST`
- **URL:** `/{phone}/request_code`

**Parâmetros de URL:**
- `{phone}`: O número de telefone a ser registrado.

---

### **Gerenciar Timer de Resposta**

#### **Para que serve?**
- `POST /timer/{phone}/{to}`: Inicia um timer. Útil para criar fluxos onde a API aguarda um tempo antes de enviar a próxima mensagem.
- `DELETE /timer/{phone}/{to}`: Para o timer, geralmente acionado quando o cliente responde antes do tempo limite.

#### **Como usar?**
- **Método:** `POST` ou `DELETE`
- **URL:** `/timer/{phone}/{to}`

**Parâmetros de URL:**
- `{phone}`: O número da sua conta.
- `{to}`: O número do destinatário.