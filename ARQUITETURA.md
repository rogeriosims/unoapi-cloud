# 🏗️ Visão Geral da Arquitetura

Este documento fornece uma visão geral da arquitetura da Unoapi Cloud, explicando o papel de cada serviço e como eles interagem.

## Componentes Principais

A Unoapi Cloud é projetada com uma arquitetura baseada em microserviços para garantir escalabilidade e resiliência. Os principais componentes são:

- **Web Service (`web`):**
  - **Responsabilidade:** É a porta de entrada da API. Ele expõe os endpoints REST (como `/messages`, `/sessions`, etc.) para o mundo externo.
  - **Funcionamento:** Recebe as requisições HTTP, valida-as e, em vez de processá-las diretamente, publica as tarefas em uma fila no RabbitMQ.

- **Worker Service (`worker`):**
  - **Responsabilidade:** É o "cérebro" da aplicação. Ele consome as tarefas da fila do RabbitMQ e as executa.
  - **Funcionamento:** O worker é quem de fato se conecta com o WhatsApp (via Baileys ou API Oficial), envia as mensagens, processa os eventos recebidos e realiza as operações mais pesadas. Essa separação garante que a API (web) possa responder rapidamente às requisições, sem esperar a conclusão de tarefas demoradas.

- **RabbitMQ (`rabbitmq`):**
  - **Responsabilidade:** Atua como um *message broker* (intermediário de mensagens).
  - **Funcionamento:** Gerencia as filas de tarefas entre o `web` service e o `worker` service. Quando o `web` service recebe um pedido para enviar uma mensagem, ele publica essa tarefa na fila. O `worker` consome a tarefa da fila e a executa. Isso desacopla os serviços e aumenta a tolerância a falhas.

- **Redis (`redis`):**
  - **Responsabilidade:** Serve como um banco de dados em memória de alta performance.
  - **Funcionamento:** É usado para armazenar dados de acesso rápido, como informações de sessão, caches e credenciais. Usar Redis é mais eficiente e escalável do que armazenar tudo em arquivos locais, especialmente em um ambiente com múltiplos workers.

- **Minio (`minio`):**
  - **Responsabilidade:** É um serviço de armazenamento de objetos compatível com a API do AWS S3.
  - **Funcionamento:** Armazena arquivos de mídia, como imagens, vídeos e documentos que são enviados ou recebidos pela API.

## Diagrama de Fluxo de Dados

Abaixo está um diagrama simplificado que ilustra o fluxo de uma requisição para enviar uma mensagem:

```
          +-------------------+
          |   Cliente da API  |
          | (Seu Aplicativo)  |
          +-------------------+
                   |
     (1) Requisição POST /messages
                   |
                   v
+---------------------------------------+
|             Web Service               |
|   (Recebe a requisição, valida e      |
|    responde 200 OK imediatamente)     |
+---------------------------------------+
                   |
   (2) Publica a tarefa "Enviar Mensagem"
                   |
                   v
+---------------------------------------+
|            RabbitMQ (Fila)            |
+---------------------------------------+
                   |
   (3) Consome a tarefa da fila
                   |
                   v
+---------------------------------------+      +------------------+
|           Worker Service              |----->|     WhatsApp     |
|  (Conecta ao WhatsApp e envia a msg)  |      | (API ou Celular) |
+---------------------------------------+      +------------------+
     |                 ^
     | (4) Salva/Lê    | (5) Recebe
     |     Sessão      |     Eventos
     v                 |
+---------------------------------------+
|                  Redis                |
+---------------------------------------+
```

### Explicação do Fluxo:

1.  **Requisição:** Seu aplicativo faz uma requisição `POST` para o endpoint `/messages` do **Web Service**.
2.  **Enfileiramento:** O **Web Service** recebe a requisição, valida os dados, e imediatamente publica uma tarefa na fila do **RabbitMQ**. Ele já responde `200 OK` ao seu aplicativo, indicando que a tarefa foi aceita.
3.  **Processamento:** O **Worker Service**, que está constantemente monitorando a fila, consome a tarefa. Ele busca as informações da sessão no **Redis**, se conecta ao WhatsApp e envia a mensagem.
4.  **Armazenamento de Sessão:** O **Worker** usa o **Redis** para armazenar e recuperar informações da sessão (status, credenciais, etc.).
5.  **Recebimento de Eventos:** Quando o WhatsApp envia um evento de volta (como uma confirmação de entrega ou uma nova mensagem), é o **Worker** quem o processa. Se um webhook estiver configurado, o worker encaminha esse evento para a URL do seu sistema.