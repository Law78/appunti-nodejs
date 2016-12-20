# appunti-nodejs

*Se le soluzioni diventano complesse, probabilmente il problema è mal posto o non esiste (Lorenzo Franceschini)*

Quando si parla di NodeJS, dobbiamo parlare subito di architettura __REST__ e del concetto che ciò di cui ci serviamo è visto come una risorsa, raggiungile attraverso una rotta detta __ROUTES__ tramite un comando chiamato più propriamento __VERBS__ del protocollo di comunicazione __HTTP__ che ci permette quindi di identificare in maniera univoca quello che si dice __URI__. La risorsa è rappresentata da un formato, definito dal Content-type e richiesta tramite l'Accept nell'header, che può essere __JSON__, __XML__, text o una forma binaria (es. image/jpg).
La comunicazione con la risorsa è considerata __Atomica__, per cui un comando porta in se tutta l'informazione necessaria alla modifica della risorsa stessa. Questo è ciò che porta una architettura REST ad essere __RESTful__ e cioè senza stati. Il fatto che la comunicazione è senza stato ci permette di scalare l'applicazione adottando dei carichi di bilanciamento e avere più nodi in risposta alle richieste, ad esempio utilizzando NGINX.
Una richiesta inviata al server genera una risposta, che, oltre a contenere la risorsa nel formato specificato, prevede l'invio di uno __STATUS__. I valori di status più comuni sono: 200 (OK), 201 (Risorsa Creata), 204 (No Content - usata per la DELETE) 400 (Richiesta non valida), 404 (Risorsa Non Trovata), 500 (Errore interno del Server).

### Ricapitolando

__REST__: è una architettura web che definisce le regole di comunicazione tra client e server. Adottato come principale architettura da molto servizi web, fornendo l'accesso a degli endpoints ad una __risorsa__ tramite i principali __VERBS__ del protocollo HTTP. Solitamente in coppia con __JSON__ e sistemi di autenticazione basati su __TOKEN__, che risiedono nell'HEADER di una Request.
__VERBS__: sono i comandi che inviamo tramite il protocollo HTTP, i più importanti sono GET, POST, PUT e DELETE che, per convenzione in molti framework e testi, vengono associati ai comandi CRUD del mondo SQL. I comandi POST e PUT sono ambedue comandi di creazione di una risorsa, in genere se l'identificativo viene creato lato server (e quindi probabilmente una risorsa ex-novo) la POST è più appropriata della PUT. Se l'identificativo è fornito lato client o comunque è una risorsa conosciuta dal client, la PUT è più appropriata. Filosofia-time. I metodi __safe__ sono quelli che non modificano le risorse e cioè la sola GET (e OPTIONS e HEAD non menzionati) e pertanto possono essere anche cachati senza danni. I metodi __idempotenti__ sono quei metodi che posso chiamare n volte senza che il risultato cambi, come ad esempio la funzione che stampa "ciao" o i verbs PUT e DELETE. Il POST? Non è ne safe ne idempotent. Ciò è da tenere a mente per costruire API fault-tolerant.


# NodeJS

_NodeJS ci permette di usare Javascript anche per il codice backend, NodeJS è single-thread con un cuore asincrono, NodeJS è EventDriven ed esegue il codice concorrentemente grazie all'event-loop. NodeJS è veloce, grazie all'engine Google V8 ed al sistema I/O non bloccante. NodeJS è immediato (ma non semplice per la sua natura asincrona). L'ordine temporale va oltre al pensiero logico della programmazione sincrona e del nostro modo di pensare. E' questo lo scoglio da superare._

L'installazione di NodeJS ci fornisce oltre al comando node (un ambiente REPL), anche l'utilissimo Node Package Manager (npm) per l'installazione e creazione di moduli.
Per la creazione di un progetto utilizziamo il comando: npm init. Questo comando ci genera il file __package.json__ con i dettagli del progetto e i moduli che andremo ad installare sempre con il comando npm.

### Module Pattern

La logica delle funzionalità viene racchiusa in __moduli__. I moduli vengono inizializzati con la __require__ una funzione che effettua il cache dell'__oggetto__ ritornato (da module.exports). Pertanto per _non avere cache_ devo ritornare una __function__ con la exports o con module.exports. Quando uso la __exports__ devo sempre aggiungere un oggetto: exports.getUser = ... e non assegnare direttamente exports!
Il modulo richiamato da NodeJS viene processato dalla librearia Module di NodeJS che crea un oggetto module che carica il nostro file e crea un funzione che wrappa il nostro file. In realtà tutto ciò che scrivo in NodeJS è incluso in questa funzione che crea NodeJS:

```js
(function(exports, require, module, __filename, __dirname){
  var hello = function(){
    console.log('Hello World!');
  };
  
  module.exports = hello;
});
```

Questo è importante da capire perchè in questo modo il mio modulo è protetto (function scope) da eventuali altri moduli che hanno stessi nome, inoltre è importante perchè potrei incorrere nell'errore che il mio file app.js iniziale possa essere legato al this del global scope, mentre sarà il this della funzione wrap. Il module.exports è ciò che ritorna la require.

Ovviamente le modalità di ciò che esporto sono varie:

```js
var english = require('./english');
...
module.exports = {
   english: english,
   italian: italian,
   french: french
}
```

Dove suppongo di avere delle require, ognuna per una lingua, che a loro volta importano delle funzionalità che espongo in un file js, che a sua volta verrà importato da un altro file e che potrà accedere alle varie funzionalità esposte dagli oggetti che rappresentano le varie lingue.
Se importo un file JSON con la require, questo viene implicitamente converitito in un oggetto JS.

Esempio di come evitare il cache e poter fare due o più require di questo stesso modulo, senza avere side effects:

```js
module.exports = function() {
   var rate = 0;
   return {
      rateObj: function(points) {
         rate = points;
      },
      getPoints: function() {
         rate ratePoints;
      }
   }
}
```

### Richieste e Risposte: il nostre Server

NodeJS ci permette di gestire le richieste e le risposte del nostro server. Sicuramente gestiremo molti file statici (file HTML/immagini/fogli di stile/ecc...), successivamente dobbiamo essere in grado di gestire gli endpoints richiesti, ovvero gli indirizzi URL in entrata al server, che chiameremo __Routes__. Le routes verranno dirottate in specifici funzioni, o meglio moduli, che ne gestiranno la logica di backend.
Quindi gli step sono: __parse url__, __file statici__, __logica delle routes__. I file statici accedono al file system, mentre la logica delle routes solitamente ad un __database__.
Il core di NodeJS ci fornisce gli strumenti base per definire un nostro server che gestisca le richieste (request o req) e le risposte (response o res) grazie al modulo __http__, il parse delle url e cioè l'analisi dell'indirizzo digitato e quindi la consapevolezza di ciò che ha chiesto l'utente e di eventuali query string passate, tramite il modulo __url__, e la gestione del file system con __path__ e __fs__. Qui un semplicissimo esempio che mette in evidenza che il browser effettua, più di una request, in questo caso oltre la request alla root (/) di localhost:3000, chiederà anche la favicon.ico:

```js
var http = require('http');

var server = http.createServer(handleRequest).listen(3000);

function handleRequest(request, response){
  console.log('Arrivata Richiesta', request.method);
  console.log(request.url);
  response.end();
}

console.log('Server in ascolto');
```

Il server creato di fatto è un oggeto di EventEmitter, che gestisce l'evento tramite una callback che ha due oggetti request e response. La request, come abbiamo visto, porta con se vari proprietà tra cui il __method__ e l'__url__. La request, anch'essa EventEmitter, implemeta uno stream di ascolto per l'invio dei dati contenuti nel body. Questo stream deve essere canalizzato (pipe) in modo da ricomporre i cari chunk con gli eventi __data__ e __end__, e gestire eventuali errori con l'evento __error__:

```js
var server = http.createServer().listen(3000);

server.on('request', function(request, response) {
  var body = [];
  request.on('error', function(err) {
    console.error(err.stack);
  }).on('data', function(chunk){
    body.push(chunk);
  }).on('end', function(){
    body = Buffer.concat(body).toString();
    console.log('body:', body);
    response.end();
  });
});
```

_Nota che la response.end() è stata inserita nella end di ricezione del body._
Prova con curl ad inviare una richiesta:

```bash
curl -H "Content-Type: application/json" -X POST -d '{"username":"xyz","password":"xyz"}' http://localhost:3000
```

L'Header di risposta si può impostare implicitamente con il response.setHeader(header) e response.statusCode(status) o esplicitamente con il response.writeHead(status, header). La differenza è con quest'ultimo stabiliamo il punto preciso dell'invio dell'header, con il primo lasciamo questa decisione a NodeJS. Inoltre anche la risposta può generare un errore e quindi anche qui dovrò prevedere l'on-error come visto per la request:

```js
var server = http.createServer().listen(3000);

server.on('request', function(request, response) {
  var body = [];
  request.on('error', function(err) {
    console.error(err.stack);
  }).on('data', function(chunk){
    body.push(chunk);
  }).on('end', function(){
    body = Buffer.concat(body).toString();
    console.log('body:', body);
    response.on('error', function(err){
      console.error(err.stack);
    });
    response.writeHead(200, {'Content-Type':'application/json', 'Powered-By':'Lorenzo'});
    response.write(JSON.stringify(body));
    response.end();
  });
});
```

Possiamo creare un server passando una callback che canalizzi la logica in base al metodo della richiesta:

```js
http.createServer(handle_request).listen(port, '127.0.0.1'); 
```

dove handle_request potrebbe essere una funzione con all'interno una switch:

```js
function handle_request(request, response) { 
 
  switch (request.method) { 
    case 'GET': 
      handle_GET_request(response); 
      break; 
    case 'POST': 
      handle_POST_request(response); 
      break; 
    ...
    default:
      handle_BAD_request(response);
  }
}

function handle_GET_request(response) {
   ...
}
...POST..PUT...
function handle_DELETE_request(response) { 
  response.writeHead(200, { 
    'Content-Type' : 'text/plain' 
  }); 
  response.end('Delete action was requested'); 
} 
 
function handle_bad_request(response) { 
  response.writeHead(400, { 
    'Content-Type' : 'text/plain' 
  }); 
  response.end('Bad request'); 
} 
```

Questo è semplicemente un banale esempio, il codice andrebbe __modularizzato__ in base agli endpoints della nostra API. Inoltre, passo fondamentale, è bene avere uno schema degli endpoints della nostra applicazione:

| Method | URI                                | Descrizione                                |
|--------|------------------------------------|--------------------------------------------|
| GET    | /albums                            | Preleva tutti gli albums                   |
| GET    | /albums/:id                        | Preleva l'album con l'identificativo :id   |
| GET    | /albums&page=2                     | Preleva tutti gli albums tramite paging    |
| GET    | /albums&international=it&year=2010 | Preleva tutti gli albums italiani del 2010 |
| POST   | /albums                            | Crea o aggiorna un albums                  |
| PUT    | /albums/:id                        | Aggiorna l'album identificato da :id       |
| DELETE | /albums/:id                        | Elimina l'album identificato da :id        |

Tabella MD generata con: [markdown tables](http://www.tablesgenerator.com/markdown_tables)

# Express

Ovviamente c'è un modo più semplice per gestire il body, tramite il modulo __body-parser__ e __multiparty__ e le request e le reponse con uno dei più importanti moduli di NodeJS: __Express__, un framework per lo sviluppo di server API.

Creiamo il progetto con npm init e installiamo con npm i moduli: express, body-parser, nodemon, eslint e morgan, ma in futuro saranno utili anche moment, compact, chalk, passport, node-uuid ecc...
I moduli possono essere installati localmente all'applicazione in modalità dipendenza o devDependecy o in modalità globale. Ad esempio nodemon è installabile globalmente. Se un modulo è esclusivamente in uso per lo sviluppo (ad es. morgan) possiamo installarlo con npm con --save-dev anzichè --save. Altri tool utili sarà tutta la suite babel per la transpilazione del codice in ES6: babel-core babel-cli babel-preset-es2015 babel-preset-stage-0

```bash
npm install -g nodemon
npm install --save express body-parser ejs
npm install --save-dev morgan eslint
```

Express utilizza di default JADE come template per la costruzione delle view (pagine HTML). La sintassi utilizzata da JADE è familiare a chi proviene da Python e poco amata da chi proviene da HTML. Qui proviamo ad utilizzare EJS al posto di JADE, pertanto va installato il modulo di ejs con npm.
La struttura iniziale delle cartelle di progetto sarà:

```
|_/src
   |_/config
      |_routes.js
   |_index.js
|_/static
   |_/views
      |_/pages
         |_index.ejs
         |_about.ejs
      |_/partials
         |_head.ejs
         |_header.ejs
         |_footer.ejs
|_/node_modules
|_package.json
```

Configuriamo alcune sezione di package.json, in particolare la "scripts" che ci permette di definire delle righe di comando di avvio del progetto, di test del progetto, di build speciali o semplicemente il richiamo ad un altro comando, come può essere eslint, che ci serve per la verifica della sintassi. Configuriamo anche eslint in modo da riconoscere la sintassi di ES7, di non generare un warning/errore per l'uso di console e di generarlo per variabili non utilizzate. In "scripts" ho tolto il comando relativo al test (per ora). I comandi di questa sezione si avviano con "npm start \<comando\>":

```
"scripts": {
    "dev": "NODE_ENV=development nodemon -w ./src",
    "lint": "eslint src"
  },
"eslintConfig": {
    "parserOptions": {
      "ecmaVersion": 7,
      "sourceType": "module"
    },
    "env": {
      "node": true
    },
    "rules": {
      "no-console": 0,
      "no-unused-vars": 1
    }
  },
```

Ho specificato l'ambiente "development" per NODE_ENV, in realtà è il valore di default che assegna NODE a questa variabile di ambiente a cui posso accedere via codice o in maniera diretta o in maniera indiretta tramite l'uso di Express:

```js
//diretta
var ambiente = process.env.NODE_ENV;
//indiretta
var ambiente = app.get('env');
```

La creazione di un server Express è pressapoco simile a quella vista con il modulo http. Express è derivato da Connect che a sua volta deriva dal modulo http:

```js
var express = require('express'); 
var path = require('path'); 
var logger = require('morgan'); 
var bodyParser = require('body-parser'); 
var routes = require('./config/routes');

var app = express();
var porta = 3000;

app.set('views', path.join(__dirname, '../static/views')); 
app.set('view engine', 'ejs');

app.use(logger('dev')); 
app.use(bodyParser.json()); 
app.use(bodyParser.urlencoded({ extended: false })); 

app.use('/', routes);

app.listen(porta);
console.log(`Server in ascolto sulla porta ${porta} - Ambiente: ${app.get('env')}`);

module.exports = app;
```

Con app.use indichiamo l'utilizzo dei nostri middleware, di terze parti come morgan e body-parser, o custom come routes. Impostiamo il template engine a EmbeddedJS (EJS) ed esportiamo la nostra app. Nota che il path fa uso di \_\_dirname, variabile che mi trovo nell'ambiente NodeJS che punta alla cartella del file in questione e cioè "src", per questo farò un "../".

## Routes

In NodeJS una route è il legame tra un endpoint ed una risora individuata da un URI. Una istanza express ha delle funzioni di gestione degli instradamenti: get, post, put e delete che accettano due parametri, l'URI e la funzione di gestione.

```js
app.get('/', function(req, res){
   ...
});
```

Spesso gli oggetti request e response sono abbreviati in req e res ed il parametro next omesso.

Le routes le modularizziamo a parte in un file routes.js nella cartella ./src/config utilizzando la classe express.Router, che ci ritorna un middleware:

```js
var express = require('express');

var app = express.Router();

app.get('/', function(req, res) {
  res.render('pages/index');
});

// about page 
app.get('/about', function(req, res) {
  res.render('pages/about',{
    nome: "Lorenzo",
    cognome: "Franceschini"
  });
});

module.exports = app;
```

## Middleware

[Middleware Ufficiali Express](https://github.com/senchalabs/connect#middleware)
Un middleware è una funzione che implementa una logica, la cui signature è del tipo function(request, response, next). La funzione next è la chiamata al successivo middleware (se previsto). Può passare un parametro che viene interpretato, dal successivo middleware, come un errore. Express si compone di tutta una serie di middleware che, a cascata e in ordine di dichiarazione, vengono eseguiti per arrivare dalla richiesta alla risposta.
Nella creazione del nostro server Express abbiamo utilizzato il middleware body-parser che, analizzato il body della request, popola il body per un accesso più facile rispetto a ciò che otterremmo utilizzado semplicemente url.
Morgan è un middleware per creare dei LOG in console. Può essere configurato con dei preset, in questo caso utilizzo 'dev' come preset.

Andiamo a definire i middleware di gestione dell'errore, inserendo in fondo al nostro file index.js (e prima della module.exports) il codice seguente. Qui si evidenza l'uso di next con il passaggio di un errore (POSSO SOLO PASSARE ERRORI CON NEXT):

```js
app.use(function(req, res, next) { 
  var err = new Error('Page Not Found'); 
  err.status = 404; 
  next(err); 
}); 
 
// error handlers 
 
// development error handler - stampa lo stacktrace 
if (app.get('env') === 'development') { 
  app.use(function(err, req, res, next) { 
    res.status(err.status || 500); 
    res.render('pages/error', { 
      message: err.message, 
      error: err 
    }); 
  }); 
} 
 
app.use(function(err, req, res, next) { 
  res.status(err.status || 500); 
  res.render('pages/error', { 
    message: err.message, 
    error: {} 
  }); 
}); 
```

## Views

## NodeJS ed ES6

# Note finali
Infine possiamo minificare i file js e css con GRUNT.
