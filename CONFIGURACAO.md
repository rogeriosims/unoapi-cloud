# ⚙️ Guia de Configuração (Variáveis de Ambiente)

Este documento detalha todas as variáveis de ambiente usadas pela Unoapi Cloud. Use este guia para preencher seu arquivo `.env` corretamente.

---

## 🚀 Configurações Principais

Estas são as variáveis mais importantes para o funcionamento básico da API.

| Variável | Descrição | Exemplo | Obrigatório |
| --- | --- | --- | --- |
| `PORT` | A porta em que o servidor da API será executado. | `3000` | Sim |
| `BASE_URL` | A URL pública base da sua API. Usada para gerar links, como os de download de mídia. | `http://localhost:3000` | Sim |
| `SECRET_KEY` | Uma chave secreta usada para segurança interna da aplicação. | `uma-chave-super-secreta` | Sim |
| `LOG_LEVEL` | O nível de detalhe dos logs (`trace`, `debug`, `info`, `warn`, `error`, `fatal`). | `info` | Não |

---

## 💾 Configurações de Armazenamento (Store)

Define como os dados de sessão e outras informações são armazenadas.

| Variável | Descrição | Valores Possíveis | Padrão |
| --- | --- | --- | --- |
| `STORE_TYPE` | O tipo de armazenamento a ser usado. | `file`, `redis` | `file` |
| `REDIS_URL` | A URL de conexão para o servidor Redis, caso `STORE_TYPE` seja `redis`. | `redis://redis:6379` | - |
| `SESSION_TTL` | Tempo de vida (em segundos) para os dados da sessão em cache. | `3600` | - |
| `DATA_TTL` | Tempo de vida (em segundos) para outros dados em cache. | `3600` | - |

---

## ☁️ Configurações da API Oficial do WhatsApp Business

Estas variáveis são **obrigatórias apenas se você estiver usando a API Oficial da Meta**.

| Variável | Descrição | Exemplo |
| --- | --- | --- |
| `GRAPH_API_VERSION` | A versão da Graph API da Meta a ser usada. | `v18.0` |
| `ACCESS_TOKEN` | O token de acesso permanente gerado no seu painel de desenvolvedores da Meta. | `EAAD...` |
| `PHONE_NUMBER_ID` | O ID do número de telefone que você configurou na plataforma da Meta. | `102030405060` |
| `VERIFY_TOKEN` | Um token secreto que você cria. Usado para verificar a autenticidade dos webhooks recebidos da Meta. | `token-de-verificacao-secreto` |

---

## 🎣 Configurações de Webhook

Controla como a API envia os eventos recebidos para seus próprios sistemas.

| Variável | Descrição | Exemplo |
| --- | --- | --- |
| `WEBHOOK_URL` | A URL do seu sistema que receberá os eventos da Unoapi Cloud (ex: novas mensagens). | `https://meu-sistema.com/webhook` |
| `WEBHOOK_TOKEN` | Um token de autorização que a Unoapi Cloud enviará no cabeçalho `Authorization` para seu webhook. | `meu-token-de-webhook` |
| `WEBHOOK_HEADER` | O nome do cabeçalho de autorização enviado para o seu webhook. | `Authorization` |

---

## 🗃️ Configurações de Armazenamento de Mídia (Minio / S3)

Configura o armazenamento de arquivos de mídia, como imagens, vídeos e documentos.

| Variável | Descrição | Exemplo |
| --- | --- | --- |
| `STORAGE_TYPE` | O tipo de armazenamento de mídia. | `s3`, `file` |
| `STORAGE_ENDPOINT` | O endpoint do seu serviço de armazenamento compatível com S3 (como Minio ou AWS S3). | `http://minio:9000` |
| `STORAGE_BUCKET_NAME` | O nome do "bucket" (contêiner) onde os arquivos serão armazenados. | `unoapi-media` |
| `STORAGE_ACCESS_KEY_ID` | A chave de acesso para seu serviço de armazenamento. | `minioadmin` |
| `STORAGE_SECRET_ACCESS_KEY` | A chave secreta para seu serviço de armazenamento. | `minioadmin` |
| `STORAGE_FORCE_PATH_STYLE`| Força o uso do estilo de caminho para acesso ao bucket. Necessário para Minio. | `true` |

---

## ⚙️ Configurações Diversas

Outras configurações que controlam o comportamento da API.

| Variável | Descrição | Valores Possíveis | Padrão |
| --- | --- | --- | --- |
| `IGNORE_GROUP_MESSAGES` | Se `true`, ignora mensagens recebidas de grupos. | `true`, `false` | `false` |
| `IGNORE_BROADCAST_MESSAGES` | Se `true`, ignora mensagens de listas de transmissão. | `true`, `false` | `false` |
| `IGNORE_OWN_MESSAGES` | Se `true`, ignora mensagens enviadas pela própria API. | `true`, `false` | `false` |
| `REJECT_CALLS` | Se `true`, rejeita automaticamente as chamadas de voz e vídeo. | `true`, `false` | `false` |
| `REJECT_CALLS_WEBHOOK`| URL de webhook para notificar sobre chamadas rejeitadas. | URL | - |
| `SEND_CONNECTION_STATUS` | Se `true`, envia o status da conexão para o webhook. | `true`, `false` | `false` |
| `UNOAPI_AUTH_TOKEN` | Token de autenticação para proteger os endpoints da Unoapi. | qualquer string | - |