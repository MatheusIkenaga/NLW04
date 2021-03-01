# Dia 4 - Envio de e-mail

- [x]  O que aprendemos ontem

    Teste automatizados

    Testes de integração

    Variáveis de ambiente

- [x]  O que vamos aprender hoje

    Funcionalidade de envio de email

- [x]  Criar migration de SurveysUsers

    No terminal

    ```powershell
    yarn typeorm migration:create -n CreateSurveysUsers
    ```

    na CreateSurveysUsers

    ```tsx
    import {MigrationInterface, QueryRunner, Table} from "typeorm";

    export class CreateSurveysUsers1614380875556 implements MigrationInterface {

        public async up(queryRunner: QueryRunner): Promise<void> {
            await queryRunner.createTable(
                new Table({
                    name:"surveys_users",
                    columns: [
                        {
                            name: "id",
                            type: "uuid",
                            isPrimary: true
                        },
                        {
                            name: "user_id",
                            type: "uuid",
                        },
                        {
                            name: "survey_id",
                            type: "uuid"
                        },
                        {
                            name: "value",
                            type: "number",
                            isNullable: true,

                        },
                        {
                            name: "created_at",
                            type: "timestamp",
                            default:"now()"
                        },
                    ],

                    foreignKeys:[
                        {
                            name: "FKUser",
                            referencedTableName: "users",
                            referencedColumnNames: ["id"],
                            columnNames: ["user_id"],
                            onDelete: "CASCADE",
                            onUpdate:"CASCADE"
                        },
                        {
                            name: "FKSurvey",
                            referencedTableName: "surveys",
                            referencedColumnNames: ["id"],
                            columnNames: ["survey_id"],
                            onDelete: "CASCADE",
                            onUpdate:"CASCADE"
                        }
                    ]

                })
            )
        }

        public async down(queryRunner: QueryRunner): Promise<void> {
            await queryRunner.dropTable("surveys_users")
        }

    }
    ```

    executar:

    ```powershell
    yarn typeorm migration:run
    ```

- [x]  Criar Model

    criar arquivo SurveyUser na pasta model

    ```tsx
    import { Column, CreateDateColumn, Entity, PrimaryColumn } from "typeorm";
    import { v4 as uuid } from 'uuid'

    @Entity("surveys_users")
    class SurveyUser{
        @PrimaryColumn()
        readonly id: string;

        @Column()
        user_id: string;

        @Column()
        survey_id: string;

        @Column()
        value: number;

        @CreateDateColumn()
        created_at: Date;

        constructor(){
            if(!this.id){
                this.id = uuid()
            }
        }
    }

    export { SurveyUser }
    ```

- [x]  Criar repositório

    criar o SurveysUsersRepository.ts

    ```tsx
    import { EntityRepository, Repository } from "typeorm";
    import { SurveyUser } from "./SurveyUser";

    @EntityRepository(SurveyUser)
    class SurveysUsersRepository extends Repository<SurveyUser>{}

    export { SurveysUsersRepository }
    ```

- [x]  Criar controller

    Criar arquivo SendMailController.ts , afinal quem vai criar uma pesquisa é alguém da empresa

    Primeiro vamos testar salvar na tabela as informações

    ```tsx
    import { Request, Response } from "express";
    import { getCustomRepository } from "typeorm";
    import { SurveysRepository } from "../repositories/SurveysRepository";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";
    import { UsersRepository } from "../repositories/UsersRepository";

    class SendMailController{
        async execute(request: Request, response: Response){
            const { email, survey_id } = request.body

            const usersRepository = getCustomRepository(UsersRepository)
            const surveyRepository = getCustomRepository(SurveysRepository)
            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const userAlreadyExists = await usersRepository.findOne({email})
            if(!userAlreadyExists){
                return response.status(400).json({
                    error: "User does not exists"
                })
            }

            const surveyAlreadyExists = await surveyRepository.findOne({id: survey_id})
            if(!surveyAlreadyExists){
                return response.status(400).json({
                    error: "Survey does not exists"
                })
            }

            //Salvar as informações na tabela surveyUser
            const surveyUser = surveysUsersRepository.create({
                user_id: userAlreadyExists.id,
                survey_id
            })
            await surveysUsersRepository.save(surveyUser)

            //Enviar e-mail para o usuário

    				return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    Em seguida criar a Rota:

    ```tsx
    import { Router } from 'express';
    import { UserController } from "./controllers/UserController"
    import { SurveyController } from "./controllers/SurveyController"
    import { SendMailController } from './controllers/SendMailController';

    const router = Router();

    const userController = new UserController();
    const surveyController = new SurveyController();

    const sendMailController = new SendMailController()

    router.post("/users", userController.create)
    router.post("/surveys", surveyController.create)
    router.get("/surveys", surveyController.show)

    router.post("/sendMail", sendMailController.execute)

    export { router }
    ```

    Em seguida rodar a aplicação para testar

    ```tsx
    yarn dev
    ```

    No Insomnia:

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled.png)

- [x]  Criar serviço de e-mail

    usaremos a biblioteca [NodeMailer](https://nodemailer.com/about/) e tbm precisaremos de um SMTP o [Ethereal](https://ethereal.email/) (É um fake SMTP )

    ```powershell
    yarn add nodemailer
    yarn add @types/nodemailer -D
    ```

    Criar a pasta services dentro de src

    e o arquivo SendMailService.ts

    ```tsx
    import nodemailer, { Transporter } from 'nodemailer'

    class SendMailService{
        
        private client: Transporter

        constructor(){
            nodemailer.createTestAccount().then(account =>{
                const transporter = nodemailer.createTransport({
                    host: account.smtp.host,
                    port: account.smtp.port,
                    secure: account.smtp.secure,
                    auth: {
                        user: account.user,
                        pass: account.pass
                    }
                });

                this.client = transporter

            })
        }

        /**
         * O .then() deixa a resposta dentro dele, como se fosse:
         * const resposta = await execute()
         * usamos isso pois o constructor não permite ser async
         */
        
        async execute(to:string, subject:string, body:string){

            const message = await this.client.sendMail({
                to,
                subject,
                html: body,
                from: "NPS <noreply@nps.com.br>"
            })

            console.log('Message sent: %s', message.messageId);
            console.log('Preview URL: %s', nodemailer.getTestMessageUrl(message));

        }
    }

    export default new SendMailService()
    ```

    Ir no SendMailController

    ```tsx
    import { Request, Response } from "express";
    import { getCustomRepository } from "typeorm";
    import { SurveysRepository } from "../repositories/SurveysRepository";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";
    import { UsersRepository } from "../repositories/UsersRepository";
    import SendMailService from "../services/SendMailService";

    class SendMailController{
        async execute(request: Request, response: Response){
            const { email, survey_id } = request.body

            const usersRepository = getCustomRepository(UsersRepository)
            const surveyRepository = getCustomRepository(SurveysRepository)
            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const userAlreadyExists = await usersRepository.findOne({email})
            if(!userAlreadyExists){
                return response.status(400).json({
                    error: "User does not exists"
                })
            }

            const survey = await surveyRepository.findOne({id: survey_id})
            if(!survey){
                return response.status(400).json({
                    error: "Survey does not exists"
                })
            }

            //Salvar as informações na tabela surveyUser
            const surveyUser = surveysUsersRepository.create({
                user_id: userAlreadyExists.id,
                survey_id
            })
            await surveysUsersRepository.save(surveyUser)

            //Enviar e-mail para o usuário

            await SendMailService.execute(email,survey.title, survey.description)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

- [x]  Enviar e-mail

    Para testar → yarn dev

    Send na mesma requisição SendMail do Insomnia ( Vai demorar um pouco mesmo )

    No console irá retornar um link com o email enviado:

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%201.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%201.png)

    indo no link, podemos ver:

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%202.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%202.png)

    Para customizar o HTML, iremos utilizar o [handlebars](https://handlebarsjs.com/) 

    ```powershell
    yarn add handlebars
    ```

    para criar nosso template, criar pasta views dentro de src

    dentro da views, criar a pasta emails

    e dentro da emails criar arquivo npsMail.hbs

    ```html
    <style>
        .container{
            width: 800px;
            justify-content:center;
            align-items: center;
            align-content: center;
            display: flex;
            flex-direction: column;
        }

        .value{
            padding: 10px;
            background: #8257e6;
            color: #FFF;
            border-radius: 6px;
            width: 15px;
            text-align: center;
            text-decoration: none;
        }

        .level{
            display: flex;
            margin: 10px;
            justify-content: space-between;
            width: 350px;
        }

        .answers{
            width: 350px;
            display: flex;
            justify-content: space-between;
        }

    </style>

    <div class="container">
        <label> Olá, <strong>{{name}}</strong>! Tudo bem?</label>

        <h3>{{Title}}</h3>
        <br>
        <strong>{{description}}</strong>

        <div class="level">
            <span>Pouco provável</span>
            <span>Muito provável</span>
        </div>

        <div class="answers">
            <a class="value" href="">1</a>
            <a class="value" href="">2</a>
            <a class="value" href="">3</a>
            <a class="value" href="">4</a>
            <a class="value" href="">5</a>
            <a class="value" href="">6</a>
            <a class="value" href="">7</a>
            <a class="value" href="">8</a>
            <a class="value" href="">9</a>
            <a class="value" href="">10</a>
        </div>

        </br>
        </br>

        <strong>Sua opinião é muito importante para nós</strong>
        <h3>Equipe | <strong>NLW</strong></h3>

    </div>
    ```

    ir na SendMailService.ts

    ```tsx
    import nodemailer, { Transporter } from 'nodemailer'
    import { resolve } from 'path'
    import handlebars from 'handlebars'
    import fs from 'fs'

    class SendMailService{
        
        private client: Transporter

        constructor(){
            nodemailer.createTestAccount().then(account =>{
                const transporter = nodemailer.createTransport({
                    host: account.smtp.host,
                    port: account.smtp.port,
                    secure: account.smtp.secure,
                    auth: {
                        user: account.user,
                        pass: account.pass
                    }
                });

                this.client = transporter

            })
        }

        /**
         * O .then() deixa a resposta dentro dele, como se fosse:
         * const resposta = await execute()
         * usamos isso pois o constructor não permite ser async
         */
        
        async execute(to:string, subject:string, body:string){

            const npsPath = resolve(__dirname, "..", "views", "emails", "npsMail.hbs")
            const templateFileContent = fs.readFileSync(npsPath).toString("utf8")

            const mailTemplateParse = handlebars.compile(templateFileContent)

            const html = mailTemplateParse({
                name: to,
                title: subject,
                description: body
            })

            const message = await this.client.sendMail({
                to,
                subject,
                html,
                from: "NPS <noreply@nps.com.br>"
            })

            console.log('Message sent: %s', message.messageId);
            console.log('Preview URL: %s', nodemailer.getTestMessageUrl(message));

        }
    }

    export default new SendMailService()
    ```

    testando → yarn dev

    Send no Insomnia:

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%203.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%203.png)

    Para melhorar

    SendMailController.ts

    ```tsx
    import { Request, Response } from "express";
    import { resolve } from 'path'
    import { getCustomRepository } from "typeorm";
    import { SurveysRepository } from "../repositories/SurveysRepository";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";
    import { UsersRepository } from "../repositories/UsersRepository";
    import SendMailService from "../services/SendMailService";

    class SendMailController{
        async execute(request: Request, response: Response){
            const { email, survey_id } = request.body

            const usersRepository = getCustomRepository(UsersRepository)
            const surveyRepository = getCustomRepository(SurveysRepository)
            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const user = await usersRepository.findOne({email})
            if(!user){
                return response.status(400).json({
                    error: "User does not exists"
                })
            }

            const survey = await surveyRepository.findOne({id: survey_id})
            if(!survey){
                return response.status(400).json({
                    error: "Survey does not exists"
                })
            }

            //Salvar as informações na tabela surveyUser
            const surveyUser = surveysUsersRepository.create({
                user_id: user.id,
                survey_id
            })

            
            await surveysUsersRepository.save(surveyUser)

            //Enviar e-mail para o usuário

            const npsPath = resolve(__dirname, "..", "views", "emails", "npsMail.hbs")

            const variables = {
                name: user.name,
                title: survey.title,
                description: survey.description
            }

            await SendMailService.execute(email, survey.title, variables, npsPath)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    SendMailService.ts

    ```tsx
    import nodemailer, { Transporter } from 'nodemailer'
    import handlebars from 'handlebars'
    import fs from 'fs'

    class SendMailService{
        
        private client: Transporter

        constructor(){
            nodemailer.createTestAccount().then(account =>{
                const transporter = nodemailer.createTransport({
                    host: account.smtp.host,
                    port: account.smtp.port,
                    secure: account.smtp.secure,
                    auth: {
                        user: account.user,
                        pass: account.pass
                    }
                });

                this.client = transporter

            })
        }

        /**
         * O .then() deixa a resposta dentro dele, como se fosse:
         * const resposta = await execute()
         * usamos isso pois o constructor não permite ser async
         */
        
        async execute(to:string, subject:string, variables:object, path: string){

            const templateFileContent = fs.readFileSync(path).toString("utf8")

            const mailTemplateParse = handlebars.compile(templateFileContent)

            const html = mailTemplateParse(variables)

            const message = await this.client.sendMail({
                to,
                subject,
                html,
                from: "NPS <noreply@nps.com.br>"
            })

            console.log('Message sent: %s', message.messageId);
            console.log('Preview URL: %s', nodemailer.getTestMessageUrl(message));

        }
    }

    export default new SendMailService()
    ```

    Testando → yarn dev → Send no Insomnia:

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%204.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%204.png)

    FICOU BALA

    Agora falta receber a informação de quando o cliente clicar na pontuação

    a ideia é que o link que receba a informação seja por exemplo

    http://localhost:3333/answers/${nota}?u={id_usuario} 

    npsMail.hbs?

    ```html
    <style>
        .container{
            width: 800px;
            justify-content:center;
            align-items: center;
            align-content: center;
            display: flex;
            flex-direction: column;
        }

        .value{
            padding: 10px;
            margin: 3px;
            background: #8257e6;
            color: #FFF;
            border-radius: 6px;
            width: 15px;
            text-align: center;
            text-decoration: none;
        }

        .level{
            display: flex;
            margin: 10px;
            justify-content: space-between;
            width: 350px;
        }

        .answers{
            width: 350px;
            display: flex;
            justify-content: space-between;
        }

    </style>

    <div class="container">
        <label> Olá, <strong>{{name}}</strong>! Tudo bem?</label>

        <h3>{{Title}}</h3>
        <br>
        <strong>{{description}}</strong>

        <div class="level">
            <span>Pouco provável</span>
            <span>Muito provável</span>
        </div>

        <div class="answers">
            <a class="value" href="{{link}}/1?u={{user_id}}">1</a>
            <a class="value" href="{{link}}/2?u={{user_id}}">2</a>
            <a class="value" href="{{link}}/3?u={{user_id}}">3</a>
            <a class="value" href="{{link}}/4?u={{user_id}}">4</a>
            <a class="value" href="{{link}}/5?u={{user_id}}">5</a>
            <a class="value" href="{{link}}/6?u={{user_id}}">6</a>
            <a class="value" href="{{link}}/7?u={{user_id}}">7</a>
            <a class="value" href="{{link}}/8?u={{user_id}}">8</a>
            <a class="value" href="{{link}}/9?u={{user_id}}">9</a>
            <a class="value" href="{{link}}/10?u={{user_id}}">10</a>
        </div>

        </br>
        </br>

        <strong>Sua opinião é muito importante para nós</strong>
        <h3>Equipe | <strong>NLW</strong></h3>

    </div>
    ```

    criar um arquivo na raiz do projeto ( /nlw4/aulas/api ) como ".env"

    ```
    URL_MAIL=http://localhost:3333/answers
    ```

    No SendMailController.ts

    ```tsx
    import { Request, Response } from "express";
    import { resolve } from 'path'
    import { getCustomRepository } from "typeorm";
    import { SurveysRepository } from "../repositories/SurveysRepository";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";
    import { UsersRepository } from "../repositories/UsersRepository";
    import SendMailService from "../services/SendMailService";

    class SendMailController{
        async execute(request: Request, response: Response){
            const { email, survey_id } = request.body

            const usersRepository = getCustomRepository(UsersRepository)
            const surveyRepository = getCustomRepository(SurveysRepository)
            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const user = await usersRepository.findOne({email})
            if(!user){
                return response.status(400).json({
                    error: "User does not exists"
                })
            }

            const survey = await surveyRepository.findOne({id: survey_id})
            if(!survey){
                return response.status(400).json({
                    error: "Survey does not exists"
                })
            }

            //Salvar as informações na tabela surveyUser
            const surveyUser = surveysUsersRepository.create({
                user_id: user.id,
                survey_id
            })

            
            await surveysUsersRepository.save(surveyUser)

            //Enviar e-mail para o usuário

            const npsPath = resolve(__dirname, "..", "views", "emails", "npsMail.hbs")

            const variables = {
                name: user.name,
                title: survey.title,
                description: survey.description,
                ***user_id: user.id,
                link: process.env.URL_MAIL***
            }

            await SendMailService.execute(email, survey.title, variables, npsPath)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    Testar → yarn dev → send no insomnia

    Agora ao deixar o mouse em cima da nota, ja aparece o link

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%205.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%205.png)

    Agora precisamos pegar a informação e não permitir que tenham várias pesquisas iguais no db

    rodar a query 

    ```sql
    delete from surveys_users
    ```

    No SendMailController:

    ```tsx
    import { Request, Response } from "express";
    import { resolve } from 'path'
    import { getCustomRepository } from "typeorm";
    import { SurveysRepository } from "../repositories/SurveysRepository";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";
    import { UsersRepository } from "../repositories/UsersRepository";
    import SendMailService from "../services/SendMailService";

    class SendMailController{
        async execute(request: Request, response: Response){
            const { email, survey_id } = request.body

            const usersRepository = getCustomRepository(UsersRepository)
            const surveyRepository = getCustomRepository(SurveysRepository)
            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const user = await usersRepository.findOne({email})
            if(!user){
                return response.status(400).json({
                    error: "User does not exists"
                })
            }

            const survey = await surveyRepository.findOne({id: survey_id})
            if(!survey){
                return response.status(400).json({
                    error: "Survey does not exists"
                })
            }

            const npsPath = resolve(__dirname, "..", "views", "emails", "npsMail.hbs")

            const variables = {
                name: user.name,
                title: survey.title,
                description: survey.description,
                user_id: user.id,
                link: process.env.URL_MAIL
            }

            const surveyUserAlreadyExists = await surveysUsersRepository.findOne({
                where: [{user_id: user.id}, {value: null}]
            })
            if(surveyUserAlreadyExists){
                await SendMailService.execute(email, survey.title, variables, npsPath)
                return response.json(surveyUserAlreadyExists)
            }

            //Salvar as informações na tabela surveyUser
            const surveyUser = surveysUsersRepository.create({
                user_id: user.id,
                survey_id
            })

            
            await surveysUsersRepository.save(surveyUser)

            //Enviar e-mail para o usuário

            await SendMailService.execute(email, survey.title, variables, npsPath)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    Agora se você der vários Sends no insomnia, vai retornar sempre o primeiro registro daquele email =)

    No model SurveyUser:

    ```tsx
    import { Column, CreateDateColumn, Entity, JoinColumn, ManyToOne, PrimaryColumn } from "typeorm";
    import { v4 as uuid } from 'uuid'
    import { Survey } from "./Survey";
    import { User } from "./User";

    @Entity("surveys_users")
    class SurveyUser{
        @PrimaryColumn()
        readonly id: string;

        @Column()
        user_id: string;
        **@ManyToOne(()=> User)
        @JoinColumn({name: "user_id"})
        user: User**

        @Column()
        survey_id: string;
        **@ManyToOne(()=> Survey)
        @JoinColumn({name: "survey_id"})
        survey: Survey**

        @Column()
        value: number;

        @CreateDateColumn()
        created_at: Date;

        constructor(){
            if(!this.id){
                this.id = uuid()
            }
        }
    }

    export { SurveyUser }
    ```

    No SendMailController, adicionar o relations:

    ```tsx
    import { Request, Response } from "express";
    import { resolve } from 'path'
    import { getCustomRepository } from "typeorm";
    import { SurveysRepository } from "../repositories/SurveysRepository";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";
    import { UsersRepository } from "../repositories/UsersRepository";
    import SendMailService from "../services/SendMailService";

    class SendMailController{
        async execute(request: Request, response: Response){
            const { email, survey_id } = request.body

            const usersRepository = getCustomRepository(UsersRepository)
            const surveyRepository = getCustomRepository(SurveysRepository)
            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const user = await usersRepository.findOne({email})
            if(!user){
                return response.status(400).json({
                    error: "User does not exists"
                })
            }

            const survey = await surveyRepository.findOne({id: survey_id})
            if(!survey){
                return response.status(400).json({
                    error: "Survey does not exists"
                })
            }

            const npsPath = resolve(__dirname, "..", "views", "emails", "npsMail.hbs")

            const variables = {
                name: user.name,
                title: survey.title,
                description: survey.description,
                user_id: user.id,
                link: process.env.URL_MAIL
            }

            const surveyUserAlreadyExists = await surveysUsersRepository.findOne({
                where: [{user_id: user.id}, {value: null}],
                **relations:["user", "survey"]**
            })
            if(surveyUserAlreadyExists){
                await SendMailService.execute(email, survey.title, variables, npsPath)
                return response.json(surveyUserAlreadyExists)
            }

            //Salvar as informações na tabela surveyUser
            const surveyUser = surveysUsersRepository.create({
                user_id: user.id,
                survey_id
            })

            
            await surveysUsersRepository.save(surveyUser)

            //Enviar e-mail para o usuário

            await SendMailService.execute(email, survey.title, variables, npsPath)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    Assim, ao fazer a requisição, a resposta é o objeto é por completo:

    ![Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%206.png](Dia%204%20-%20Envio%20de%20e-mail%2055ff4755f83b43caa20176289e9fc463/Untitled%206.png)

    Por hoje é só!

    #neverstoplearning