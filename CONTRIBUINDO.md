# 🤝 Guia de Contribuição

Ficamos felizes com o seu interesse em contribuir para a Unoapi Cloud! Este guia detalha como você pode configurar seu ambiente, seguir nossos padrões e submeter suas contribuições.

## 💡 Como Contribuir

Você pode contribuir de várias formas:

-   **Relatando Bugs:** Se encontrar um problema, abra uma [Issue](https://github.com/clairton/unoapi-cloud/issues) detalhando o erro, como reproduzi-lo e a versão que você está usando.
-   **Sugerindo Melhorias:** Tem uma ideia para uma nova funcionalidade ou melhoria? Abra uma [Issue](https://github.com/clairton/unoapi-cloud/issues) para discutir.
-   **Enviando Código:** Se você quer corrigir um bug ou implementar uma nova funcionalidade, siga os passos abaixo.

## 🛠️ Configurando o Ambiente de Desenvolvimento

A maneira mais fácil e recomendada de configurar o ambiente de desenvolvimento é usando o Docker.

1.  **Clone o Fork do Repositório:**
    - Primeiro, faça um [fork](https://github.com/clairton/unoapi-cloud/fork) do repositório para a sua própria conta do GitHub.
    - Em seguida, clone o seu fork localmente:
      ```bash
      git clone https://github.com/SEU-USUARIO/unoapi-cloud.git
      cd unoapi-cloud
      ```

2.  **Configure o Arquivo `.env`:**
    - Copie o arquivo de exemplo e preencha as variáveis necessárias para o ambiente de desenvolvimento:
      ```bash
      cp .env.example .env
      ```

3.  **Inicie o Ambiente com Docker Compose:**
    - Use o `docker-compose.yml` da raiz do projeto para iniciar todos os serviços:
      ```bash
      docker-compose up -d
      ```
    - O código-fonte local será montado dentro dos contêineres, então qualquer alteração que você fizer nos arquivos será refletida em tempo real.

## ✨ Padrões de Código

Para manter a consistência e a qualidade do código, usamos as seguintes ferramentas:

-   **[ESLint](https://eslint.org/):** Para análise estática e padronização do código.
-   **[Prettier](https://prettier.io/):** Para formatação automática do código.

Antes de submeter seu código, certifique-se de que ele está de acordo com as regras, executando os seguintes comandos:

-   **Verificar Linting:**
    ```bash
    npm run lint
    ```
-   **Formatar o Código:**
    ```bash
    npm run format
    ```

## 🧪 Executando os Testes

É crucial que todas as contribuições passem nos testes existentes e, se for o caso, incluam novos testes para as novas funcionalidades.

-   Para executar a suíte de testes completa:
    ```bash
    npm test
    ```
-   Se você estiver usando o ambiente Docker, pode executar os testes dentro do contêiner `web`:
    ```bash
    docker-compose exec web npm test
    ```

## 🚀 Submetendo suas Contribuições (Pull Request)

1.  **Crie uma Nova Branch:**
    - Crie uma branch descritiva para a sua alteração:
      ```bash
      git checkout -b feat/minha-nova-funcionalidade
      ```
      ou
      ```bash
      git checkout -b fix/correcao-de-bug
      ```

2.  **Faça o Commit das Suas Alterações:**
    - Adicione e faça o commit das suas alterações com uma mensagem clara, seguindo o padrão [Conventional Commits](https://www.conventionalcommits.org/).
      ```bash
      git add .
      git commit -m "feat: Adiciona suporte para envio de stickers"
      ```

3.  **Envie para o Seu Fork:**
    - Envie a sua branch para o seu fork no GitHub:
      ```bash
      git push origin feat/minha-nova-funcionalidade
      ```

4.  **Abra um Pull Request:**
    - Vá para a página do repositório original da Unoapi Cloud no GitHub.
    - O GitHub irá detectar que você enviou uma nova branch e mostrará um botão para "Compare & pull request".
    - Clique nele, preencha o template do Pull Request com uma descrição detalhada das suas alterações e submeta-o para revisão.

Agradecemos por sua contribuição!