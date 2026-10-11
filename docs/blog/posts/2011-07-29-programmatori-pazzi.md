---
date: 2011-07-29
categories:
  - sicurezza
---
# Programmatori pazzi

Bene o male, ma soprattutto male, qualunque sviluppatore impara a scrivere qualche piccola applicazione per il Web.  
  
Magari abbiamo scritto la nostra applicazione per tenere traccia delle attività da svolgere, come pagare la bolletta del metano, ricordarsi di revisionare la macchina, portare Tobia alla toelettatura per cani, consegnare un nuovo prototipo al cliente, o pagare gli alimenti per la prole alla ex fidanzata. L'accesso all'applicativo sarà previa autenticazione, e gli utenti potranno inserire i propri dati, leggerli in tabella, ricercarli, filtrarli.  
  
Poi che cosa manca? Gli utenti, oltre ad interagire con l'applicativo Web, vorranno scaricare i dati e portarli sul telefonino, farsene una copia su chiavetta, o importarli in Excel e farci qualche piccola elaborazione.  
  
Da qualche parte nella pagina si dovrà predisporre un link o una paginetta di download, e gli utenti, dopo aver aggiustato qualche filtro, potranno scaricare il file sul proprio client e gestirlo come preferiscono.  
  
E quindi, il programmatore medio che cosa si inventa? Generalmente la tecnica scelta si attesta intorno a queste soluzioni:  
  

  1. si crea una bella _crontab_ sul server, con una riga che viene eseguita ogni tot giorni, che crea una estrazione per ogni utente in _\var\www\home\downloads\attivita-nomeutente.dat_ ;
  2. la pagina di download, quando viene richiamata, inizia a scrivere un file in _\var\www\home\downloads\attivita-nomeutente.dat_. Una volta competato il file, viene mandato in output un bel link all'estrazione appena creata;
  3. la pagina di download richiama un processo sul server che inizia a scrivere un file in _\var\www\home\downloads\attivita-nomeutente.dat_. Poichè il processo è asincrono, si mette uno sleep di 5 secondi in modo da dare il tempo al processo asincrono di completare il file. Poi ci mettiamo un bel link al file appena generato.

Notate qualche cosa di strano?  
  
La prima tecnica è molto limitata: come possiamo applicare dei filtri? Ma uno sviluppatore creativo è in grado di risolvere anche questo: in fondo, quanto ci vuole a creare in anticipo tutti i file con tutti i filtri possibili ed immaginabili, tutti già preimpostati?  
Per la seconda tecnica, uno script con privilegi di scrittura su di una cartella che poi verrà servita tramite server HTTP è il sogno di ogni criminale informatico. Ma se proprio non ci sono altre possibilità, conviene almeno sapere che oltre al _chmod 777_ o all'_Everyone Full Control_ esistono anche svariate vie di mezzo.  
Per la terza ogni commento è superfluo. Posso solo giurare di aver visto anche questo!  
  
Se però non avete notato altro, forse la sicurezza informatica non è il vostro mestiere, e probabilmente non lo sarà mai.  
  
Il vero problema, grosso come un pilone portante del Burj Khalifa di Dubai, è che se mettiamo un file in _\var\www\home\downloads\_ l'utente vi potrà accedere felicemente andando al link _http://www.example.com/downloads/attivita-nomeutente.dat_ ma oltre al nostro amico autorizzato vi potrà entrare anche tutto il resto del mondo. Certo, disabilitare il browsing delle directory è senz'altro un'idea brillante, ma non basta.  
  
Il concetto è che al nostro Web Server non importa assolutamente nulla dell'autenticazione che abbiamo sviluppato nel nostro programma: abbiamo implementato un sistema di autenticazione per le nostre paginette Web? Bene. Abbiamo utilizzato un framework che implementa la parte di sicurezza? Meglio ancora. Ma questo sistema lo dobbiamo implementare anche per i nostri file. Solo allora possiamo servire le nostre pagine, dinamicamente, e protette dalle nostre credenziali.  
  
(Ci sono casi in cui generare precedentemente i file da far scaricare agli utenti è cosa buona e giusta; tra l'altro il Web Server è sicuramente molto più veloce del nostro script nel servire le pagine; in questi casi però cerchiamo almeno di dare ai file un indirizzo URL casuale).
