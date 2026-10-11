---
date: 2009-10-12
categories:
  - windows
  - sicurezza
---
# Bloccare gli aggiornamenti software

Disabilitare gli aggiornamenti del sistema operativo oppure del software applicativo è in genere **una pratica da evitare** : un sistema aggiornato è più stabile, e molto più sicuro!  
  
Ad oggi le poche aziende che non hanno attivato un sistema di aggiornamento automatico per il sistema operativo Windows ne pagano le conseguenze (vedi ad esempio Conficker). Ed è curioso notare che, poiché un sistema Windows **aggiornato e configurato adeguatamente** offre sistemi di sicurezza avanzati e difficili da aggirare, si sta assistendo ad un cambio di tendenza e l'obiettivo principale degli attacchi del malware più recente si sta spostando dal sistema operativo agli applicativi utente installati sopra.  
  
Applicazioni che girano in modalità utente quali Flash Player, Silverlight, Acrobat Reader, Internet Explorer, Firefox e relativi plugin, sono tutte ottime candidate.... i bug non mancano mai, e se ci riflettiamo un poco perché attaccare tutto il sistema? Dati sensibili, password e quant'altro si trovano proprio dal lato utente!  
  
Naturalmente verso questo tipo di attacchi il famigerato UAC di Windows Vista e Windows 7 spesso non può fare molto... così come può fare ben poco la richiesta di password dei vari Linux, o MacOS: il sistema è protetto, e va bene, ma i dati degli utenti forse non troppo!  
  
Perché quindi bloccare gli aggiornamenti del software applicativo?  
  
Ci devono essere delle ottime ragioni, ed una di queste è che gli aggiornamenti spesso e volentieri sono progettati per funzionare su computer _domestici_ , in cui l'utente ha accesso ad Internet completo ed accede al desktop con amministrativi.  
  
In un'azienda **non tutti i PC hanno accesso ad Internet** , quelli che vi accedono probabilmente **dispongono di un accesso filtrato** , ed anche se riescono a scaricare gli aggiornamenti, propabilmente non riescono ad installarli poiché l'utente **non dispone di diritti per aggiornare il software installato**.  
  
Normalmente il software applicativo presente nei computer aziendali non è lo stesso software che può scaricare l'utente finale: in genere è preferibile distribuire **versioni amministrative o aziendali** oppure versioni **ripacchettizzate** (es. Firefox, Flash Player, e Acrobat Reader si trovano anche in versione MSI, e nel caso in cui non si trovino, basta cercare per esempio sul sito [www.appdeploy.com](<http://www.appdeploy.com/>) per trovare tutto quello di cui c'è bisogno).  
  
E normalmente il software applicativo viene configurato per non aggiornarsi automaticamente: gli aggiornamenti sono disabilitati, perché è inutile disporre di decine di updater uno per ogni servizio, quando invece **sono disponibili applicazioni apposite per la distribuzione di tutti gli aggiornamenti**!  
  
Cosa fare per tutto il software che invece è stato scaricato da una versione "tradizionale" e quindi con gli aggiornamenti attivati?  
  
In genere le informazioni di configurazione sono sparsi in qualche chiave di registro, naturalmente nella HKLM. Abbiamo bisogno quindi di un buon script WSH da eseguire all'avvio della macchina da lanciare con privilegi amministrativi.  
  
Creiamo una nuova policy, e su Computer Configuration - Windows Settings - Scripts aggiungiamo uno script di avvio simile al seguente:  
  

    
    Dim WshShell
    Set WshShell = WScript.CreateObject("WScript.Shell")
    
    On Error Resume Next
    
    ' disabilita aggiornamenti Adobe Acrobat
    
    WshShell.RegWrite "HKLM\Software\Policies\Adobe\Acrobat Reader\8.0\FeatureLockdown\bUpdater", 0, "REG_DWORD"
    WshShell.RegWrite "HKLM\SOFTWARE\Policies\Adobe\Acrobat Reader\9.0\FeatureLockdown\bUpdater", 0, "REG_DWORD"
    WshShell.RegWrite "HKLM\SOFTWARE\Adobe\Acrobat Reader\9.0\AdobeViewer\EULA", 1, "REG_DWORD"
    WshShell.RegWrite "HKLM\SOFTWARE\Adobe\Acrobat Reader\9.0\AdobeViewer\Launched", 1, "REG_DWORD"
    
    ' disabilita aggiornamenti Java
    
    WshShell.RegWrite "HKLM\SOFTWARE\JavaSoft\Java Update\Policy\EnableJavaUpdate",  0, "REG_DWORD"
    WshShell.RegWrite "HKLM\SOFTWARE\JavaSoft\Java Update\Policy\EnableAutoUpdateCheck", 0, "REG_DWORD"
    
    Wscript.Quit
    
  
Questi sono i parametri che ho trovato in giro. Ovviamente possiamo personalizzare questo script a piacimento per supportare altri programmi oppure altre versioni. Naturalmente cerchiamo di farne buon uso!  
  
Llo stesso identico procedimento può essere utilizzato per modificare all'avvio qualsiasi chiave di sistema oppure, se creiamo lo script lato utente, possiamo impostare le chiavi in HKEY_CURRENT_USER.  
  
Molto utile quindi non solo per gli aggiornamenti, ma anche per poter preconfigurare ad ogni avvio una buona parte del software lato desktop.
