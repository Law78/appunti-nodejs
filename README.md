# appunti-nodejs

*Se le soluzioni diventano complesse, probabilmente il problema è mal posto o non esiste (Lorenzo Franceschini)*

La logica delle funzionalità viene racchiusa in __moduli__. I moduli vengono inizializzati con la __require__ una funzione di commonjs (adottata da NodeJS) che effettua il cache dell'__oggetto__ ritornato. Pertanto per non avere cache devo ritornare una __function__ con la exports o con module.exports. Quando uso la __exports__ devo sempre aggiungere un oggetto: exports.getUser = ... e non assegnare direttamente exports!

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

NodeJS ci permette di gestire le richieste e le risposte del nostro server. Sicuramente gestiremo molti file statici (file HTML/immagini/fogli di stile/ecc...), successivamente dobbiamo essere in grado di gestire gli endpoints richiesti, ovvero gli indirizzi URL in entrata al server, che chiameremo __Routes__. Le routes verranno dirottate in specifici funzioni, o meglio moduli, che ne gestiranno la logica di backend.
Quindi gli step sono: __parse url__, __file statici__, __logica delle routes__. I file statici accedono al file system, mentre la logica delle routes solitamente ad un __database__.

Possiamo minificare i file js e css con GRUNT.

__REST__: In breve è una architettura web che definisce le regole di comunicazione tra client e server. Adottato come principale architettura da molto servizi web, fornendo l'accesso a degli endpoints ad una __risorsa__ tramite i principali __VERBS__ del protocollo HTTP. Solitamente in coppia con __JSON__ e sistemi di autenticazione basati su __TOKEN__.

Un importante, se non il più importante package di NodeJS è __Express__, un framework per lo sviluppo del server.
