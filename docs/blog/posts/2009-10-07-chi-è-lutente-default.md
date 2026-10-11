---
date: 2009-10-07
categories:
  - windows
---
# Chi è l'utente .Default?

Sfogliando la chiave di registro HKEY_USERS possiamo osservare che sono presenti tutte le chiavi HKEY_CURRENT_USER, organizzate per SID, di tutti gli utenti attualmente connessi al sistema.  
  
La HKEY_CURRENT_USER di ogni utente si trova nel file NTUSER.DAT all'interno di ogni profilo. Quando un utente effettua l'accesso al computer, il sistema operativo carica il file NTUSER.DAT nella HKEY_USERS assieme a tutti gli altri utenti, ed ogni utente nella HKEY_CURRENT_USER vede soltanto le sue chiavi.  
  
Oltre ai SID degli utenti, possiamo osservare che è presente un utente .Default. Ma chi è l'utente .Default?  
  
Naturalmente è ragionevole pensare che l'utente .Default sia utilizzato come profilo template per i nuovi utenti: modificando le impostazioni dell'utente .Default è logico aspettarsi che tutti i nuovi utenti creati nel sistema ereditino queste impostazioni.  
  
In effetti la scelta del nome .Default è stata infelice, e le cose vanno diversamente. Le chiavi di registro dell'utente .Default appartengono all'utente Local System, mentre il template di default per i nuovi utenti di sistema non è generalmente caricato in memoria e si trova nel percorso C:\Users\Default\NTUSER.DAT per Vista o Seven, oppure in C:\Documents and Settings\Default User\NTUSER.DAT per XP.  
  
Per modificare le impostazioni di default è possibile caricare manualmente questo Hive in memoria, modificarlo e salvarlo, anche se questo metodo non è ufficialmente supportato.  
  
Conviene utilizzare il buon SysPrep... oppure ho risolto il problema con un meraviglioso script WSH all'avvio.  
  
Per approfondire: [The .Default user is not the default user](<http://blogs.msdn.com/oldnewthing/archive/2007/03/02/1786493.aspx>).
