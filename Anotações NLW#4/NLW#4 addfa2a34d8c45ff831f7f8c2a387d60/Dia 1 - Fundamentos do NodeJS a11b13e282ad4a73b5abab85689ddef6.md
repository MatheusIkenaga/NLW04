# Dia 1 - Fundamentos do NodeJS

- [x]  Boas Vindas ao NLW04
- [x]  Overview da trilha de nodeJS
- [x]  O que faremos nessa aula?
- [x]  Apresentação da Dani (Instrutora)
- [x]  Dicas para ir até o fim do projeto
    - [x]  Fazer parte da comunidade
    - [x]  Tirar dúvidas
    - [x]  Se conectar com outros devs
    - [x]  Se apresentar no #network
    - [x]  Desafios com prêmios exclusivos
        - [x]  Um código por aula
        - [x]  Ficar atendo aos e-mails e na nossa comunidade
- [x]  Apresentação do Projeto
- [x]  O que é uma API?
- [x]  Por que usar Typescript?

    É um JS melhorado com algumas features (Exemplo Tipagem)

- [x]  Criar o projeto com NodeJS

    Pasta do projeto

    projeto/aulas/api

    ```powershell
    yarn init -y
    yarn add express
    yarn add @types/express -D
    ```

    criar a pasta src e dentro dela o arquivo server.ts

    ```tsx
    import express from 'express';

    const  app = express();

    app.listen(3333, ()=> console.log("Server is runninhg!"))
    ```

    O node não entende o import (pelo menos não no TypeScript

    ```powershell
    yarn add typescript -D
    yarn tsc --init
    ```

    No arquivo tsconfig.json, mudar o "strict": "true" para false

    ![Dia%201%20-%20Fundamentos%20do%20NodeJS%20a11b13e282ad4a73b5abab85689ddef6/Untitled.png](Dia%201%20-%20Fundamentos%20do%20NodeJS%20a11b13e282ad4a73b5abab85689ddef6/Untitled.png)

    ```powershell
    yarn add ts-node-dev -D
    ```

    criar o scripts no package.json

    ```json
    "scripts": {
        "dev":"ts-node-dev src/server.ts"
      },
    ```

    Depois disso, para rodar o código do server.ts:

    ```powershell
    yarn dev
    ```

    Depois é possível otimizar o package.json assim:

    ```json
    "scripts": {
        "dev":"ts-node-dev --transpile-only --ignore-watch node_modules src/server.ts"
      },
    ```

    - [x]  Criar primeira rota

    ```tsx
    import express, { request, response } from 'express';

    const  app = express();

    app.get("/", (request,response)=>{
        return response.json({message:"Hello World - NLW04"})
    })

    app.listen(3333, ()=> console.log("Server is runninhg!"))
    ```

    - [x]  Conhecer os tipo de métodos

        GET → Buscar

        POST → Salvar

        PUT → Alterar

        DELETE → Deletar

        PATCH → Alteração específica

    - [x]  Criar rota POST

        ```tsx
        import express, { request, response } from 'express';

        const  app = express();

        app.get("/", (request,response)=>{
            return response.json({message:"Hello World - NLW04"})
        })

        app.post("/", (request,response)=>{
            return response.json({ message: "Dados salvos com sucesso"})
        })

        app.listen(3333, ()=> console.log("Server is runninhg!"))
        ```

    - [x]  Configurar o Insomnia

        Criar um novo workspace

        ![Dia%201%20-%20Fundamentos%20do%20NodeJS%20a11b13e282ad4a73b5abab85689ddef6/Untitled%201.png](Dia%201%20-%20Fundamentos%20do%20NodeJS%20a11b13e282ad4a73b5abab85689ddef6/Untitled%201.png)

        ![Dia%201%20-%20Fundamentos%20do%20NodeJS%20a11b13e282ad4a73b5abab85689ddef6/Untitled%202.png](Dia%201%20-%20Fundamentos%20do%20NodeJS%20a11b13e282ad4a73b5abab85689ddef6/Untitled%202.png)

#rumoaoproximonivel