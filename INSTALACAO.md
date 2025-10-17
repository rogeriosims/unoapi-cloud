# 🚀 Guia de Instalação e Configuração

Este guia detalha o processo completo para instalar, configurar e iniciar a aplicação Unoapi Cloud em seu ambiente local.

## 📋 Pré-requisitos

Antes de começar, garanta que você tenha os seguintes softwares instalados em sua máquina:

- **[Node.js](https://nodejs.org/)**: Versão 20.x ou superior.
- **[npm](https://www.npmjs.com/)**: Geralmente instalado junto com o Node.js.
- **[Git](https://git-scm.com/)**: Para clonar o repositório.

## ⚙️ Passo a Passo da Instalação

### 1. Clone o Repositório

Primeiro, clone o repositório do projeto para a sua máquina local usando o Git:

```bash
git clone https://github.com/clairton/unoapi-cloud.git
cd unoapi-cloud
```

### 2. Instale as Dependências

A aplicação utiliza o `npm` para gerenciar seus pacotes. Execute o comando abaixo para instalar todas as dependências necessárias listadas no arquivo `package.json`:

```bash
npm install --legacy-peer-deps
```

> **Nota:** Usamos a flag `--legacy-peer-deps` para resolver conflitos de dependências que podem ocorrer entre os pacotes do projeto.

### 3. Configure as Variáveis de Ambiente

As configurações da aplicação são gerenciadas por meio de variáveis de ambiente. O projeto inclui um arquivo de exemplo chamado `.env.example`.

**a. Crie seu arquivo de configuração:**

Copie o arquivo de exemplo para criar seu próprio arquivo `.env`:

```bash
cp .env.example .env
```

**b. Edite o arquivo `.env`:**

Abra o arquivo `.env` em um editor de texto e ajuste as variáveis conforme sua necessidade. As variáveis mais importantes a serem configuradas inicialmente são:

- `PORT`: A porta em que a API será executada (ex: `3000`).
- `BASE_URL`: A URL base da sua API (ex: `http://localhost:3000`).
- `SECRET_KEY`: Uma chave secreta para a aplicação.
- `STORE_TYPE`: O tipo de armazenamento para as sessões (`file` ou `redis`).

### 4. Execute a Aplicação

Após a configuração, você pode iniciar a aplicação em modo de desenvolvimento. Este modo monitora as alterações nos arquivos e reinicia o servidor automaticamente.

```bash
npm run dev
```

Se tudo estiver configurado corretamente, você verá uma mensagem no console indicando que o servidor está online:

```
[INFO] Unoapi Cloud version: X.X.X, listening on port: 3000
```

## ✅ Pronto!

Sua instância da Unoapi Cloud está instalada e em execução. Agora você pode prosseguir para o [Guia de Conexão com o WhatsApp](./CONEXAO_WHATSAPP.md) para começar a usar a API.

##  troubleshoot

### Erro `node-gyp` no `npm install`
Este erro geralmente ocorre se as ferramentas de compilação (como `build-essentials` no Linux ou `Xcode Command Line Tools` no macOS) não estiverem instaladas. Consulte a documentação do [`node-gyp`](https://github.com/nodejs/node-gyp) para obter instruções de instalação específicas para o seu sistema operacional.

### Conflitos de Dependência
Se `npm install --legacy-peer-deps` não funcionar, você pode tentar `npm install --force`. No entanto, isso pode levar a comportamentos inesperados. A melhor abordagem é tentar entender o conflito de dependências e resolvê-lo, se possível.

---

## 🐳 Instalação com Docker

Para uma instalação mais isolada e consistente, você pode usar o Docker. O projeto oferece duas configurações principais com `docker-compose`.

### Opção 1: Ambiente de Desenvolvimento (Recomendado para Testes)

Esta opção usa o `docker-compose.yml` na raiz do projeto. É ideal para desenvolvimento e testes locais, pois monta o código-fonte diretamente nos contêineres, permitindo o hot-reload.

**Passo a Passo:**

1.  **Pré-requisitos:**
    - [Docker](https://www.docker.com/get-started)
    - [Docker Compose](https://docs.docker.com/compose/install/)

2.  **Configure o `.env`:**
    - Certifique-se de ter criado e configurado seu arquivo `.env` a partir do `.env.example`, como explicado na instalação manual. As configurações de `REDIS_URL`, `AMQP_URL`, etc., serão usadas pelos contêineres.

3.  **Inicie os Serviços:**
    - No diretório raiz do projeto, execute:
      ```bash
      docker-compose up -d
      ```
    - Este comando irá construir as imagens (se necessário) e iniciar todos os serviços (`web`, `worker`, `redis`, `rabbitmq`, `minio`) em segundo plano (`-d`).

4.  **Acompanhe os Logs:**
    - Para ver os logs dos serviços e verificar se tudo iniciou corretamente:
      ```bash
      docker-compose logs -f
      ```

5.  **Parando o Ambiente:**
    - Para parar todos os serviços:
      ```bash
      docker-compose down
      ```

### Opção 2: Ambiente de Produção/Demonstração

Esta opção usa o `docker-compose.yml` localizado na pasta `examples/`. É uma configuração mais complexa e robusta, que utiliza uma imagem pré-construída e inclui serviços adicionais como `redis-commander` e um proxy reverso.

**Passo a Passo:**

1.  **Navegue até a Pasta de Exemplos:**
    ```bash
    cd examples/
    ```

2.  **Configure as Variáveis de Ambiente:**
    - Este compose depende de um grande número de variáveis de ambiente. Você precisará criar um arquivo `.env` dentro da pasta `examples/` e preenchê-lo com as configurações necessárias, como `DOMAIN`, `REDIS_PASSWORD`, credenciais do Minio, etc. Consulte o arquivo `examples/docker-compose.yml` para ver todas as variáveis necessárias.

3.  **Inicie os Serviços:**
    ```bash
    docker-compose -f examples/docker-compose.yml up -d
    ```

> **Atenção:** Esta configuração é avançada e requer um bom entendimento de Docker e redes. É projetada para ser implantada em um servidor com um nome de domínio configurado.