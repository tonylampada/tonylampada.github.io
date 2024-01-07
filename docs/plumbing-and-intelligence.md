
# Plumbing + Intelligence: A Practical Mindset for Software Architecture
## 1. Introduction

As I gained experience in building software, I realized a fundamental distinction between two types of code, which in my mind, I refer to as:

* "plumbing" code, vs
* "intelligence" code.

I want to explain this duality because I believe it assists in cultivating a programming mindset that tends to produce quality software (*).

(*) Quality software: software that doesn't have a lot of bugs and can be modified ~~without wanting to pull your hair out~~ painlessly.

From my experience, in well-crafted applications, we find a subtle harmony between "plumbing" — these channels and conduits that guide the flow of data, — and "intelligence" — the logic and functionalities that breathe life into the system.

When I'm coding, seeing these two roles in the code in front of me makes me feel like I'm "mastering the secrets" of good architecture, and indeed the resulting software has less coupling, more cohesion, does not violate the DRY principle, in short, it has more quality (*).

And most importantly: many of the quality issues I see arise when we mix plumbing and intelligence in the same place. Therefore, recognizing this distinction is crucial to avoid falling into the trap of mixing these two types of things.

So, let's go.

## 2. Plumbing vs Intelligence: What I Mean by This

**Plumbing** refers to those parts of the code that define the structure of the application AND move data from one place to another. For example:

* An entry point that calls a service.
* A routing configuration.
* An adapter that communicates with an external API.
* A catalog of front-end components.
  
It's simple, "boring", repetitive code. But that doesn't make it any less important. 
For a software project, this type of code functions both as:

* A skeleton - defining the shape and structure of the "organism."
* A circulatory system - channels through which data flows, connectors that unite different modules.

**Intelligence** is those parts of the code where business logic, decision-making, and domain-specific rules reside. For example:

* The implementation of business rules in a backend.
* Interactive frontend components.
* "Stores" of reactive state for frontend (typical in React or Vue.js apps).

These pieces of code are the organs of the system: the heart, the lungs, the intestines. 
Each organ has a different and specific function and particular rules about how it operates. 
It's through the action of these organs that we implement the purpose of the system.

Well, that's the general idea, but it might still be too abstract. 
So, I'm going to give an example to materialize how we apply this in a backend.

## 3. Materializing: Three-Layer Backend
I like to think of backend architecture in these three layers:

![CleanShot 2024-01-06 at 23 11 46@2x](https://github.com/tonylampada/tonylampada.github.io/assets/218821/c4a2354d-efab-4f59-91ad-248f91ddde02)

Note that this is "one" way – not necessarily "the" way – to apply this division in a backend. 
But it's a way that I find works well for the majority of cases and I recommend using it unless there's a good reason not to.

**Layer 1: Surface**

The Surface layer is the initial contact point in the backend. They are the "entry-points." 
It deals with external requests, calls a business rule, and sends the response in the appropriate format. 
Pure "Plumbing." 
The focus is on directing requests to the correct services.

Responsibilities:

* Knowing which business method to call.
* Basic validation of input data.
* Implementation of security.
* Observability and handling of unforeseen errors.

The Surface should not contain business logic. 
The methods in this layer are usually short because they only delegate execution to some service. 
The "secondary" responsibilities of security, validation, and observability should preferably be implemented in the form 
of middlewares, decorators, or "helper functions" so that it's easy to activate/deactivate these rules that are "in front" 
or "around" the implementation.

**Layer 2: Services**

Services are the heart of "Intelligence" in the backend. 
This layer contains the essential business logic and is responsible for executing the central operations of the system.

Responsibilities:

* Execution of the main business logic.
* Making decisions based on the data provided.
* Interacting with other parts of the system to perform tasks.

Services should preferably be stateless and focused on executing specific tasks. Services:

* Rely on the Surface layer for the delivery of consistent and reliable data.
* Depend on Adapters to take actions that have effects in the "external world."
* Can also depend and orchestrate the execution of other services

Services should not bother with the details of how to talk to a database or an external API. There are adapters for that. 
Services also should not try to handle unexpected errors. They should handle only the errors that are part of the expected alternative business flows. 
Any unexpected error that occurs will be handled generically by the Surface layer.

**Layer 3: Adapters**

Adapters are the other end of "Plumbing." 
They serve as intermediaries between Services and the external or internal resources of the system, such as databases, 
messaging systems, or third-party APIs.

Responsibilities:

* Connection to databases and execution of queries.
* Communication with external APIs.
* Transformation of data between formats expected by Services and external formats.
* Translation of external API errors into custom exceptions.

Adapters isolate the complexity of external integrations, allowing Services to remain focused on business logic.

## 4. Falar é fácil. Mostra o código aí

Aqui um exemplo de como isso fica numa aplicação em node/express com firebase.

Vamos olhar primeiro pro "jeito ruim". O exemplo aqui é um endpoint de autenticação com login e senha que tenta dar match no banco com um hash da senha, e devolve um token secreto temporário associado ao usuario.


```javascript
//no layers. Plumbing + intelligence blended and scrambled together
const admin = require('firebase-admin');
const db = admin.firestore();
const {hash, randomToken} = require('some-crypto-thing')
const cache = {};


app.post('/api/login', async (req, res) => {
   try {
       const { login, password } = req.body;
       if (!login || !password) {
           console.warn(`/api/login status=400 bad input`);
           return res.status(400).
               json({ error: 'Login and password are required' });
       }
       const collection = db.collection('users');
       const snapshot = await collection.
           where('login', '==', login).
           where('password', '==', hash(password)).get();
       if (snapshot.empty) {
           console.warn(`/api/login status=401 auth failed`);
           res.status(401).json({ error: 'Authentication failed' });
       } else {
           const user = snapshot.docs[0].data();
           const token = randomToken()
           cache[token] = user; //in-memory for simplicity. could be Redis or something
           console.log(`/api/login status=200 OK`);
           res.status(200).json({
               message: 'Authentication successful',
               token: token
           });
       }
   } catch (err) {
       console.error(`/api/login - Error: ${err.message}`);
       res.status(500).send('Unknown server error');
   }
});
```

Isso aí é o "mau exemplo" do encanamento (tratamento da requisição HTTP, acesso ao BD) misturado com a inteligência (a regra de negócio que diz que "você só passa daqui com a senha correta")

Olha só o monte de problema aqui:

* Você vai mesmo querer ficar logando console.log("status=blah") em todo request handler?
* E esse try-catch gigante, também?
* Não tem a menor chance de você conseguir reusar essa lógica de negócio de autenticação sem duplicar código.
* O conhecimento de como acessar o banco de dados também vai ficar espalhado em tudo que é request handler.

Enfim. Eu não gosto disso aí não. É o tipo de código que é até rápido de criar, mas que dificulta o reuso, dificulta o DRY e é lento de mudar à medida que a aplicação cresce.

Eu mudaria isso da seguinte forma:

**Camada 1 - Request handler (encanamento)**

```javascript
// surface: handling http requests for login
// "app" is an express application
// "authenticationService" is a service that knows how to authenticate users

app.use(loggingMiddleware);
app.use(handleUnknownErrorsMiddleware);
app.post('/api/login', reqLogin);

function loggingMiddleware(req, res, next) {
   res.on('finish', () => {
       console.log(`INFO: Method ${req.method} called at URL ${req.originalUrl} - Status: ${res.statusCode}`);
   });
   next();
}

function handleUnknownErrorsMiddleware(err, req, res, next){
   console.error(`ERROR: An error occurred on method ${req.method} at URL ${req.originalUrl} - Error: ${err.message}`);
   res.status(500).send('Server error occurred');
}

async function reqLogin(req, res){
   const { login, password } = req.body;
   if (!login || !password) {
       return res.status(400).json({ error: 'Login and password are required' });
   }
   const result = await authenticationService.authenticate(login, password);
   if (result.success) {
       res.status(200).json({ message: 'Authentication successful', token: result.token });
   } else {
       res.status(401).json({ error: 'Authentication failed' });
   }
}
```
Lido com request e response. A lógica pertence ao serviço. O serviço nem sabe o que é request, response, ou status code. Observabilidade implementada em middlewares.

**Camada 2 - authenticationService (inteligência)**

```javascript
// authenticationService

const {userDao, filters} = require('./adapters/databaseAdapter');
const {userByToken} = require('./adapters/cacheAdapter')
const {isEqual} = filters
const {hash, randomToken} = require('some-crypto-thing')

async function authenticate(login, password) {
   const user = await userDao.findWhere([isEqual("login", login), isEqual("password", hash(password))]);
   if (user) {
       const token = randomToken()
       await userByToken.set(token, user)
       return { success: true, token: token }; // Exemplo de token
   } else {
       return { success: false };
   }
}

async function getUserByToken(token) {
   // exercise. think about it.
}

module.exports = {
   authenticate,
   getUserByToken
};
```
Conheço a regra pra autenticar o usuario. Orquestro diferentes adapters pra implementar tarefas mais baixo nível.

**Camada 3 - databaseAdapter (encanamento)**

```javascript
// databaseAdapter.

const admin = require('firebase-admin');
const db = admin.firestore();

class BaseDao {
   constructor(collectionName) {
       this.collection = db.collection(collectionName);
   }

   async findWhere(conditions) {
       let query = this.collection;
       for (const condition of conditions) {
           query = query.where(condition.field, condition.operator, condition.value);
       }
       const snapshot = await query.get();
       if (snapshot.empty) {
           return null;
       }
       let results = [];
       snapshot.forEach(doc => results.push({ id: doc.id, ...doc.data() }));
       return results.length === 1 ? results[0] : results;
   }
}

const filters = {
   isEqual(field, value){
       return {field, operator: '==', value}
   },
   // isGreater, isLess, ...
}

module.exports = {
   userDao: new BaseDao("users"),
   filters: filters
};
```
Sei falar com o firestore (pra que serviços não precisem saber). "admin.firestore()" não é chamado em nenhum outro lugar. Serviços não se incomodam em saber COMO falar com o banco de dados (ou storage, ou cache, ou APIs externas). Então quando essas coisas mudam, não é doloroso e ninguém precisa arrancar o resto dos cabelos.

É isso. Espero que tenha achado essas idéias úteis. 
Até a próxima 🙂
