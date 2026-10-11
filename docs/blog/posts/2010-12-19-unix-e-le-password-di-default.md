---
date: 2010-12-19
categories:
  - sicurezza
---
# Unix e le password di default...

Qualche settimana fa ricevo un avviso automatico che segnala un probabile malfunzionamento ad un apparato hardware che abbiamo in sala macchine.  
Ora, generalmente non mi occupo di queste cose, ma visto che in ufficio non c'era nessun altro prendo in carico la segnalazione e chiamo l'assistenza:   
  
**Tecnico help desk** : _Sì, sembrerebbe proprio un guasto hardware, se è così faccio subito uscire un tecnico per la riparazione. Possiamo fare un po' di troubleshooting per avere conferma che si tratti di proprio un guasto hardware?_  
**Io** : _Certamente... ma mi deve spiegare come fare perché non è un apparato che conosco o che gestisco io!_  
**Tecnico help desk** : _Allora intanto si colleghi alla console...._  
**Io** : V _a bene! Mi dia un attimo di tempo che devo recuperare la password..._  
**Tecnico help desk** : _La password? Nessun problema, quella gliela posso comunicare io!_  
**Io** : _Ah, però! Conoscete le password degli apparati dei clienti? Che efficienza! :-)_  
**Tecnico help desk** : A _llora, le detto le credenziali: l'utente è root, e la password abcde123_  
**Io** : _Ehm, abcde123? Ma cos'è? La password di default?? Perché c'è una leggera probabilità che nella nostra azienda le password di default vengano modificate..._  
**Tecnico help desk** : (silenzio)  
**Io** : _Ehm, ma è la prima volta che vi capita un cliente che modifica la password dei propri apparati?_  
**Tecnico help desk** : (silenzio)  
  
Una volta recuperata la vera password, ed una volta confermato il guasto hardware, nel giro di un paio d'ore arriva il tecnico on site:  
  
**Tecnico on site** : _Questo è il pezzo da sostituire. Ora dobbiamo entrare nella console e disabilitare quello guasto prima di scollegarlo._  
**Io** : _OK, mi collego alla console..._  
**Tecnico on site** : _Perfetto... l'utente è root e la password è abcde123...._  
  
Sistemato anche questo, chiamo il nostro consulente e chiedo conferma che il problema sia stato risolto:  
  
**Consulente** : _Allora! Prima di lasciare andare via il tecnico, puoi fare qualche controllo?_  
**Io** : _Certamente! Basta che mi spieghi come fare..._  
**Consulente** : _Allora, ti colleghi alla console, la password è abcde123 e...._  
  
Uhm! Ho l'impressione che nel mondo Unix la sicurezza non sia sempre presa molto sul serio...
