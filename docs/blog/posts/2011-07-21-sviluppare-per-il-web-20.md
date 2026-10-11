---
date: 2011-07-21
categories:
  - web
---
# Sviluppare per il Web 2.0?

Ho iniziato a sviluppare applicativi Web aziendali, pubblicati sulla Intranet, fin dal 1998.  
Al tempo l'azienda non era nemmeno collegata ad Internet, per cui non avevo ancora a disposizione una casella di posta elettronica aziendale, e le linee di collegamento tra le varie sedi distaccate erano decisamente molto lente.  
  
La piattaforma per cui sviluppavo era Windows NT, con Internet Information Server, e gli applicativi erano realizzati con tecnologia ASP in linguaggio VBScript. Il database era spesso un Microsoft Access (dite quello che volete, ma si tratta di un prodotto miracoloso!) o, quando eravamo fortunati, un SQL Server.  
  
Dopo pochi anni sono passato allo sviluppo per Apache su Linux, ma ho accuratamente scelto di evitare il PHP. So che il PHP è molto amato dalle comunità di sviluppo, ma quando si impara a conoscere un prodotto come ASP, passare al PHP sembra fare salto indietro di secoli. In ASP c'era un solo RecordSet, lo stesso RecordSet di Access, lo stesso di SQL Server, lo stesso di qualsiasi altro database di sistema. Non un RecordSet simile con le stesse funzioni: era proprio la stessa libreria di sistema: di per se, ASP era un sistema molto scarno, ma che si appoggiava largamente alle librerie di Windows.  
  
Nel mondo PHP invece ogni diverso database aveva del codice diverso per essere gestito. La connessione ad un MySql era diversa dalla connessione ad un PostgreSQL, diversa dalla connessione ad un ODBC, diversa la connessione da una qualsiasi altra fonte dati. Diversa da qualsiasi connessione a qualsiasi database sviluppato per la console di Linux o per qualche applicativo client server.  
  
(A vete intuito per quale ragione nel mondo Windows si parla di DLL Hell, mentre nel mondo Linux no? Ho il sospetto che nel mondo Linux vada di moda la riscrittura del codice piuttosto che la condivisione delle librerie... nonostante questo, avrei comunque qualche bella storia di .so Hell da raccontare!)  
  
Per cui ho deciso che lo strumento per cui avrei sviluppato sarebbe stato il linguaggio Perl.  
Non si tratta di un linguaggio per puristi, ma in fondo chi se ne importa? Ho programmato in qualsiasi cosa mi sia capitata per le mani, e l'ho trovato fin da subito un prodotto molto efficiente. E pazienza che la sintassi fosse orrenda e poco coerente: una volta abituati al VBScript, non si può che migliorare!  
  
La cosa che mi ha convito fin subito del Perl era la sua immensa collezione di librerie, per connettersi o per gestire qualsiasi cosa (database, periferiche, API di sistema, c'era addirittura del codice per gestire il proprio account MySpace, il tutto via codice...!). Generalmente le librerie del Perl erano di buona qualità e progettate con buon gusto. Inoltre, da non trascurare il fatto che la stessa libreria che si utilizzava per sviluppare applicativi di sistema era la stessa libreria da utilizzare per lo sviluppo Web: avevo finalmente trovato la coerenza anche nel mondo Linux!  
  
Ma soprattutto avevo trovato il miglior framework per il Web esistente sulla terra: [HTML::Mason](<http://www.masonhq.com/>)!  
Il suo concetto di ereditarietà era favoloso: potevi concentrarti nello scrivere una paginetta che mostrava il contenuto della pagina, e tutto il resto, dal vestito grafico, alle connessioni al database, alle autorizzazioni necessarie, venivano ereditate secondo uno schema deciso in fase di progettazione.  
  
Purtroppo si trattava di un sistema molto difficile da installare e da configurare (non vi sto a raccontare di tutti i conflitti di librerie di sistema per riuscire a farlo funzionare... quando inizi ad utilizzare componenti che non sono presenti in una distribuzione e sono scarsamente utilizzati, anche il mondo dei pacchetti e delle dipendenze mostra i suoi limiti) ma una volta predisposto il tutto diventava uno strumento davvero veloce per sviluppare applicazioni Web complesse.  
  
L'ultimo applicativo che ho sviluppato era una specie di Cartella Clinica, con dati dei pazienti, variazioni anagrafiche, esami di laboratorio, esami radiologici, prescrizioni farmaceutiche, il tutto visualizzabile via Web oppure scaricabile in formato leggibile dal software del medico, previo accesso tramite utente e password.  
  
Le fonti di dati erano MySql, Oracle, SQL Server (eh sì, vi si poteva accedere anche da Linux!), o file di testo importati tramite crontab.  
Le pagine erano realizzate in XHTML, con CSS, con qualche bello spunto preso da [CSS Zen Garden](<http://www.csszengarden.com/>). JavaScript, come era giusto fare al tempo, era ridotto al minimo e si limitava a pre-convalidare i Form.  
  
Poi si sa come vanno le cose nella pubblica amministrazione. Lo sviluppo interno viene abbandonato, e viene appaltato all'esterno. Così il mio lavoro non consiste più nel progettare e realizzare programmi, ma consiste nel riuscire a far funzionare programmi scritti da terzi, con tecniche di programmazione arcaiche e/o discutibili, e che ~~probabilmente~~ sicuramente avrei potuto scrivere meglio!  
  
Negli ultimi cinque anni il mondo del Web è cambiato, il nuovo Web si chiama Web 2.0.  
Cosa bisogna imparare per restare al passo?  
Ho scelto di utilizzare il Google App Engine, questo richiede di imparare a programmare in Python ma (database a parte) tutto ciò che si utilizzava sotto Apache è ancora molto attuale. E all'autenticazione, il lato debole di ogni applicativo Web, non ci pensa più il programmatore ma se ne occupa Google!  
La lista seguente serve soprattutto a me, per capire che cosa è cambiato dal punto in cui ero rimasto. Non è definitiva e la aggiornerò quando serve:  
  

  * XHTML è stato abbandonato. Il successore di HTML4.01 è HTML5
  * Un applicativo, per essere funzionale ed apparire moderno, deve utilizzare AJAX
  * Utilizzare JavaScript non è più un tabù!
  * Creare a mano il proprio JavaScript è tabù: dove possibile, utilizzare JQuery o altre librerie standard!
  * Il framework standard di Google App Engine è Django. Non so se sarà il mio preferito, forse dovrei approfondire Pylons (che deriva in qualche modo da HTML::Mason)
  * Non bisogna dimenticarsi di integrare il proprio applicativo con Facebook e altri social networks, come ad esempio Google+.

Happy coding!
