# Dia 3 - Testando a nossa aplicação

- [x]  O que aprendemos ontem

    Criamos o primeiro controller

    Criamos a Migration

    Acesso ao DB

    Entendemos o que são Entidades

- [x]  O que vamos aprender hoje

    Refatorar nosso código

    Separar as responsabilidades

    Dar início aos testes

    Criar tabela de pesquisa

- [x]  Refatorar nosso controller

    Criar dentro da pasta src a pasta repositories

    - [x]  Criar um repository de usuário

        Em seguida criar o arquivo "UsersRepository.ts"

        ```tsx
        import { Entity, EntityRepository, Repository } from "typeorm";
        import { User } from "../models/User";

        @EntityRepository(User)
        class UsersRepository extends Repository<User>{}

        export { UsersRepository }
        ```

    - [x]  Alterar no controller para o repository criado

        No UserController, tirar o getRepository e colocar getCustomRepository()

        ```tsx
        import { Request, Response } from 'express'
        import { getRepository } from 'typeorm';
        import { User } from "../models/User"
        import { UsersRepository } from '../repositories/UsersRepository';

        class UserController{

            async create(request: Request ,response: Response){
                const { name, email } = request.body;

                const usersRepository = getCustomRepository(UsersRepository)

        ...
        ```

        Em seguida, retirar os Imports não utilizados ( Alt + Shift + O )

        UserController.ts:

        ```tsx
        import { Request, Response } from 'express';
        import { getCustomRepository } from 'typeorm';
        import { UsersRepository } from '../repositories/UsersRepository';

        class UserController{

            async create(request: Request ,response: Response){
                const { name, email } = request.body;

                const usersRepository = getCustomRepository(UsersRepository)

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

        export { UserController };
        ```

- [x]  Criar migration de pesquisas (survey)

    No terminal ( nlw/aulas/api )

    ```powershell
    yarn typeorm migration:create -n CreateSurveys
    ```

    na migration da Surveys

    ```tsx
    import {MigrationInterface, QueryRunner, Table} from "typeorm";

    export class CreateSurveys1614292563735 implements MigrationInterface {

        public async up(queryRunner: QueryRunner): Promise<void> {
            await queryRunner.createTable(
                new Table({
                    name: "surveys",
                    columns: [
                        {
                            name: "id",
                            type: "uuid",
                            isPrimary: true
                        },
                        {
                            name: "title",
                            type: "varchar"
                        },
                        {
                            name: "description",
                            type: "varchar"
                        },
                        {
                            name: "created_at",
                            type: "timestamp",
                            default: "now()",
                        }
                    ],
                })
            )
        }
        

        public async down(queryRunner: QueryRunner): Promise<void> {
            await queryRunner.dropTable("surveys")
        }

    }
    ```

    Rodar a migration

    ```powershell
    yarn typeorm migration:run
    ```

    Em seguida, criar o Model de pesquisas

    src/models criar um arquivo Survey.ts

    ```tsx
    import { Column, CreateDateColumn, Entity, PrimaryColumn } from "typeorm";
    import { v4 as uuid } from 'uuid'

    @Entity("surveys")

    class Survey{
        @PrimaryColumn()
        readonly id: string;

        @Column()
        title: string;

        @Column()
        description: string;

        @CreateDateColumn()
        created_at: Date;

        constructor(){
            if(!this.id){
                this.id = uuid()
            }
        }
    }

    export { Survey }
    ```

- [x]  Criar repository de pesquisas

    SurveysRepository.ts

    ```tsx
    import { Entity, EntityRepository, Repository } from "typeorm";
    import { Survey } from "../models/Survey";

    @EntityRepository(Survey)
    class SurveysRepository extends Repository<Survey> {}

    export { SurveysRepository }
    ```

- [x]  Criar controller de pesquisas

    SurveyController.ts

    ```tsx
    import { Request, Response } from 'express'
    import { SurveysRepository } from '../repositories/SurveysRepository';

    class SurveyController{
        async create ( request: Request, response: Response){
            const { title, description } = request.body;

            const surveysRepository = getCustomRepository(SurveysRepository)

            const survey = surveysRepository.create({
                title,
                description
            })

            await surveysRepository.save(survey)

            return response.status(201).json(survey)
        }
    }

    export { SurveyController }
    ```

    Em seguida criar uma rota para o surveyController

    routes.ts:

    ```tsx
    import { Router } from 'express';
    import { UserController } from "./controllers/UserController"
    import { SurveyController } from "./controllers/SurveyController"

    const router = Router();

    const userController = new UserController();
    const surveyController = new SurveyController();

    router.post("/users", userController.create)
    router.post("/surveys", surveyController.create)

    export { router }
    ```

    Depois rodar aplicação → yarn dev

    Aproveitando...

    Vamos criar mais um método dentro do SurveyController para listar as surveys

    ```tsx
    import { Request, Response } from 'express'
    import { getCustomRepository } from 'typeorm';
    import { SurveysRepository } from '../repositories/SurveysRepository';

    class SurveyController{
        async create ( request: Request, response: Response){
            const { title, description } = request.body;

            const surveysRepository = getCustomRepository(SurveysRepository)

            const survey = surveysRepository.create({
                title,
                description
            })

            await surveysRepository.save(survey)

            return response.status(201).json(survey)
        }
        async show(request:Request, response: Response){
            const surveysRepository = getCustomRepository(SurveysRepository)

            const all = await surveysRepository.find()

            return response.json(all)
        }

    }

    export { SurveyController }
    ```

    adicionar o método nas routes

    ```tsx
    import { Router } from 'express';
    import { UserController } from "./controllers/UserController"
    import { SurveyController } from "./controllers/SurveyController"

    const router = Router();

    const userController = new UserController();
    const surveyController = new SurveyController();

    router.post("/users", userController.create)
    router.post("/surveys", surveyController.create)
    router.get("/surveys", surveyController.show)

    export { router }
    ```

---

- [x]  O que são testes automatizados?

    Tipos:

    - Testes unitários

        Geralmente utilizado ao fazer o TDD

        TDD é começar a desenvolver o código orientado à testes

        Criamos nessa ordem Migration, Model, Repository e o Controller

        Provavelmente será criado repository fake, etc para testar a aplicação

    - Testes de integração

        Testar a funcionalidade completa

        Criação de usuário

        Simular acesso, como por exemplo:

        → Request → Routes → Controller → Repository

        ← Repository ← Controller ← Response

        Ou seja, testar todo o fluxo da aplicação

    - Testes ponta a ponta (E3E) end-to-end

        Mais utilizado em aplicações front-end

        Testar desde o preenchimento do campo, recarregar a página, tudo

        Para aplicações backend, são utilizados os testes unitários e de integração

- [x]  Criar o primeiro teste

    Primeiro, vamos instalar as ferramentas #jest

    Jest ( [Docs](https://jestjs.io/) )

    ```powershell
    yarn add jest @types/jest -D
    ```

    Criar arquivo de configuração do jest

    ```powershell
    yarn jest --init
    ```

    Em seguida haverão estas perguntas:

    ![Dia%203%20-%20Testando%20a%20nossa%20aplicac%CC%A7a%CC%83o%20c7ca925a4a5e4797b620cf4b148d7d69/Untitled.png](Dia%203%20-%20Testando%20a%20nossa%20aplicac%CC%A7a%CC%83o%20c7ca925a4a5e4797b620cf4b148d7d69/Untitled.png)

    Será criado um arquivo jest.config.ts

    Descomentar o bail e sinalizar como true

    ![Dia%203%20-%20Testando%20a%20nossa%20aplicac%CC%A7a%CC%83o%20c7ca925a4a5e4797b620cf4b148d7d69/Untitled%201.png](Dia%203%20-%20Testando%20a%20nossa%20aplicac%CC%A7a%CC%83o%20c7ca925a4a5e4797b620cf4b148d7d69/Untitled%201.png)

    O que o bail faz?

    Por padrão, quando está comentado, ao rodar o script de testes caso o jest pegue algum erro, ele continua a rodar o script

    Como estamos trabalhando com testes, sinalizando ele como True, qualquer erro que ocorra ele irá parar de rodar o script

    Comentar o testEnvironment

    ```tsx
    // testEnvironment: "node",
    ```

    No testMatch → Descomentar

    Esse é o caminho onde iremos colocar nossos testes

    ```tsx
    testMatch: [
        "**/__tests__/*.test.ts"
      ],
    ```

    Criar na src a pasta "__tests__"

    E dentro dela o arquivo "First.test.ts"

    Em seguida no terminal

    ```tsx
    yarn add ts-jest -D
    ```

    No jest.config.ts

    Descomentar a linha do preset e colocar como "ts-jest"

    ```tsx
    preset: "ts-jest",

    ```

    Novamente no arquivo First.test.js

    ---

    ### Testando o script de testes

    **(*É só para fim de testar o script de testes*)**

    ```tsx
    describe("First", ()=>{

        it("Deve ser possível somar 2 números", ()=>{

            expect(2 + 2). toBe(4)
        })
        it("Deve ser possível somar 2 números", ()=>{

            expect(2 + 2). toBe(5)
        })
    })
    ```

    Para rodar → 

    ```tsx
    yarn test
    ```

    O resultado será

    O primeiro It vai passar, afinal 2+2 = 4

    O segundo deve dar erro pois 2+2 ≠ 5

    ![Dia%203%20-%20Testando%20a%20nossa%20aplicac%CC%A7a%CC%83o%20c7ca925a4a5e4797b620cf4b148d7d69/Untitled%202.png](Dia%203%20-%20Testando%20a%20nossa%20aplicac%CC%A7a%CC%83o%20c7ca925a4a5e4797b620cf4b148d7d69/Untitled%202.png)

    Também podemos dizer que o 2+2 NÃO DEVE dar 5

    ```tsx
    describe("First", ()=>{

        it("Deve ser possível somar 2 números", ()=>{

            expect(2 + 2). toBe(4)
        })
        it("Deve ser possível somar 2 números", ()=>{

            expect(2 + 2)**.not.**toBe(5) // <- HERE
        })
    })
    ```

    Após isso, podemos remover esse arquivo para darmos inicio aos nossos testes de integração

    ---

    ### Dando inicio aos testes de integração:

    Caso tenha pulado o começo das instalações, é necessário a instalação do Jest, procure por *#jest* nesse arquivo

    Criar o arquivo "User.test.ts"

    Precisamos instalar mais uma ferramenta para simular o servidor inicializado, para isso utilizaremos o [**supertest**](https://www.npmjs.com/package/supertest)

    ```powershell
    yarn add @supertest @types/supertest -D  
    (No meu não funcionou com o yarn)

    npm install supertest --save-dev
    ```

    deixar o server.ts assim:

    ```tsx
    import { app } from "./app";

    app.listen(3333, ()=> console.log("Server is runninhg!"))
    ```

    e criar o app.js na srv

    ```tsx
    import 'reflect-metadata';
    import express, { request, response } from 'express';
    import "./database";
    import { router } from './routes';

    const  app = express();

    app.use(express.json())
    app.use(router)

    export { app }
    ```

    Na User.test.ts

    ```tsx
    import request from 'supertest'
    import { app } from '../app'

    describe("User", ()=>{
        request(app).post("/users")
        .send({
            email: "user@example.com",
            name: "User Example"
        })
    })
    ```

    Como precisamos testar o fluxo completo de dados, nós não queremos que esses testes sejam feitos no mesmo banco que é utilizado em produção

    Para isso, precisamos criar um banco para testes

    ./src/database/index.ts/

    ```tsx
    import { Connection, createConnection } from 'typeorm'

    export default async() : Promise<Connection> => {
        
        return createConnection();
    }
    ```

    Precisamos saber qual ambiente estamos rodando a aplicação, se é de teste, se é de dev, de produção

    Variáveis de ambiente

    No package.json

    ```json
    "scripts": {
        "dev": "ts-node-dev --transpile-only --ignore-watch node_modules src/server.ts",
        "typeorm": "ts-node-dev node_modules/typeorm/cli.js",
        "test": "NODE_ENV=test jest"
      },
    ```

    Esse NODE_ENV, manda no banco de dados que é ambiente de testes

    index.ts

    ```tsx
    import { Connection, createConnection, getConnectionOptions } from 'typeorm'

    export default async() : Promise<Connection> => {
        const defaultOptions = await getConnectionOptions()
        return createConnection(
            Object.assign(defaultOptions, {
                database: 
                process.env.NODE_ENV === 'test' 
                    ? "./src/database/database.test.sqlite" 
                    : defaultOptions.database
            })
        );
    }
    ```

    Foi utilizado um if ternário → Explicação

    ```tsx
    *process.env.NODE_ENV === 'test' 
                    ? "./src/database/database.test.sqlite" 
                    : defaultOptions.database

    //É como se fosse isso aqui:

    if (process.env.NODE_ENV === 'test' ){
    	"./src/database/database.test.sqlite" 
    }else{
    	defaultOptions.database
    }*
    ```

    Após isso

    ir no app.ts

    ```tsx
    import express from 'express';
    import 'reflect-metadata';
    import createConnection from "./database";
    import { router } from './routes';

    createConnection()
    const  app = express();

    app.use(express.json())
    app.use(router)

    export { app };
    ```

    // Após isso eu precisei dar um  " yarn add reflect-metadata " novamente

    No UserController.ts

    ```tsx
    ...

    return response**.status(201)**.json(user)
        }
    }

    export { UserController };
    ```

    No User.test.ts

    ```tsx
    import request from 'supertest'
    import { app } from '../app'

    import createConnection from '../database'

    describe("User",()=>{
        beforeAll(async()=>{
            const connection = await createConnection()
            await connection.runMigrations()
        })

        it("Should be able to create a new user", async()=>{
            const response = await request(app).post("/users").send({
                email: "user@example.com",
                name: "User Example"
            })
        expect(response.status).toBe(201)
        })
        
    })
    ```

    Se rodar um yarn test

    Ele vai criar o bd, rodar o script e verificar se o status retornou 201

    Agora para verificar a parte de enviar 2 vezes o mesmo email para cadastro

    ```tsx
    import request from 'supertest'
    import { app } from '../app'

    import createConnection from '../database'

    describe("User",()=>{
        beforeAll(async()=>{
            const connection = await createConnection()
            await connection.runMigrations()
        })

        it("Should be able to create a new user", async()=>{
            const response = await request(app).post("/users").send({
                email: "user@example.com",
                name: "User Example"
            })
        expect(response.status).toBe(201)
        });

        ***it("Should not be able to create a user with exists email", async()=>{
            const response = await request(app).post("/users").send({
                email: "user@example.com",
                name: "User Example"
            })
        expect(response.status).toBe(400)
        });***

    })
    ```

    Ao rodar novamente, o primeiro teste falha e o segundo passa

    Pois o registro ja foi criado no primerio yarn test

    Precisamos que sempre que finalize de rodar o teste, o banco seja dropado

    No package.json:

    ```json
    "scripts": {
        "dev": "ts-node-dev --transpile-only --ignore-watch node_modules src/server.ts",
        "typeorm": "ts-node-dev node_modules/typeorm/cli.js",
        "test": "NODE_ENV=test jest",
        "posttest": "rm ./src/database/database.test.sqlite"
      },

    ```

    Finalizado o teste de Usuário

    Teste de pesquisa:

    criar arquivo Surveys.test.ts

    ```tsx
    import request from 'supertest'
    import { app } from '../app'

    import createConnection from '../database'

    describe("Surveys",()=>{
        beforeAll(async()=>{
            const connection = await createConnection()
            await connection.runMigrations()
        })

        it("Should be able to create a new survey", async()=>{
            const response = await request(app).post("/surveys").send({
                title: "Title Example",
                description: "Description Example"
            })
        expect(response.status).toBe(201)
        expect(response.body).toHaveProperty("id")
        });

        it("Should be able to get all surveys", async()=>{
            await request(app).post("/surveys").send({
                title: "Title Example 2",
                description: "Description Example 2"
            })
            
            const response = await request(app).get("/surveys")

            expect(response.body.length).toBe(2)

        })

    })
    ```

    #focopraticagrupo