# appunti-nodejs

*Se le soluzioni diventano complesse, probabilmente il problema è mal posto o non esiste (Lorenzo Franceschini)*

Quando si parla di NodeJS, dobbiamo parlare subito di architettura __REST__ e del concetto che ciò di cui ci serviamo è visto come una risorsa, raggiungile attraverso una rotta detta __ROUTES__ tramite un comando chiamato più propriamento __VERBS__ del protocollo di comunicazione __HTTP__ che ci permette quindi di identificare in maniera univoca quello che si dice __URI__. La risorsa è rappresentata da un formato, definito dal Content-type e richiesta tramite l'Accept nell'header, che può essere __JSON__, __XML__, text o una forma binaria (es. image/jpg).
La comunicazione con la risorsa è considerata __Atomica__, per cui un comando porta in se tutta l'informazione necessaria alla modifica della risorsa stessa. Questo è ciò che porta una architettura REST ad essere __RESTful__ e cioè senza stati. Il fatto che la comunicazione è senza stato ci permette di scalare l'applicazione adottando dei carichi di bilanciamento e avere più nodi in risposta alle richieste, ad esempio utilizzando NGINX.
Una richiesta inviata al server genera una risposta, che, oltre a contenere la risorsa nel formato specificato, prevede l'invio di uno __STATUS__. I valori di status più comuni sono: 200 (OK), 201 (Risorsa Creata), 204 (No Content - usata per la DELETE) 400 (Richiesta non valida), 404 (Risorsa Non Trovata), 500 (Errore interno del Server).

__REST__: è una architettura web che definisce le regole di comunicazione tra client e server. Adottato come principale architettura da molto servizi web, fornendo l'accesso a degli endpoints ad una __risorsa__ tramite i principali __VERBS__ del protocollo HTTP. Solitamente in coppia con __JSON__ e sistemi di autenticazione basati su __TOKEN__, che risiedono nell'HEADER di una Request.
__VERBS__: sono i comandi che inviamo tramite il protocollo HTTP, i più importanti sono GET, POST, PUT e DELETE che, per convenzione in molti framework e testi, vengono associati ai comandi CRUD del mondo SQL. I comandi POST e PUT sono ambedue comandi di creazione di una risorsa, in genere se l'identificativo viene creato lato server (e quindi probabilmente una risorsa ex-novo) la POST è più appropriata della PUT. Se l'identificativo è fornito lato client o comunque è una risorsa conosciuta dal client, la PUT è più appropriata. Filosofia-time. I metodi __safe__ sono quelli che non modificano le risorse e cioè la sola GET (e OPTIONS e HEAD non menzionati) e pertanto possono essere anche cachati senza danni. I metodi __idempotenti__ sono quei metodi che posso chiamare n volte senza che il risultato cambi, come ad esempio la funzione che stampa "ciao" o i verbs PUT e DELETE. Il POST? Non è ne safe ne idempotent. Ciò è da tenere a mente per costruire API fault-tolerant.
__PROTOCOLLO__:
__TCP/IP__:
__HTTP__:
__ADDRESS__ e __PORT__:
__SOCKET__:

# NodeJS

_NodeJS ci permette di usare Javascript anche per il codice backend, NodeJS è single-thread con un cuore asincrono, NodeJS è EventDriven ed esegue il codice concorrentemente grazie all'event-loop. NodeJS è veloce, grazie all'engine Google V8 ed al sistema I/O non bloccante. NodeJS è immediato (ma non semplice per la sua natura asincrona). L'ordine temporale va oltre al pensiero logico della programmazione sincrona e del nostro modo di pensare. E' questo lo scoglio da superare._
Javascript è sincrono, NodeJS è asincrono.

Concetti da avere ben in mente per affrontare NodeJS: function come first-class citizen, callback, promise, ereditarietà prototipale, moduli, scope, closure sono concetti di Javascript, mentre sono relativi all'architettura NodeJS i concetti che stanno dietro [libvu](http://libuv.org) (la libreria scritta in C che interagisce con il Sistema Operativo), V8 (sincrono) che esegue il codice Javascript, l'event loop che gira internamente a libvu e costantemente controllato da libvu insieme alla coda degli eventi (event queue), ciò che esce dalla event queue passa a V8 e, se è un evento a cui è associata una callback, alla callback javascript che verrà eseguita nel momento in cui V8 termina l'attuale codice in esecuzione, in quanto V8 è sincrono. Questo sistema di gestione eventi di libvu è un meccanismo simile a quello che implementano i browser e che permette a Javascript di essere "asincrono" (es. chiamata Ajax o il setTimeout). Ma tutta questa architettura fa si che V8-libvu-NodeJS sia una architettura event-driven non bloccante per operazione di I/O. Tutte quelle operazioni di accesso ai file, accesso ad un database, accesso a dati via internet, avvengono a basso livello e gestiti grazie a libvu e al suo sistema di eventi e alla abilità di chiamare del codice Javascript tramite l'Event Emitter che vedremo più avanti.

__Libuv__: E' la libreria C che implementa l'event loop di Nodejs, la modalità asincrona della piattaforma, inoltre funge da libreria astratta per facilirare l'accesso ai vari sistemi operativi, come l'interazione al filesystem, sockets, eventi di sistema, database e threads.
__V8__: La libreria in C++ di Nodejs che "estende" Javascript fornendo una API che ci permette di creare oggetti, chiamare funzioni, ecc...

L'installazione di NodeJS ci fornisce oltre al comando node (un ambiente REPL), anche l'utilissimo Node Package Manager (npm) per l'installazione e creazione di moduli.
Per la creazione di un progetto utilizziamo il comando: npm init. Questo comando ci genera il file __package.json__ con i dettagli del progetto e i moduli che andremo ad installare sempre con il comando npm.

Alcune funzionalità sono disponibili a livello globale per Javascript, e quindi non ho bisogno di includere dei file esterni, come ad esempio l'uso di Buffer o \_\_dirname: [Global Objects](https://nodejs.org/api/globals.html#globals_global_objects)

### Convenzioni

La prima convenzione che mi appunto è quella detta Error-First Callback: il primo parametro di una callback è un parametro di errore, se non ci sono errori allora il suo valore sarà null.

*Sezione in lavorazione*


### Invocare funzioni C++

Sebbene questo argomento non è rilevante per poter utilizzare NodeJS, ho voluto scrivere due righe sull'argomento a mò di memoria personale e curiosità. 
Il codice Javascript che scrivo, NodeJS lo "invia" all'engine V8 (scritto in C++) che a sua volta lo trasforma in codice macchina utilizzabile sul processore che sto usando. E' possibile scrivere le proprie funzioni C++ che estendono V8 e mapparle per essere usate anche in Javascript, e quindi implementare nuove funzionalità esposte da NodeJS! That's cool!


E' possibile scrivere funzioni e oggetti in C++ e invocarli da Javascript tramite gli Addons.
Vediamo come mappare una funzione e i suoi argomenti passati da Javascript e il valore di ritorno passato da C++. Prima di tutto devo installare globalmente node-gyp globalmente, creo un file .cc e creo il file binding.gyp. Quello che stiamo per fare è equivalente a questo codice Javascript:

```
exports.greetings = function(arg1, arg2) { return arg1+arg2; };
```

Per fare il build del progetto abbiamo bisogno di un file di configurazione per node-gyp (precedentemente installato globalmente). Il file sorgente cc lo andiamo a creare in una cartella src.

```
//binding.gyp
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "src/test.cc" ]
    }
  ]
}

```

Scriviamo un piccolo codice in C:

```c
#include <node.h>
#include <stdio.h>

namespace piccolotest{

  void stampa(){
    printf("Eccomi\n");
  }
  
  int init() {
    printf("Ciao\n");
    stampa();
    return 1;
  }

  NODE_MODULE(piccolotest, init)

}
```

Ora mi basta un file Javascript per eseguire il codice C++ (in verità C):

```
const addon = require('./build/Release/addon');

addon;
```

Ora creiamo un altro file di configurazione (volendo posso usare lo stesso e aggiungere più sources all'interno dell'array):

```
//binding.gyp
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "src/greetings.cc" ]
    }
  ]
}
```

Vediamo come definire un metodo ad un oggetto che utilizzero all'interno di Javascript con la sintassi V8.
Per definire un metodo la signature sarà del tipo:

```
void Method(const v8::FunctionCallbackInfo<v8::Value>& args) 
```

mentre il bind di questo metodo in Javascript avviene con questa riga di codice:

```
NODE_SET_METHOD(target, "metodo", Method);
```

Creo il file greetings.cc

```cpp
// greetings.cc
#include <node.h>

namespace greetings {

using v8::Exception;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void Greetings(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  // Controllo il numero di argomenti passati
  if (args.Length() < 2) {
    // Throw an Error that is passed back to JavaScript
    isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate, "Wrong number of arguments")));
    return;
  }

  // Controllo il tipo degli argomenti
  if (!args[0]->IsString() || !args[1]->IsString()) {
    isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate, "Wrong arguments")));
    return;
  }

  Local<String> concatena = String::Concat(args[0]->ToString(isolate), args[1]->ToString(isolate));
  args.GetReturnValue().Set(concatena);
}

void Init(Local<Object> exports) {
  NODE_SET_METHOD(exports, "greetings", Greetings);
}

NODE_MODULE(addon, Init)

} 
```

Lancio i comandi:

```bash
node-gyp configure
node-gyp build
```

ed il file greetings.js:

```js
const addon = require('./build/Release/addon');

console.log(addon.greetings("hello","world"));
```

e con node greetings.js ottengo: "helloworld" :D

*Sezione in lavorazione*


https://nodejs.org/api/addons.html

### Compilare V8

Passo veloce: brew install v8

Passi standard:
I passi che seguono servono per compilare V8 su MacOS Sierra, ed è una continuazione della sessione precedente.
Prima di tutto ho clonato il repo di V8: git clone https://chromium.googlesource.com/v8/v8
Poi ho installato [depot_tools](http://dev.chromium.org/developers/how-tos/depottools) per gclient: git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
rendendolo disponibile nel PATH, nel mio caso zsh: vim ~/.zshrc e ho inserito:
export PATH=~/Documents/NodeJS/depot_tools:"$PATH"
Poi dalla cartella immediatamente superiore a v8 (del primo step) ho fatto: gclient config https://chromium.googlesource.com/v8/v8
Poi entro nella cartella v8 e scrivo: gclient sync
ed infine faccio: make native

Come compilare la shell.cc di sample e avere un REPL javascript:

"clang++ -std=c++11 -I. -Iinclude samples/shell.cc -o samples/shell out/native/*.a"
"cp out/native/*.bin samples/" 
"./samples/shell"

a questo punto ho la mia shell dove posso scrivere codice Javascript e utilizzare anche funzioni non Javascript come print, definita in shell.cc oppure definire una mia funzione in 3 semplici passi, la dichiaro, la lego al contesto e la scrivo ;)

```
void Ciaone(const v8::FunctionCallbackInfo<v8::Value>& args);

...

global->Set(
      v8::String::NewFromUtf8(isolate, "ciaone", v8::NewStringType::kNormal)
          .ToLocalChecked(),
      v8::FunctionTemplate::New(isolate, Ciaone));
      
...

void Ciaone(const v8::FunctionCallbackInfo<v8::Value>& args) {
  printf("Ciaone!!!");
  printf("\n");
  fflush(stdout);
}
```

e ora in shell posso scrivere ciaone() ^^

### Module Pattern

La logica delle funzionalità viene racchiusa in __moduli__. I moduli vengono inizializzati con la __require__ una funzione che effettua il cache dell'__oggetto__ ritornato (da module.exports). Pertanto per _non avere cache_ devo ritornare una __function__ con la exports o con module.exports. Quando uso la __exports__ (un alias di module.exports) devo sempre aggiungere un oggetto: exports.getUser = ... e non assegnare direttamente exports!
Il modulo richiamato da NodeJS viene processato dalla librearia Module di NodeJS che crea un oggetto module.exports che carica il nostro file e crea un funzione che wrappa il nostro file. In realtà tutto ciò che scrivo in NodeJS è incluso in questa funzione che crea NodeJS:

```js
(function(exports, require, module, __filename, __dirname){
  var hello = function(){
    console.log('Hello World!');
  };
  
  module.exports = hello;
});
```

e verrà eseguita in questo modo:

```js
fn(module.exports, require, module, filename, dirname);
```

Nota il legame tra module.exports e l'alias exports creato, ricorda che se volessi usare exports al posto di module.exports, devo sempre agganciare una proprietà, mutando il mio oggeto module.exports, altrimenti sovrascrivo questo legame:

```js
exports.foo = function(){}
```

Capire perchè il mio codice è wrappato in questo modo è importante perchè non solo il mio modulo è protetto (function scope) da eventuali altri moduli che hanno stessi nome, ma è importante perchè potrei incorrere nell'errore che il mio file app.js iniziale possa essere legato al this del global scope, mentre sarà il this della funzione wrap. Il module.exports è l'oggeto che ritorna la require.

Ovviamente le modalità di ciò che esporto sono varie, posso ancora esporta un oggetto oppure sovrascrivere l'oggeto ed esportare una funzione (ed evitare il cache - vedi avanti):

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

In questo modo sovrascrivo il mio oggetto module.exports con una funzione. Posso esportare con module.exports una proprietà che contiene una funzione:

```js
module.exports.bar = function(){
  console.log("I'm bar, where's foo?");
}
```

e che userò in questo modo:

```js
var foo = require('./foo');
foo.bar();

// oppure:
var foo = require(('./foo').bar;
foo();
```

Posso sovrascrivere l'oggetto module.exports con il mio oggetto che posso costruire con la Class, con Object.create o con una funzione costruttore:

```js
function Foo(){
  this.foo = "I'm foo instance!";
  this.greet = function(name){
    console.log(`Hi ${name}`);
  }
}

module.exports = new Foo();
```

e che userò in questo modo:

```js
var foo = require('./foo');
foo.greet('Lorenzo');
```

Ricorda, in questo modo però, se ho più require di questo stetto file, essendo un oggetto, avrò sempre lo stesso oggetto per via del caching che effettua NodeJS (cachedModule). Per evitare il cache, mi basterà cambiare il module.exports ed esportare Foo e non l'oggetto creato:

```js
...
module.exports = Foo;
```

che userò con la new, perchè funzione construttrice.
Infine possiamo esporre un oggetto, che a sua volta espone solo ciò che consideriamo pubblico e tralasciando fuori ciò che consideriamo privato:

```
var bar = "Valore privato";
var foo = function pubblica(){}

module.exorts = {
  foo: foo
}

```

Posso usare export e import di ES6? Certamente, ma con Babel (che vedrò più avanti):

```js
export function func(){}
export default {}
```

```js
import * as func from './foo';
oppure
import func as func from './foo';
...
```

### Eventi

NodeJS è fortemente orientato agli eventi, per questo è necessario avere una buona conoscenza di cos'è un evento. Un evento è qualcosa che succede, in un tempo non determinato, a cui la nostra applicazione risponde tramite un event listener, ioè un handler, una funzione. In NodeJS abbiamo eventi di sistema, che gestiamo con il core di V8 (tramite la libreria libvu), scritti in C++, ed eventi custom, scritti in Javascript, emessi da Event Emitter. Quest'ultimi saranno gli eventi con cui abbiamo a che fare. Di fatto la parte di libreria scritta in Javascript per la gestione degli eventi non è altro che un wrapper per gli eventi gestiti in realtà da libvu, quindi la parte Javascript è una "decorazione", un "fake" di eventi reali emessi da libvu.

Come funziona un gestore di eventi? Proviamo a scrivere del codice, funzionante, che simula una coda di eventi e che per ogni evento è possibile associare uno o più listeners. L'idea è quella di avere un oggetto, a cui è possibile associare un numero dinamico di eventi, ogni evento sarà una array il cui contenuti è una funzione listener. Il codice che segue fa uso della funzione costruttrice per ritornare un oggetto vuoto, e due prototype "on" ed "emit" per aggiungere ed emettere l'evento. E' plausibile avere più di un oggetto Emitter, ognuno con un suo elenco di eventi, magari diviso per logica di dominio, ecco perchè si usa una funzione costruttrice. Inoltre il prototype farà si che queste funzioni non siano copiate ad ogni istanza creata, ma condivise da tutte le istanze. Per comprendere questo codice ricordati che in Javascript le funzioni sono first-class citizen e che un oggetto, oltre ad accedere alle proprietà con la dot notation, posso utilizzare la brackets notation per la gestione dinamica delle proprietà. Dal codice [EventEmitter](https://github.com/nodejs/node/blob/master/lib/events.js), una versione semplificata:

```js
function Emitter(){
  this.events = {}
}

Emitter.prototype.on = function(event, listener){
// se l'evento esiste già come proprietà dell'oggetto allora utilizzalo, altrimenti crealo come nuovo array
  this.events[event] = this.events[event] || [];
// inserisci per questo array la funzione listener
  this.events[event].push(listener);
}

Emitter.prototype.emit = function(event){
  if(this.events[event]){
    this.events[event].map(function(listener){
      listener();
    });
  }
}

module.exports = Emitter;
```

Per usare questo codice, creo un emitter.js e un config.js

```js
var Emitter = require('./emitter');
var events = require('./config').events;

var emitter = new Emitter();

emitter.on('phone-ring', function(){
  console.log('Hey, rispondo al telefono!');
});

emitter.on(events.PHONE_RING, function(){
  console.log('Hey, ho risposto anche io!!!');
});

emitter.emit(events.PHONE_RING);
```

```js
module.exports = {
  events: {
    PHONE_RING: 'phone_ring'
  }
}
```

L'Event Emitter di NodeJS, linkato in alto, ovviamente è più complesso, prevede non solo delle ottimizzazioni di gestione, come il clone di Array oppure l'inserimento iniziale del listener direttamente nella proprietà e di convertire tale proprietà in array se e solo se ho più di un listener. Prevede l'impostazione di un domain, di un max_listeners, di passare degli argomenti, il check del listener che sia una funzione, ecc... . Il pattern utilizzato da NodeJS è il seguente:

```js
function Emitter(){
  Emitter.init.call(this);
}

Emitter.init = function(){
  this.events = {}
}

module.exports = Emitter;
```

E' un pattern che si trova in molte librerie e framework di Javascript. Vediamo un esempio più complesso, e quindi più simile all'implementazione reale di NodeJS. Dovresti essere in grado di capirlo:

```js
function Emitter(){
  Emitter.init.call(this);
 }

Emitter.prototype.on = function on(event, listener){
  if (typeof listener !== 'function')
    throw new TypeError('Il "listener" deve essere una funzione!');
  if (!this.events) {
    this.events = {};
  }
  this.events[event] = this.events[event] || [];
  this.events[event].push(listener);
}

Emitter.prototype.emit = function emit(event){
// Controllo se sono stati registrati eventi
  if(!this.events)
    return;
  len = arguments.length;
  handler = this.events[event];
  switch (len) {
     // fast cases
    case 1:
      emitNone(handler, this);
      break;
    case 2:
      emitOne(handler, this, arguments[1]);
      break;
    default:
      args = new Array(len - 1);
      for (i = 1; i < len; i++)
        args[i - 1] = arguments[i];
      emitMany(handler, this, args)
  }
}

function emitNone(handler, self) {
  var len = handler.length;
  var listeners = arrayClone(handler, len);
  for (var i = 0; i < len; ++i)
    listeners[i].call(self);
}

function emitOne(handler, self, arg1) {
  var len = handler.length;
  var listeners = arrayClone(handler, len);
  for (var i = 0; i < len; ++i)
    listeners[i].call(self, arg1);
}

function emitMany(handler, self, args){
  var len = handler.length;
  var listeners = arrayClone(handler, len);
  for (var i = 0; i < len; ++i)
    listeners[i].apply(self, args);
}

function arrayClone(arr, i) {
  var copy = new Array(i);
  while (i--)
    copy[i] = arr[i];
  return copy;
}

Emitter.init = function(){
  this.events = {}
}

module.exports = Emitter;
```

Questa versione fa uso del pattern di inizializzazione, controlla che il listener sia una funzione, inoltre possibile passare degli argomenti:

```js
var Emitter = require('./emitter');
var events = require('./config').events;

var emitter = new Emitter();

emitter.on(events.PHONE_RING, function(name){
  console.log('Hey, Sta chiamando:', name);
});

emitter.emit(events.PHONE_RING, 'Teresa');
```

Ora posso fare una prova, togliere il mio codice emitter sostituendo la prima riga con questa che utilizza il codice di NodeJS e verificare che il comportamento di fatto è identico:

```js
var Emitter = require('events');
```

NodeJS ci permette di creare oggetti che estendono da EventEmitter utilizzando la libreria util e il metodo inherits, che richiede il passaggio di due funzioni costruttrici (e non gli oggetti finali) e l'effetto è quello di modificare il prototype-chain della funzione costruttrice di sinistra, legandola con la funzione costruttrice di destra:

```js
var Emitter = require('./emitter');
var util = require('util');

function Persona(){
  Emitter.call(this);
  this.nome = "Lorenzo";
}

util.inherits(Persona, Emitter);

Persona.prototype.mangia = function mangia(cibo){
  console.log('Mangia', cibo);
  this.emit('mangia');
}

persona = new Persona();

persona.on('mangia', function(){
  console.log('Ho mangiato qualcosa');
});

persona.mangia('pasta');
```

Emitter è la nostra classe che simula Event Emitter di NodeJS, Persona è una funzione costruttrice, che ora può accedere ai metodi definiti nel **prototype** da Emitter (puoi provare ad usare anche Events di NodeJS) grazie a **util.inherits**. Quello che di fatto fa util.inherits è, oltre ad una serie di controlli sul typeof, è l'uso di "Object.setPrototypeOf(Persona.prototype, Emitter.prototype);
", che è di fatto equivalente a: "Persona.prototype.\_\_proto\_\_ = Emitter.prototype;" [Approfondimento](https://developer.mozilla.org/it/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)
E' importante osservare la chiamata **Emitter.call(this)** che faccio nella funzione costruttore. Questa chiamata di fatto è la chiamata al costruttore base, al super constructor, che mi permette di legare al mio oggetto this eventuali proprietà e funzioni che Emitter crea nel suo costruttore direttamente all'oggetto this. Se non facessi questa chiamata, avrei come risultato l'accessibilità alle sole proprietà e funzioni che Emitter lega al prototipo e perderei le eventuali proprietà e funzioni legate direttamente al this perchè il __costruttore di Emitter non verrebbe invocato(!!!)__. In questo modo la mia ereditarietà è completa: eredito tutto ciò che fa parte del prototipo più tutto ciò che la classe da cui derivo aggancia al this!

### Richieste e Risposte: il nostro Server

NodeJS ci permette di gestire le richieste e le risposte del nostro server. Sicuramente gestiremo molti file statici (file HTML/immagini/fogli di stile/ecc...), successivamente dobbiamo essere in grado di gestire gli endpoints richiesti, ovvero gli indirizzi URL in entrata al server, che chiameremo __Routes__. Le routes verranno dirottate in specifici funzioni, o meglio moduli, che ne gestiranno la logica di backend.
Quindi gli step sono: __parse url__, __file statici__, __logica delle routes__. I file statici accedono al file system, mentre la logica delle routes solitamente ad un __database__.
Il core di NodeJS ci fornisce gli strumenti base per definire un nostro server che gestisca le richieste (request o req) e le risposte (response o res) grazie al modulo __http__, il parse delle url e cioè l'analisi dell'indirizzo digitato e quindi la consapevolezza di ciò che ha chiesto l'utente e di eventuali query string passate, tramite il modulo __url__, e la gestione del file system con __path__ e __fs__. Qui un semplicissimo esempio che mette in evidenza che il browser effettua, più di una request, in questo caso oltre la request alla root (/) di localhost:3000, chiederà anche la favicon.ico:

```js
//app.js
var http = require('http');

var server = http.createServer(function(req, res){
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Ciao!\n');

});

server.listen(8000,'127.0.0.1');

module.exports = server;
```

Posso verificare le due richieste anche nella sezione "Network" degli strumenti di sviluppo di Google Chrome. Cliccando sula richiesta localhost (dovresti vedere anche quella di favicon) puoi analizzare l'intestazione (Header)  sia della request che della response.


Oppure un'altra versione, leggermente diversa in cui specifico solo la porta e la funzione di callback a parte e applico solo delle console.log a terminale:

```js
//app.js
var http = require('http');

var server = http.createServer(handleRequest).listen(3000);

function handleRequest(request, response){
  console.log('Arrivata Richiesta', request.method);
  console.log(request.url);
  response.end();
}

console.log('Server in ascolto');
```

Creiamo un server di esempio che fornisce come risposta un file index.html, che creiamo semplicemente con un tag h1 'Ciao Mondo', il nostro server sarà:

```js
//app.js
var http = require('http');
var fs = require('fs');

var server = http.createServer(function(req, res){
  res.writeHead(200, {'Content-Type': 'text/html'});
  var html = fs.readFileSync(__dirname + '/index.html');
  res.end(html);

});

server.listen(8000,'127.0.0.1');

module.exports = server;
```

Il server creato di fatto è un oggeto di EventEmitter, che gestisce l'evento tramite una callback che ha due oggetti request e response. La request, come abbiamo visto, porta con se vari proprietà tra cui il __method__ e l'__url__. La request, anch'essa EventEmitter, implemeta uno stream di ascolto per l'invio dei dati contenuti nel body. Questo stream deve essere canalizzato (pipe) in modo da ricomporre i cari chunk con gli eventi __data__ e __end__, e gestire eventuali errori con l'evento __error__:

```js
//app.js
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
//app.js
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

# Test

Riprendiamo la prima versione del nostro server della sezione precedente, in cui ho esportato il server e la listen come chiamata successiva. Introduciamo una breve sezione per il testing del nostro server NodeJS. Installiamo i seguenti tools: npm install --save-dev mocha chai request e (se non l'ho fatto fate npm init) nel package.json andiamo ad inserire il comando "script" per lanciare mocha:

```
//package.json
...
"scripts": {
    "test": "./node_modules/.bin/mocha --reporter spec"
  }
...
```

creo una cartella test con un file serverTest.js:

```js
//serverTest.js
var chai = require('chai');
var request = require('request');
var expect = chai.expect;
var server = require('../app');

describe('server response', function () {
  before(function () {
    server.listen(8000);
  });
  after(function () {
    server.close();
  });
  it('should return 200', function (done) {
    request.get('http://localhost:8000', function (err, res, body){
      console.log(res.statusCode)
      expect(res.statusCode).to.equal(200);
      expect(res.body).to.equal('Ciao!\n');
      done();
    });
  });
});
```

Lanciamo il test con "npm run test".
Proviamo a fare il test del server che fornisce come risposta la pagina index.html con il tag h1. Lanciando questo test, ovviamente la expect fallirà. Modifichiamo il test:

```js
var chai = require('chai');
var request = require('request');
var expect = chai.expect;
var server = require('../index');
var fs = require('fs');

describe('server response', function () {
  before(function () {
    server.listen(8000);
  });

  after(function () {
    server.close();
  });
  it('should return 200', function (done) {
    request.get('http://localhost:8000', function (err, res, body){
      var html = fs.readFileSync('index.html','utf8');
      expect(res.statusCode).to.equal(200);
      expect(res.body).to.equal(html);
      done();
    });
  });
});
```


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
