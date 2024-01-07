# Encanamento + Inteligência: um mindset prático para arquitetura de software

## 1. Introdução

À medida que eu fui ganhando experiência construindo software, percebi uma distinção fundamental entre dois tipos código, que, na minha cabeça, eu chamo de: 

* código de "encanamento", vs
* código de "inteligência". 

Vou explicar essa dualidade porque eu acredito que ela ajuda ter um mindset de programação que tende a gerar software de qualidade (*).

(*) Software de qualidade: software que não tem muito bug e você consegue mudar de maneira indolor sem ter vontade de arrancar os cabelos.

Da minha experiência, em aplicações bem feitas, encontramos uma harmonia sutil entre o "encanamento" — esses canais e condutos que guiam o fluxo de dados, — e a "inteligência"  — a lógica e as funcionalidades que dão vida ao sistema. 

Quando eu programo enxergando esses dois papéis no código que tá na minha frente, eu sinto que estou "dominando os segredos" da boa arquitetura, e de fato o software resultante tem menos acoplamento, mais coesão, não viola o DRY, enfim, tem mais qualidade (*).

E o mais importante: muitos dos problemas de qualidade que eu vejo aparecem quando a gente mistura encanamento + inteligência no mesmo lugar. Então enxergar essa distinção é importante justamente pra gente evitar cair nas armadilhas de misturar esses dois tipos de coisa.

Então vamo lá.

## 2. Encanamento vs Inteligência: O que eu quero dizer com isso

Encanamento são aquelas partes do código que basicamente levam dados de um lugar pra outro. Por exemplo:
* Um entrypoint que chama um serviço
* Uma configuração de roteamento
* Um adapter que fala com uma API externa
* Um catálogo de componentes de front-end

É código simples, "boring", repetitivo. Mas nem por isso ele é menos importante. Pro projeto de software, esse tipo de código funciona como: 
* um esqueleto - que define a forma e a estrutura do "organismo"
* um sistema circulatório - canais por onde os dados fluem, conectores que unem diferentes módulos

**Inteligência** são aquelas partes do código onde fica a lógica de negócio, as tomadas de decisão, as regras específicas ao domínio de aplicação. É o núcleo que confere propósito ao sistema. Por exemplo:
* A implementação das regras de negócio em um backend
* Componentes interativos de frontend
* "Stores" de estado reativo pra frontend (típico de apps React ou Vue.js)

Esses pedaços de código são os órgãos do sistema: o coração, o pulmão, o intestino. Cada órgão tem uma função diferente e específica e regras bem particulares de como ele funciona. É através da ação desses órgãos que a gente implementa o propósito do sistema.

Bom, isso é a idéia geral. Mas talvez ainda esteja abstrato demais. Então eu vou dar um exemplo tangibilizando como a gente aplica isso num backend.

## 3. Tangibilizando: Backend em três camadas
Eu gosto de pensar em backend nestas 3 camadas:

![CleanShot 2024-01-06 at 23 11 46@2x](https://github.com/tonylampada/tonylampada.github.io/assets/218821/c4a2354d-efab-4f59-91ad-248f91ddde02)

Repare que isso é "um" jeito – não necessariamente "o" jeito – de aplicar essa divisão em um backend. Mas é um jeito que eu acho que funciona bem pra grande maioria dos casos e eu recomendo usar isso na falta de um bom motivo pra não.

**Camada 1: Superfície**

A camada da Superfície é o ponto de contato inicial no backend. São os "entry-points". Ela lida com as requisições externas chama uma regra de negócio e manda a resposta no formato adequado. Puro "Encanamento". O foco é  direcionar as solicitações para os serviços corretos.

Responsabilidades:
* Saber qual método de negócio chamar.
* Validação básica de dados de entrada.
* Implementação de segurança.
* Observabilidade e tratamento de erros imprevistos.

A Superfície não deve conter lógica de negócios. Os métodos dessa camada normalmente são curtos porque apenas delegam a execução para algum serviço. As responsabilidades "secundárias" de segurança, validação e observabilidade devem ser implementadas preferencialmente na forma de middlewares, decorators, ou "helper functions" de maneira que seja fácil ativar/desativar essas regrinhas que ficam "na frente" ou "ao redor" da implementação.

**Camada 2: Serviços**

Os Serviços são o coração da "Inteligência" no backend. Esta camada contém a lógica de negócios essencial e é responsável por executar as operações centrais do sistema.

Responsabilidades:
* Execução da lógica de negócios principal.
* Tomada de decisões com base nos dados fornecidos.
* Interagir com outras partes do sistema para realizar tarefas.

Os serviços devem ser preferencialmente stateless e focados na execução de tarefas específicas. Serviços: 
* Confiam na camada de Superfície para a entrega de dados consistentes e confiáveis. 
* Dependem de Adapters para tomar ações que têm efeitos no "mundo externo".

Serviços não devem se incomodar com os detalhes de como falar com um banco de dados ou uma API externa. Eles usam adapters pra isso. Serviços também não devem tentar tratar erros imprevistos. Eles só se incomodam em tratar erros que fazem parte do funcionamento previsto da aplicação. Qualquer erro imprevisto que acontecer vai ser tratado de maneira genérica pela camada de Superfície.

**Camada 3: Adapters**

Os Adapters são a outra ponta do "Encanamento". Eles servem como intermediários entre os Serviços e os recursos externos ou internos do sistema, como bancos de dados, sistemas de mensageria ou APIs de terceiros.

Responsabilidades:
* Conexão com bancos de dados e execução de consultas.
* Comunicação com APIs externas.
* Transformação de dados entre formatos esperados pelos Serviços e formatos externos.
* Tradução de erros da API externa pra exceções customizadas

Os Adapters isolam a complexidade das integrações externas, permitindo que os Serviços se mantenham focados na lógica de negócios.

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
