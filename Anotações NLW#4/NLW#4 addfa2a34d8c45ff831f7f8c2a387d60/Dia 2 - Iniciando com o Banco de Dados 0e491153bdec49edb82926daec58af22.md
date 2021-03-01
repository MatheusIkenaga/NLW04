# Dia 2 - Iniciando com o Banco de Dados

- [x]  O que aprendemos ontem

    Conceitos básicos de node

    Métodos HTTP

    Criamos 2 primeiras rotas (Get e Post)

    Aprendemos o que é uma API

- [x]  O que vamos aprender hoje

    Dar inicio ao trabalho com banco de dados

- [x]  Conhecendo as formas de trabalhar com banco de dados na aplicação

    Existem 3 formas de inserir banco de dados na aplicação

    - Inserindo o **próprio driver do banco de dados** na aplicação

        Além de baixar o driver, é necessário ler documentação pois varia de banco para banco

        Pontos negativos usar driver nativo:

        - Caso precise mudar de Banco de dados (  Ir de Postgres p/ My SQL por exemplo )
        - É necessário alterar todas as consultas ao banco

    - Usando um Query Builder

        Pontos negativos de usar Query Builder

        - É necessário escrever algumas querys ( Ja utilizando algumas funções de forma mais fácil)

    - Usando um ORM

        É um mapeamento entre objetos

        Basicamente pega a nossa classe e vai mapear para uma tabela do DB

        Pontos negativos de usar ORM:

        - Em algumas ocasiões, é necessário criar querys manualmente

        Por que usar um ORM?

        - Nesse caso usaremos o TypeORM, ele trabalha muito bem com o TypeScript
        - Por ser mais genérico, caso precise mudar o driver é só alterar nas configurações

- [x]  Configurar o TypeORM na aplicação

    No terminal ( nlw/aulas/api )

    ```powershell
    yarn add typeorm reflect-metadata
    ```

    Em seguida instalar o driver do banco que irá utilizar (Documentação [aqui](https://typeorm.io/#/) )

    Nesse caso utilizaremos o SQLite ( Ele é um banco "em memória" )

    ```powershell
    yarn add sqlite3
    ```

    Criar agora o ORM config dentro da pasta api( [doc](https://typeorm.io/#/using-ormconfig) )

    ```json
    {
        "type": "sqlite",
        "database": "./src/database/database.sqlite"
    }
    ```

    Criar a pasta "database" dentro de src

    e criar um arquivo "index.ts"

    ```tsx
    import { createConnection } from 'typeorm'

    createConnection();
    ```

    Dentro do server.ts, importar o reflect-metadata e o database:

    ```tsx
    import 'reflect-metadata';
    import express, { request, response } from 'express';
    import "./database";

    const  app = express();

    ...
    ```

    Testar → yarn dev

    Após isso será criado automaticamente o arquivo database.sqlite 

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled.png)

- [x]  Criar migration de usuário

    Uma migration serve para que as alterações do banco fiquem dentro do próprio projeto

    É como se fosse um histórico de tudo do banco de dados

    Package.json → Adicionar dentro de script o "typeorm"

    ```json
    "scripts": {
        "dev": "ts-node-dev --transpile-only --ignore-watch node_modules src/server.ts",
        "typeorm": "ts-node-dev node_modules/typeorm/cli.js"
      },
    ```

    Criar pasta migration dentro de database

    Ir no arquivo ormconfig.json

    ```json
    {
        "type": "sqlite",
        "database": "./src/database/database.sqlite",
        "migrations": ["./src/database/migrations/**.ts"],
        "cli":{
            "migrationsDir": "./src/database/migrations"
        }
    }
    ```

    Criar a primeira migration:

    no terminal

    ```powershell
    yarn typeorm migration:create -n CreateUsers
    ```

    Ir no arquivo da Migration "CreateUsers"

    ```tsx
    import {MigrationInterface, QueryRunner, Table} from "typeorm";

    export class CreateUsers1614214211975 implements MigrationInterface {

        public async up(queryRunner: QueryRunner): Promise<void> {
            await queryRunner.createTable(
                new Table({
                    name: "users",
                    columns: [
                        {
                            name:"id",
                            type: "uuid",
                            isPrimary: true
                        },
                        {
                            name: "name",
                            type: "varchar",
                        },
                        {
                            name: "email",
                            type: "varchar"
                        },
                        {
                            name: "created_at",
                            type: "timestamp",
                            default: "now()"
                        }
                    ]
                })
            )
        }

        public async down(queryRunner: QueryRunner): Promise<void> {
            await queryRunner.dropTable("users");
        }

    }
    ```

    para rodar → terminal

    ```powershell
    yarn typeorm migration:run
    ```

    Para testar, instale a extensão do VS Code "sqlite"

    Ctrl + Shift + P

    - Ctrl + Shift + P
    - sqlite: Open database

    Visualize aqui:

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%201.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%201.png)

    Ou utilizar o [Beekeeper Studio](https://www.beekeeperstudio.io/)

    Caso seja necessário dar um *rollback* na **ultima** migration

    ```powershell
    yarn typeorm migration:revert
    ```

- [x]  Criar controller do usuário

    Criar dentro da src a pasta controllers

    - Em seguida criar o arquivo "UserController.ts"

    ```tsx
    import { Request, Response } from 'express'

    class UserController{

        async create(request: Request ,response: Response){
            const body = request.body;
            console.log(body)
            return response.send()
        }
    }

    export { UserController}
    ```

    Esse foi apenas um teste para ver se funciona

- [x]  Criar rota do usuário

    Após isso, precisamos chamar esse Controller no nosso server.ts

    MAAAAAASSS para ficar melhor divido os serviços das rotas, será criado um arquivo routes.ts dentro da src

    routes.ts:

    ```tsx
    import { Router } from 'express';
    import { UserController } from "./controllers/UserController"

    const router = Router();

    const userController = new UserController();

    router.post("/users", userController.create)

    export { router }
    ```

    server.ts:

    ```tsx
    import express, { request, response } from 'express';
    import "./database";
    import { router } from './routes';

    const  app = express();

    app.use(router)

    app.listen(3333, ()=> console.log("Server is runninhg!"))
    ```

    Rodar aplicação → yarn dev

    Ao testar, enviando uma requisição pelo Insomnia para o nosso server, o console.log vai dar undefined

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%202.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%202.png)

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%203.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%203.png)

    isso acontece pqe o Body não recebe informações apenas em JSON

    Precisamos informar isso para o nosso server

    server.ts:

    ```tsx
    import express, { request, response } from 'express';
    import "./database";
    import { router } from './routes';

    const  app = express();

    **app.use(express.json())**   // <- HERE
    app.use(router)

    app.listen(3333, ()=> console.log("Server is runninhg!"))
    ```

    Após isso, o console.log consegue printar o resultado:

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%204.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%204.png)

- [x]  Criar model de usuário

    Criar pasta "models" dentro de src

    - Em seguida criar o arquivo "User.ts"

    ```tsx
    import { Entity } from "typeorm";

    @Entity("users")
        //@Entity é a tipagem do TypeScript entrando em ação
        //users é o nome da tabela do banco
        
    class User{

    }

    export { User }
    ```

    Ir no arquivo tsconfig.json :

    "descomentar" essas duas linhas:  "experimentalDecorators" e "emitDecoratorMetadata"

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%205.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%205.png)

    "descomentar" essa linha e trocar para false

    ![Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%206.png](Dia%202%20-%20Iniciando%20com%20o%20Banco%20de%20Dados%200e491153bdec49edb82926daec58af22/Untitled%206.png)

    no arquivo ormconfig.json

    ```json
    {
        "type": "sqlite",
        "database": "./src/database/database.sqlite",
        "migrations": ["./src/database/migrations/**.ts"],
        "entities" : ["./src/models/**.ts"],
        "cli":{
            "migrationsDir": "./src/database/migrations"
        }
    }
    ```

    No arquivo User.ts

    ```tsx
    import { Column, CreateDateColumn, Entity, PrimaryColumn } from "typeorm";

    @Entity("users")
        //@Entity é a tipagem do TypeScript entrando em ação
        //users é o nome da tabela do banco

    class User{

        @PrimaryColumn()
        id: string;

        @Column()
        name: string;

        @Column()
        email: string;

        @CreateDateColumn()
        created_at: Date;

    }

    export { User }
    ```

    A responsabilidade de criar o UUID será do código e não do banco

    Por isso...

    ```powershell
    yarn add uuid
    yarn add @types/uuid -D
    ```

    No User.ts

    ```tsx
    import { Column, CreateDateColumn, Entity, PrimaryColumn } from "typeorm";
    import { v4 as uuid } from 'uuid'

    @Entity("users")
        //@Entity é a tipagem do TypeScript entrando em ação
        //users é o nome da tabela do banco

    class User{

        @PrimaryColumn()
        readonly id: string;

        @Column()
        name: string;

        @Column()
        email: string;

        @CreateDateColumn()
        created_at: Date;

        constructor(){
            if(!this.id){
                this.id = uuid()
            }
        }

    }

    export { User }
    ```

    No Controller:

    ```tsx
    import { Request, Response } from 'express'
    import { getRepository } from 'typeorm';
    import { User } from "../models/User"

    class UserController{

        async create(request: Request ,response: Response){
            const { name, email } = request.body;

            const usersRepository = getRepository(User)

            // SELECT * FROM USERS WHEER EMAIL = "EMAIL"
            const userAlreadyExists = await usersRepository.findOne({
                email
            });
            if(userAlreadyExists){
                return response.status(400).json({
                    error: "User already exists!"
                })            
            }

            const user = usersRepository.create({
                name, email
            })

            await usersRepository.save(user)

            return response.json(user)
        }
    }

    export { UserController}
    ```

    OBS: para saber qual query está indo para o banco, podemos colocar o logging no ormconfig

    ```json
    {
        "type": "sqlite",
        "database": "./src/database/database.sqlite",
        "migrations": ["./src/database/migrations/**.ts"],
        "entities" : ["./src/models/**.ts"],
        "logging": true,
        "cli":{
            "migrationsDir": "./src/database/migrations"
        }
    }
    ```

#jornadainfinita