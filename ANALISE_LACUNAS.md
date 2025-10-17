# Análise de Lacunas na Documentação da API

Este documento detalha os endpoints que foram implementados no código-fonte (`src/router.ts`) mas não foram incluídos na documentação inicial (`API_DOCUMENTATION.md` e `openapi.yaml`).

## 📈 Resumo

A análise revelou que uma parte significativa dos endpoints da API não estava documentada. A documentação inicial focou nas operações mais comuns, como envio de mensagens e gerenciamento de sessões, mas deixou de fora funcionalidades importantes como webhooks, gerenciamento de mídia, templates e outras rotas de utilidade.

## 📝 Endpoints Não Documentados

A seguir, a lista completa de endpoints que precisam ser documentados:

### Webhooks
- `POST /webhooks/whatsapp/:phone`: Rota principal para receber eventos do WhatsApp (mensagens, status, etc.).
- `GET /webhooks/whatsapp/:phone`: Endpoint para a verificação inicial (handshake) do webhook com a plataforma do WhatsApp.
- `POST /webhooks/fake/:phone`: Um webhook para testes internos, que simula o recebimento de mensagens.

### Rotas de Interface e Status
- `GET /`: Rota raiz, geralmente exibe uma página inicial ou status.
- `GET /connect/:phone`: Exibe uma página para o usuário escanear o QR Code e conectar uma nova sessão.
- `GET /ping`: Endpoint simples para verificar se a API está online (`pong`).

### Gestão de Números e Tokens
- `GET /:version/debug_token`: Endpoint para obter um token de depuração.
- `GET /:version/:phone`: Obtém informações detalhadas sobre um número de telefone específico.
- `GET /:version/:phone/phone_numbers`: Lista todos os números de telefone associados a uma conta.
- `POST /:phone/request_code`: Inicia o processo de registro solicitando um código de pareamento.

### Templates e Mensagens
- `POST /:version/:phone/templates`: Cria ou envia uma mensagem baseada em um template pré-aprovado.
- `POST /:version/:phone/marketing_messages`: Envia mensagens de marketing (requer opt-in do usuário).

### Mídia
- `GET /:version/:phone/:media_id`: Obtém informações sobre um arquivo de mídia a partir de seu ID.
- `GET /:version/download/:phone/:file`: Endpoint para fazer o download direto de um arquivo de mídia.

### Utilitários
- `POST /:phone/blacklist/:webhook_id`: Adiciona ou remove um número da lista negra (blacklist) para um webhook específico.
- `POST /timer/:phone/:to`: Inicia um temporizador para aguardar uma resposta antes de enviar uma mensagem.
- `DELETE /timer/:phone/:to`: Interrompe o temporizador.

## ✅ Próximos Passos

A documentação será expandida para incluir todos os endpoints listados acima, seguindo o formato e as diretrizes detalhadas na nova solicitação de tarefa.