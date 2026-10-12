---
date: 2009-06-29
categories:
  - windows
title: Eliminare File Temporanei
---
Spesso applicazioni ed installer lasciano in giro per l'hard disk un sacco di spazzatura, dimenticandosi allegramente di cancellarla. Anche se è vero che i dischi sono sempre più capienti, e che un po' di file in eccesso non fanno poi così male come molti credono, ogni tanto vale la pena mettere un po' di ordine e di fare un poco di pulizia.  
  
Posizioni dei file temporanei  
Dove vengono creati generalmente i file temporanei? In genere vengono salvati nelle seguenti posizioni:  

  * %WINDIR%\Temp (tipicamente C:\WINDOWS\Temp)
  * %TEMP% (tipicamente C:\Users\nomeutente\AppData\Local\TEMP, in Windows Vista)

  
(i file temporanei di Internet Explorer o di altri browser preferisco non toccarli: in questo articolo voglio solo eliminare soltanto i file che si trovano nelle cartelle temporanee utilizzate dagli altri programmi).  
  
  
Per spazzare via un po' di file inutili dovrebbero quindi essere sufficienti i seguenti comandi:  

    
    DEL /F /S /Q "%WINDIR%\Temp"
    DEL /F /S /Q "%TEMP%"
    
  
Giusto? Al di là del fatto che questi comandi **sono particolarmente delicati** (attenzione a non sbagliare il percorso!) è importante notare che questi comandi **non sono sufficienti in un ambiente** _**multiuser**!_  
  
Si noti infatti che %TEMP% contiene soltanto il percorso dei file temporanei che appartengono all'utente corrente, ma ogni utente di sistema dispone della sua personale cartella temporanea!  
  
Per svolgere bene il nostro compito, dobbiamo quindi scorrere tutte le variabili di ambiente di tutti gli utenti presenti nel sistema!  
  
  
Come conoscere tutti gli utenti?  
L'elenco degli identificativi degli utenti (detto SID) che dispongono di un profilo nel sistema lo si ottiene interrogando la chiave HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\ProfileList:  
  

    
    REG QUERY "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\ProfileList
    
  
  
ed all'interno di ogni SID possiamo ottenere il percorso su disco del profilo interrogando la chiave _ProfileImagePath_.  
  
Dove sono memorizzate le variabili d'ambiente?  
Le variabili per l'utente corrente si trovano in _HKEY_CURRENT_USER\Environment_ , mentre le vabili di tutti gli utenti attualmente attivi sul sistema le possiamo recuperare in _HKEY_USERS\SID\Environment._  
  
Se vogliamo fare le cose fatte bene dobbiamo scorrere le chiavi anche per tutti gli utenti che non sono attualmente attivi: in questo caso Windows non carica il profilo in memoria nella HKEY_USERS ma tutte le chiavi si trovano su disco nel file NTUSER.DAT che si trova all'interno di ogni _ProfileImagePath_.  
  
Lo script  
Nello script seguente ho mescolato un po' di tecniche diverse: **WMI** per scorrere il registro, **WScript.Shell** per leggere il registro in modalità non espansa (le variabili d'ambiente che contentono il percorso %USERPROFILE% escono tutte espanse con il profilo dell'utente che lancia lo script, e non va bene! Ovviamente non mi piace questo metodo, e spero di trovare una soluzione migliore), ed infine per caricare il file NTUSER.DAT in una chiave temporanea, non ho trovato nulla di meglio di lanciare dalla shell un buon **REG LOAD**...arh, brutto ma funzionale!  
Fortuna che dopo qualche errore di codifica mi rifaccio, e per memorizzare l'elenco delle cartelle ho utilizzato un ArrayList, così come per le stringhe le **espressioni regolari** vanno sempre più che bene!  
  

    
    On Error Resume Next
    
    Const HKEY_LOCAL_MACHINE = &H80000002
    Const HKEY_USERS         = &H80000003
    
    strComputer = "."
    
    Set objRegistry = GetObject("winmgmts:\\" & strComputer & "\root\default:StdRegProv")
    Set ws = CreateObject("Wscript.Shell")
    Set temporaryFolders = CreateObject("System.Collections.ArrayList")
    Set re = new RegExp
    
    Function GetTemp(ProfilePath, K, Key)
    On Error Resume Next
    Temp = ws.RegRead(K & "\" & Key & "\Environment\TEMP")
    
    If err.number = 0 Then
    re.Pattern = "%USERPROFILE%"
    Temp = re.Replace(Temp, ProfilePath)
    Else
    Temp = Null
    End If
    
    GetTemp = Temp
    On Error Goto 0
    End Function
    
    strKeyPath = "SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList"
    objRegistry.EnumKey HKEY_LOCAL_MACHINE, strKeyPath, arrSubkeys
    
    For Each objSubkey In arrSubkeys
    rem *** legge il percorso del profilo ***
    strValueName = "ProfileImagePath"
    strSubPath = strKeyPath & "\" & objSubkey
    objRegistry.GetExpandedStringValue HKEY_LOCAL_MACHINE, strSubPath, strValueName, strProfile
    
    rem *** legge il percorso della cartella TEMP ***
    FolderTmp = GetTemp(strProfile, "HKEY_USERS", objSubkey)
    
    if isNull(FolderTmp) Then
    rem *** non ha letto la chiave, quindi prova a caricare il registro da NTUSER.DAT ***
    ws.Run """%WINDIR%\System32\REG.EXE"" LOAD HKLM\TempHive """ & strProfile & "\NTUSER.DAT""", 7, true
    msgbox "Leggo " & strProfile
    FolderTmp = GetTemp(strProfile, "HKEY_LOCAL_MACHINE", "TempHive")
    msgbox "Letto " & FolderTmp
    ws.Run """%WINDIR%\System32\REG.EXE"" UNLOAD HKLM\TempHive", 7, true
    End If
    temporaryFolders.Add FolderTmp
    Next
    
    For Each folder In temporaryFolders
    Wscript.Echo "Percorso temporaneo da svuotare: " & folder
    Next
    
  
  
Come al solito il codice è brutto sporco e cattivo, inoltre in caso di errore.... be meglio verificare per bene prima di eliminare, non si sa mai! Ovviamente posso estendere questo codice, ed aggiungere i percorsi temporanei di Internet Explorer, Mozilla Firefox, oppure qualsiasi altro, basta solo conoscere la posizione di registro che indica il percorso dove sono salvati, oppure basta solo conoscere l'eventuale percorso del file di configurazione presente nel profilo.  
  
Ah forse lo stesso risultato lo si poteva ottenere solamente con un file batch, qualche FOR e molte chiamate all'utility REG.EXE: sarebbe stato più coerente, ma sicuramente molto più illeggibile!  
  
Attenzione  
Nel caso in cui vengano utilizzati dei profili roaming, è sempre un rischio andare a toccare i profili salvati in locale, poiché si rischia di far perdere al sistema le informazioni di quale sia l'ultimo profilo utilizzato.
