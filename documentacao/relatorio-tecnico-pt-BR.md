# Relatório Técnico: Arquitetura e Mitigação de Falhas da Uno API Cloud

Este documento fornece uma análise detalhada da arquitetura da Uno API Cloud, com foco em estratégias para maximizar o recebimento de mensagens, garantir o alto desempenho e mitigar falhas.

## 1. Análise de Falhas no Recebimento de Mensagens (Webhook)

A perda de mensagens é um problema crítico em qualquer sistema de comunicação. Na Uno API Cloud, a resiliência do webhook é fundamental para garantir a entrega confiável de mensagens.

### Erros Comuns de Webhook

Os erros mais comuns em webhooks são:

*   **Timeout:** Ocorre quando o seu servidor de webhook demora muito para responder à requisição da Uno API Cloud.
*   **Código HTTP não-200:** A Uno API Cloud espera um código de status `200 OK` do seu webhook para confirmar o recebimento da mensagem. Qualquer outro código (como `4xx` ou `5xx`) será interpretado como uma falha.

### Mecanismos de Retry (Retentativas)

A Uno API Cloud utiliza o RabbitMQ para gerenciar o envio de mensagens para os webhooks. A lógica de retentativa é a seguinte:

*   **Retentativas com Delay Exponencial:** Se o seu webhook falhar, a mensagem não será descartada. Em vez disso, ela será reenviada para uma fila de espera (`*.delayed`) e, em seguida, devolvida à fila principal para uma nova tentativa de entrega. O tempo de espera entre as tentativas aumenta a cada falha, seguindo um padrão de delay exponencial.
*   **Limite de Retentativas:** O número máximo de retentativas é definido pela variável de ambiente `UNOAPI_MESSAGE_RETRY_LIMIT` (o padrão é 5).
*   **Dead Letter Queue (DLQ):** Se a mensagem continuar a falhar após o número máximo de retentativas, ela será movida para uma "Dead Letter Queue" (`*.dead`). Isso garante que nenhuma mensagem seja perdida, permitindo que você analise a causa da falha e processe a mensagem manualmente, se necessário.

### Notificação de Falhas

A Uno API Cloud pode notificar sobre falhas de webhook, mas isso requer uma configuração específica:

*   **`STATUS_FAILED_WEBHOOK_URL`:** Você deve configurar esta variável de ambiente com a URL do seu webhook que receberá as notificações de falha.
*   **Fila `UNOAPI_QUEUE_WEBHOOK_STATUS_FAILED`:** Quando um webhook falha, a Uno API Cloud envia a mensagem para esta fila. Um processo consumidor então envia a notificação de falha para a URL que você configurou.

### Configurações Recomendadas no Lado do Cliente

Para garantir um recebimento consistente e tolerante a falhas, recomendamos as seguintes configurações no seu servidor de webhook:

*   **Responda Rapidamente:** Seu webhook deve responder com um `200 OK` o mais rápido possível para evitar timeouts. Se você precisar executar um processamento demorado, faça-o de forma assíncrona (em uma fila ou em segundo plano) após o envio da resposta.
*   **Seja Idempotente:** Seu sistema deve ser capaz de lidar com mensagens duplicadas. Como a Uno API Cloud pode reenviar mensagens, é possível que você receba a mesma mensagem mais de uma vez. Use um identificador único da mensagem (como o `message_id`) para evitar o processamento duplicado.
*   **Monitore seus Logs:** Monitore os logs do seu servidor de webhook para identificar e corrigir erros rapidamente.

## 2. Deploy de Alta Performance com Docker Compose e Portainer

A Uno API Cloud é projetada para ser executada em contêineres, o que facilita o deploy e a escalabilidade. O arquivo `examples/docker-compose.yml` fornece uma excelente base para um ambiente de produção.

### Guia de Deploy Passo a Passo

1.  **Pré-requisitos:**
    *   Docker e Docker Compose instalados.
    *   Portainer instalado e operacional.
    *   Um arquivo `.env` com as variáveis de ambiente necessárias (veja a seção de variáveis de ambiente abaixo).

2.  **Crie uma Stack no Portainer:**
    *   No menu do Portainer, vá para "Stacks" e clique em "Add stack".
    *   Dê um nome para a sua stack (por exemplo, `uno-api`).
    *   No editor web, cole o conteúdo do arquivo `examples/docker-compose.yml`.
    *   Certifique-se de que o Portainer tenha acesso ao seu arquivo `.env` ou configure as variáveis de ambiente diretamente na interface do Portainer.
    *   Clique em "Deploy the stack".

### Configurações de Alta Disponibilidade e Escalabilidade

O arquivo `examples/docker-compose.yml` já inclui configurações de limites de recursos para garantir a estabilidade do sistema.

*   **Limites de CPU e Memória:**
    ```yaml
    deploy:
      resources:
        limits:
          cpus: '0.75'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 128M
    ```
    *   **`limits`:** Define o máximo de CPU e memória que um contêiner pode usar.
    *   **`reservations`:** Garante que o contêiner tenha a quantidade de CPU e memória reservada.

*   **Réplicas (Escalabilidade):**
    Para aumentar a capacidade de processamento da sua aplicação, você pode aumentar o número de réplicas dos serviços `web` e `worker`. No Portainer, você pode fazer isso editando o serviço e ajustando o número de réplicas.

### Variáveis de Ambiente Essenciais

As seguintes variáveis de ambiente são cruciais para a configuração da Uno API Cloud:

*   `AMQP_URL`: A URL de conexão com o RabbitMQ.
*   `REDIS_URL`: A URL de conexão com o Redis.
*   `WEBHOOK_URL`: A URL do seu webhook.
*   `WEBHOOK_TOKEN`: O token de autenticação para o seu webhook.
*   `SENTRY_DSN`: O DSN do seu projeto Sentry (veja a próxima seção).
*   `UNOAPI_MESSAGE_RETRY_LIMIT`: O número máximo de retentativas para o envio de mensagens.
*   `UNOAPI_MESSAGE_RETRY_DELAY`: O tempo de espera (em milissegundos) entre as retentativas.
*   `STATUS_FAILED_WEBHOOK_URL`: A URL para receber notificações de falha de webhook.

## 3. Configuração e Monitoramento de Logs com Sentry

O Sentry é uma ferramenta poderosa para o monitoramento de erros e o rastreamento de eventos.

### Integração e Variáveis de Ambiente

A integração com o Sentry é simples. Basta definir a seguinte variável de ambiente:

*   **`SENTRY_DSN`:** O DSN (Data Source Name) do seu projeto Sentry. Você pode encontrá-lo nas configurações do seu projeto no Sentry.

Com esta variável definida, a Uno API Cloud enviará automaticamente todos os erros não tratados para o Sentry.

### Rastreamento de Mensagens e Diagnóstico de Erros

Para um monitoramento eficaz, recomendamos rastrear o `message_id` em seus logs e eventos do Sentry. Isso permitirá que você:

*   **Acompanhe o Fluxo de uma Mensagem:** Siga o ciclo de vida de uma mensagem desde o recebimento até a entrega.
*   **Diagnostique Erros Rapidamente:** Quando um erro ocorrer, você poderá usar o `message_id` para encontrar a mensagem exata que causou o problema e analisar o seu conteúdo.

## 4. Estrutura Detalhada do RabbitMQ

O RabbitMQ é o coração da arquitetura de mensagens da Uno API Cloud. Ele garante a persistência, a ordem e a entrega confiável de mensagens.

### Mapeamento de Filas

A Uno API Cloud usa várias filas para gerenciar diferentes tipos de mensagens:

*   **`unoapi.outgoing`:** Fila principal para o envio de mensagens.
*   **`unoapi.incoming`:** Fila principal para o recebimento de mensagens.
*   **`unoapi.webhook.status.failed`:** Fila para mensagens de webhook que falharam.
*   **`unoapi.media`:** Fila para o processamento de mídias.
*   **`unoapi.notification`:** Fila para o envio de notificações.
*   **`unoapi.delayed`:** Fila de espera para mensagens que serão reenviadas após um delay.
*   **`unoapi.dead`:** Fila para mensagens que falharam após todas as retentativas (DLQ).

### Funcionamento de Filas de Envio, Recebimento, Retentativas e DLQ

O fluxo de mensagens no RabbitMQ é o seguinte:

1.  **Envio:** Quando uma mensagem é enviada, ela é publicada na exchange `unoapi.broker` e roteada para a fila `unoapi.outgoing`.
2.  **Consumo:** Um consumidor (worker) da fila `unoapi.outgoing` processa a mensagem e a envia para o webhook.
3.  **Falha:** Se o webhook falhar, o consumidor envia a mensagem para a fila `unoapi.delayed` com um tempo de expiração.
4.  **Retentativa:** Após o tempo de expiração, a mensagem é movida de volta para a fila `unoapi.outgoing` para uma nova tentativa.
5.  **DLQ:** Se a mensagem continuar a falhar, ela é movida para a fila `unoapi.dead` para análise manual.

Este mecanismo garante que nenhuma mensagem seja perdida e que o sistema seja resiliente a falhas temporárias no seu webhook.

## 5. Stacks de Exemplo (Docker Compose)

O diretório `examples/` contém várias configurações de `docker-compose.yml` para diferentes casos de uso.

### Stack Padrão (Produção)

*   **Arquivo:** `examples/docker-compose.yml`
*   **Propósito:** Fornece uma configuração de produção robusta e pronta para uso, com serviços essenciais como Redis, RabbitMQ e Minio. Inclui limites de recursos para garantir a estabilidade.

### Stack de Monitoramento

*   **Arquivo:** `examples/monitoring/docker-compose.yml`
*   **Propósito:** Configura um ambiente de monitoramento completo com Loki, Promtail, Grafana, Prometheus e cAdvisor. É ideal para coletar, visualizar e analisar logs e métricas de todos os seus contêineres Docker em tempo real.

### Stack UnoChat (Uno API + Chatwoot)

*   **Arquivo:** `examples/unochat/docker-compose.yml`
*   **Propósito:** É a stack mais completa, oferecendo uma solução de atendimento ao cliente pronta para uso. Ela combina a Uno API Cloud com o Chatwoot, utilizando Traefik como reverse proxy para gerenciar o roteamento e os certificados SSL. Além do Chatwoot, a stack inclui todos os outros serviços necessários: Redis, PostgreSQL, RabbitMQ e Minio.

## 6. Depuração Avançada: Rastreando Mensagens entre Baileys e Uno API

Se você suspeita que as mensagens estão sendo perdidas entre a biblioteca Baileys (que se comunica com o WhatsApp) e o núcleo da Uno API, siga estes passos para rastrear o fluxo da mensagem.

### Habilitando Logs de Depuração

O primeiro passo é aumentar a verbosidade dos logs. Configure a seguinte variável de ambiente no seu `docker-compose.yml` ou arquivo `.env`:

```
LOG_LEVEL=debug
```

Isso fará com que a aplicação gere logs detalhados de cada etapa do processamento da mensagem.

### O Fluxo da Mensagem: Pontos de Verificação

Com os logs de depuração ativados, você pode acompanhar a jornada da mensagem através dos seguintes pontos de verificação:

1.  **Recebimento da Mensagem (Baileys):**
    *   **Arquivo:** `src/services/client_baileys.ts`
    *   **Log a Procurar:** Procure por logs com o texto `messages.upsert`. A presença desse log indica que a mensagem foi recebida com sucesso do WhatsApp pela biblioteca Baileys.

2.  **Processamento pelo Listener:**
    *   **Arquivo:** `src/services/listener_baileys.ts`
    *   **Log a Procurar:** Procure por `Listener receive message`. Este log confirma que a mensagem foi passada do Baileys para o listener da Uno API para ser processada e transformada no formato interno.

3.  **Publicação na Fila (RabbitMQ):**
    *   **Arquivo:** `src/services/outgoing_amqp.ts`
    *   **Log a Procurar:** Procure por `Publishing at exchange`. Este log é a confirmação final de que a mensagem foi formatada com sucesso e publicada na fila do RabbitMQ para ser entregue ao seu webhook.

Se a mensagem aparecer no log do passo 1, mas não no passo 3, o problema provavelmente está em uma das etapas de processamento ou transformação dentro da Uno API. Analise os logs de erro entre esses dois pontos para identificar a causa raiz. Se a mensagem não aparecer nem no passo 1, o problema pode estar na conexão com o WhatsApp ou em uma configuração incorreta da sessão.

## 7. Plano de Migração: de Baileys para Whatsmeow

Migrar a biblioteca de conexão com o WhatsApp de Baileys (JavaScript) para Whatsmeow (Go) é uma mudança arquitetônica complexa, mas que pode trazer benefícios de performance e estabilidade.

### Proposta de Arquitetura: Microserviço `whatsapp-connector`

A abordagem recomendada é a criação de um novo microserviço em Go, que chamaremos de `whatsapp-connector`. Este serviço será o único componente do sistema a interagir diretamente com a biblioteca Whatsmeow.

*   **Comunicação via RabbitMQ:**
    *   O serviço `whatsapp-connector` (Go) se comunicará com a aplicação Uno API (Node.js) exclusivamente através do RabbitMQ.
    *   **Recebimento:** Ao receber uma mensagem do WhatsApp, o `whatsapp-connector` a publicará em uma nova fila do RabbitMQ (ex: `unoapi.incoming.whatsmeow`).
    *   **Envio:** O `whatsapp-connector` consumirá mensagens de uma fila dedicada a envios (ex: `unoapi.outgoing.whatsmeow`) e utilizará a Whatsmeow para enviá-las ao WhatsApp.

### Pontos de Refatoração na Aplicação Node.js

A aplicação principal em Node.js precisará das seguintes modificações:

1.  **Remoção do Baileys:** A dependência do `baileys` e toda a camada de serviços que interage com ela (`client_baileys.ts`, `listener_baileys.ts`, `socket.ts`) serão removidas.
2.  **Novos Consumidores e Produtores RabbitMQ:**
    *   Um novo consumidor RabbitMQ será criado para processar as mensagens da fila `unoapi.incoming.whatsmeow`.
    *   A lógica de envio de mensagens (`outgoing_amqp.ts`) será modificada para, em vez de chamar o `client.send`, publicar a mensagem na fila `unoapi.outgoing.whatsmeow`.
3.  **Refatoração do `transformer.ts`:** A maior parte do trabalho estará em reescrever ou adaptar a lógica de transformação.
    *   O novo serviço em Go será responsável por converter a mensagem do formato Whatsmeow para um formato JSON genérico antes de publicá-la no RabbitMQ.
    *   A aplicação Node.js precisará adaptar a função `fromBaileysMessageContent` para que ela entenda este novo formato genérico.

### Principais Desafios

*   **Linguagem de Programação:** A migração exige conhecimento sólido em Go para desenvolver o novo microserviço.
*   **Paridade de Funcionalidades:** É crucial garantir que o novo serviço com Whatsmeow tenha todas as funcionalidades que a integração com o Baileys atualmente oferece (envio de todos os tipos de mídia, gerenciamento de grupos, recibos de leitura, etc.).
*   **Complexidade de Deploy:** A nova arquitetura adiciona mais um serviço para ser gerenciado, o que aumenta a complexidade do `docker-compose.yml` e do monitoramento.
*   **Transformação de Dados:** A lógica de conversão de mensagens entre os formatos das bibliotecas e o formato interno da Uno API é complexa e precisará ser cuidadosamente reescrita.
