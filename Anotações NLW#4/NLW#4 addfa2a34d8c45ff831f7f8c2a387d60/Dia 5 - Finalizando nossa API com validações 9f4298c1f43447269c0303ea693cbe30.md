# Dia 5 - Finalizando nossa API com validações

- [x]  O que aprendemos ontem

    Envio de emails

- [x]  O que vamos aprender hoje

    Refatorar controller de envio de email

    Criar controller de resposta do usuário

    Alterar a nota da resposta no bd

    Criar o calculo do NPS

    Ver validações para os controllers

- [x]  Oportunidade
    - [x]  Nosso método (Grupo, foco e prática)
    - [x]  Transformar esses elementos em uma metodologia é nosso papel
    - [x]  Um currículo alinhado

- [x]  Refatorar o SendMailController

    Precisamos transformar o OR para um AND no SendMailController, assim conseguiremos salvar a nota do usuário

    ```tsx
    //COMO ESTAVA:
    const surveyUserAlreadyExists = await surveysUsersRepository.findOne({
                where: [{user_id: user.id}, {value: null}],
                relations:["user", "survey"]
            })

    //COMO DEVE FICAR:

    const surveyUserAlreadyExists = await surveysUsersRepository.findOne({
                where: {user_id: user.id , value: null},
                relations:["user", "survey"]
            })
    ```

    Outro ponto, devemos alterar as variables

    Um Usuário pode ter mais de uma pesquisa, precisamos saber de qual pesquisa pertence a avaliação

    Precisamos enviar o ID da SurveyUser

    Alteramos a localização das variables no código e colocamos nos devidos lugares para que seja validade quando exista o surveyUser e se não existir, salvar quando criar

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

            const surveyUserAlreadyExists = await surveysUsersRepository.findOne({
                where: {user_id: user.id , value: null},
                relations:["user", "survey"]
            })

            const variables = {
                name: user.name,
                title: survey.title,
                description: survey.description,
                id: "",
                link: process.env.URL_MAIL
            }

            if(surveyUserAlreadyExists){
                variables.id = surveyUserAlreadyExists.id;
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
            variables.id = surveyUser.id
            await SendMailService.execute(email, survey.title, variables, npsPath)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    Agora no npsMail.hbs:

    Alterar o user_id para id

    Ctrl + Shift + L OU Cmd + Shift + L para selecionar todos os "user_id"

    ```html
    <div class="answers">
            <a class="value" href="{{link}}/1?u={{id}}">1</a>
            <a class="value" href="{{link}}/2?u={{id}}">2</a>
            <a class="value" href="{{link}}/3?u={{id}}">3</a>
            <a class="value" href="{{link}}/4?u={{id}}">4</a>
            <a class="value" href="{{link}}/5?u={{id}}">5</a>
            <a class="value" href="{{link}}/6?u={{id}}">6</a>
            <a class="value" href="{{link}}/7?u={{id}}">7</a>
            <a class="value" href="{{link}}/8?u={{id}}">8</a>
            <a class="value" href="{{link}}/9?u={{id}}">9</a>
            <a class="value" href="{{link}}/10?u={{id}}">10</a>
        </div>
    ```

    Testar no terminal

    ```powershell
    yarn dev
    ```

- [x]  Criar controller de resposta de usuário

    Criar o arquivo do AnswerController.ts

    Relembrando

    Exemplo: [http://localhost:3333/answers/1?u=1007bc7e-694d-4a0d-b147-f93598e506e5](http://localhost:3333/answers/1?u=1007bc7e-694d-4a0d-b147-f93598e506e5)

    ROUTE PARAMS → Parâmetros que compõem a rota

    routes.get("/answers/:value")

    QUERY PARAMS → Busca, Paginação, não obrigatórios:

    Chave = valor

    Nesse caso, o usuário por exemplo seria um queryparam

    No AnswerController.ts

    ```tsx
    import { Request, Response } from "express";
    import { getCustomRepository } from "typeorm";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";

    class AnswerController{
        async execute (request: Request, response: Response) {
            const{ value } = request.params
            const { u } = request.query

            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const surveyUser = await surveysUsersRepository.findOne({
                id: String(u)
            })

            if(!surveyUser){
                return response.status(400).json({
                    error: "Survey User does not exists!"
                })
            }

            surveyUser.value = Number(value)

            await surveysUsersRepository.save(surveyUser)

            return response.json(surveyUser)

        }
    }

    export { AnswerController }
    ```

    - [x]  Validar se o usuário existe

        Isso é feito aqui:

        ```tsx
                if(!surveyUser){
                    return response.status(400).json({
                        error: "Survey User does not exists!"
                    })
                }
        ```

    - [x]  Alterar a nota da resposta

        Isso é feito aqui:

        ```tsx
                await surveysUsersRepository.save(surveyUser)

                return response.json(surveyUser
        ```

    No arquivo routes.ts

    ```tsx
    import { Router } from 'express';
    import { UserController } from "./controllers/UserController"
    import { SurveyController } from "./controllers/SurveyController"
    import { SendMailController } from './controllers/SendMailController';
    import { AnswerController } from './controllers/AnswerController';

    const router = Router();

    const userController = new UserController();
    const surveyController = new SurveyController();

    const sendMailController = new SendMailController()
    const answerController = new AnswerController()

    router.post("/users", userController.create)
    router.post("/surveys", surveyController.create)
    router.get("/surveys", surveyController.show)

    router.post("/sendMail", sendMailController.execute)
    router.get("/answers/:value", answerController.execute)

    export { router }
    ```

- [x]  Criar controller com cálculo do NPS

    As notas são: 

    1 2 3 4 5 6 7 8 9 10

    - Detratores => 0 - 6
    - Passivos => 7 - 8
    - Promotores => 9 - 10

    O Calculo é o seguinte:

    (Número de Promotores - Número de Detratores)/ (Numero de respondentes) * 100

    Criar o NpsController.ts

    ```tsx
    import { Request, Response } from "express";
    import { getCustomRepository, Not, IsNull } from "typeorm";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";

    class NpsController{

        async execute( request: Request, response: Response){
            const { survey_id } = request.params
            
            const surveyUserRepository = getCustomRepository(SurveysUsersRepository)
            
            const surveyUsers = await surveyUserRepository.find({
                survey_id,
                value: Not(IsNull())
            })

            const detractor = surveyUsers.filter(survey => 
                (survey.value >= 0 && survey.value <=6)
            ).length

            const promoters = surveyUsers.filter(survey =>
                (survey.value >= 9 && survey.value <= 10)    
            ).length
            const passives = surveyUsers.filter(survey=>
                (survey.value >= 7 && survey.value <= 8)
            ).length

            const totalAnswers = surveyUsers.length

            const calculate = Number(
                ((( promoters - detractor) / totalAnswers ) * 100).toFixed(2)
            )
            
            return response.json({
                detractor,
                promoters,
                passives,
                totalAnswers,
                nps : calculate
            })

        }
    }

    export { NpsController }
    ```

    Criar a rota:

    ```tsx
    import { Router } from 'express';
    import { UserController } from "./controllers/UserController"
    import { SurveyController } from "./controllers/SurveyController"
    import { SendMailController } from './controllers/SendMailController';
    import { AnswerController } from './controllers/AnswerController';
    import { NpsController } from './controllers/NpsController';

    const router = Router();

    const userController = new UserController();
    const surveyController = new SurveyController();

    const sendMailController = new SendMailController()
    const answerController = new AnswerController()
    const npsController = new NpsController()

    router.post("/users", userController.create)
    router.post("/surveys", surveyController.create)
    router.get("/surveys", surveyController.show)

    router.post("/sendMail", sendMailController.execute)
    router.get("/answers/:value", answerController.execute)

    router.get("/nps/:survey_id", npsController.execute)

    export { router }
    ```

- [x]  Criar validações

    Utilizaremos a biblioteca [yup validation](https://github.com/jquense/yup)

    ```powershell
    yarn add yup
    ```

    Importar na UserController

    ```powershell
    import * as yup from 'yup'
    ```

    No UserController

    ```tsx
    import { Request, Response } from 'express';
    import { getCustomRepository } from 'typeorm';
    import { UsersRepository } from '../repositories/UsersRepository';
    import * as yup from 'yup'

    class UserController{

        async create(request: Request ,response: Response){
            const { name, email } = request.body;

            const schema = yup.object().shape({
                name: yup.string().required("Name is required"),
                email: yup.string().email().required("Invalid email"),

            })

            try{
                await schema.validate(request.body, { abortEarly: false})
            }catch(err){
                return response.status(400).json({error: err})
            }

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

            return response.status(201).json(user)
        }
    }

    export { UserController };
    ```

- [x]  BONUS: Refatorar nosso código para não precisar remover a nossa base de dados

    Remover o posttest do package.json

    E no Survey.test.ts após o BeforeAll, colocar o AfterAll

    ```tsx
    afterAll(async ()=>{
            const connection = getConnection()
            await connection.dropDatabase()
            await connection.close()
        })
    ```

    No package.json na parte de test, colocar -i

    ```json
    "scripts": {
        "dev": "ts-node-dev --transpile-only --ignore-watch node_modules src/server.ts",
        "typeorm": "ts-node-dev node_modules/typeorm/cli.js",
        "test": "NODE_ENV=test jest -i"
      },
    ```

    o -i faz rodar um depois o outro

- [x]  BONUS 2 : Criar uma tratativa melhor de erros

    Criar a pasta errors dentro de src

    Criar arquivo AppError.ts

    ```tsx
    export class AppError{
        public readonly message: string
        public readonly statusCode: number

        constructor( message: string, statusCode = 400){
            this.message = message
            this.statusCode = statusCode
        }
    }
    ```

    Agora todo erro que a gente retornava antes no controller, a gente vai ir jogando pra cima

    Por exemplo na AnswerController, quem chama ela é o routes.ts e quem chama a routes.ts é o app.ts

    Agora ao invés disso:

    ```tsx
    if(!surveyUser){
                return response.status(400).json({
                    error: "Survey User does not exists!"
                })
            }
    ```

    teremos isso:

    ```tsx
    if(!surveyUser){
         throw new AppError("Survey User does not exists!")
    }
    ```

    Substituir os erros dos controllers:

    AnswerController:

    ```tsx
    import { Request, Response } from "express";
    import { getCustomRepository } from "typeorm";
    import { AppError } from "../errors/AppError";
    import { SurveysUsersRepository } from "../repositories/SurveysUsersRepository";

    class AnswerController{
        async execute (request: Request, response: Response) {
            const{ value } = request.params
            const { u } = request.query

            const surveysUsersRepository = getCustomRepository(SurveysUsersRepository)

            const surveyUser = await surveysUsersRepository.findOne({
                id: String(u)
            })

            if(!surveyUser){
                throw new AppError("Survey User does not exists!")
            }

            surveyUser.value = Number(value)

            await surveysUsersRepository.save(surveyUser)

            return response.json(surveyUser)

        }
    }

    export { AnswerController }
    ```

    SendMailController:

    ```tsx
    import { Request, Response } from "express";
    import { resolve } from 'path'
    import { getCustomRepository } from "typeorm";
    import { AppError } from "../errors/AppError";
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
                throw new AppError("User does not exists")
            }

            const survey = await surveyRepository.findOne({
                id: survey_id
            })
            
            if(!survey){
                throw new AppError("Survey does not exists")
            }

            const npsPath = resolve(__dirname, "..", "views", "emails", "npsMail.hbs")

            const surveyUserAlreadyExists = await surveysUsersRepository.findOne({
                where: {user_id: user.id , value: null},
                relations:["user", "survey"]
            })

            const variables = {
                name: user.name,
                title: survey.title,
                description: survey.description,
                id: "",
                link: process.env.URL_MAIL
            }

            if(surveyUserAlreadyExists){
                variables.id = surveyUserAlreadyExists.id;
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
            variables.id = surveyUser.id
            await SendMailService.execute(email, survey.title, variables, npsPath)

            return response.json(surveyUser)

        }

    }

    export { SendMailController }
    ```

    UserController

    ```tsx
    import { Request, Response } from 'express';
    import { getCustomRepository } from 'typeorm';
    import { UsersRepository } from '../repositories/UsersRepository';
    import * as yup from 'yup'
    import { AppError } from '../errors/AppError';

    class UserController{

        async create(request: Request ,response: Response){
            const { name, email } = request.body;

            const schema = yup.object().shape({
                name: yup.string().required("Name is required"),
                email: yup.string().email().required("Invalid email"),

            })

            try{
                await schema.validate(request.body, { abortEarly: false})
            }catch(err){
                throw new AppError(err)   
            }

            const usersRepository = getCustomRepository(UsersRepository)

            // SELECT * FROM USERS WHEER EMAIL = "EMAIL"
            const userAlreadyExists = await usersRepository.findOne({
                email
            });
            if(userAlreadyExists){
                throw new AppError("User already exists!")           
            }

            const user = usersRepository.create({
                name, email
            })

            await usersRepository.save(user)

            return response.status(201).json(user)
        }
    }

    export { UserController };
    ```

    No app.ts devemos alterar para ele receber os erros:

    ```tsx
    import 'reflect-metadata';
    import express, { NextFunction, Request, Response } from 'express';
    import createConnection from "./database";
    import { router } from './routes';
    import { AppError } from './errors/AppError';

    createConnection()
    const  app = express();

    app.use(express.json())
    app.use(router)

    app.use((err: Error, request: Request, response:Response, _next:NextFunction)=>{
        if(err instanceof AppError){
            return response.status(err.statusCode).json({
                message: err.message
            })
        }
        return response.status(500).json({
            status: "Error",
            message: `Internal server error ${err.message}`
        })
    })

    export { app };
    ```

    Porém para o express conseguir lidar com esses erros, precisamos utilizar uma bibioteca

    ```powershell
    yarn add express-async-errors
    ```

    E importar no app.ts após a importação do express

    ```tsx
    import 'reflect-metadata';
    import express, { NextFunction, Request, Response } from 'express';
    import "express-async-errors"
    import createConnection from "./database";
    import { router } from './routes';
    import { AppError } from './errors/AppError';

    createConnection()
    const  app = express();

    app.use(express.json())
    app.use(router)

    app.use((err: Error, request: Request, response:Response, _next:NextFunction)=>{
        if(err instanceof AppError){
            return response.status(err.statusCode).json({
                message: err.message
            })
        }
        return response.status(500).json({
            status: "Error",
            message: `Internal server error ${err.message}`
        })
    })

    export { app };
    ```

    #missioncomplete