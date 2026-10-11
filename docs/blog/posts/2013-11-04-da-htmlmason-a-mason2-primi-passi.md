---
date: 2013-11-04
---
# Da HTML::Mason a Mason2... primi passi

Attenzione, questa pagina potrebbe contenere inesattezze: sono soltanto degli appunti su cui sto ancora lavorando!  

###  Installazione Mason2
La nuova versione di HTML::Mason si chiama semplicemente Mason, ed è un sistema per la creazione di template e contenuti dinamici.  
  
A differenza di HTML::Mason, la nuova versione si occupa di gestione pura dei template ed è stata resa indipendente dall'ambiente Web, la parte di integrazione con il server Web è invece che è realizzata dal framework Poet, tramite utilizza PSGI/Plack.  
  
L'installazione di Poet si può fare in questo modo:   
  

    
    cpanm -S --notest Poet
    
  
e poiché Mason è un prerequisito, verrà installato anch'esso. _cpanminus_ è un componente per la gestione di pacchetti presenti in CPAN.   
  
Installiamo anche _Mason::Plugin::PSGIHandler_ con il quale possiamo creare il nostro primo sito funzionante:   
  

    
    mkdir /var/wwwmason
    mason_psgi_setup /var/wwwmason/firstapp/
    
  
ora per lanciare il nostro sito possiamo usare il seguente comando   
  

    
    cd /var/wwwmason/firstapp/; plackup -r
    
  
e il nostro applicativo sarà in ascolto su _http://localhost:5000_  
se vogliamo configurare Apache per caricare il nostro sito, proviamo così:  
  

    
    <Location "/firstapp">
    
      SetHandler perl-script
      PerlResponseHandler Plack::Handler::Apache2
      PerlSetVar psgi_app  /var/wwwmason/firstapp/app.psgi
    
    
    </Location>
  
attenzione che sembra convenga utilizzare _plackup -r_ per lo sviluppo, mentre la soluzione su Apache va bene per i sistemi in produzione ma risulta difficoltoso ricaricare l'applicativo senza fare il restart di Apache: [How do you deploy a PSGI script in Apache without restarting?](<http://stackoverflow.com/questions/6124010/how-do-you-deploy-a-psgi-script-in-apache-without-restarting>)  
  
verificare anche i permessi della directory _/var/wwwmason/firstapp/data_ dove Mason va a creare la cache ed i file precompilati.
