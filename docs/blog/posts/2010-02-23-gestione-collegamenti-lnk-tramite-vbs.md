---
date: 2010-02-23
---
# Gestione Collegamenti LNK tramite VBS

È un dato di fatto: i desktop dei PC aziendali sono una schifezza. Sono pieni di collegamenti, magari vecchi ed obsoleti o che puntano al percorso errato.  
  
A volte sono pieni di collegamenti scritti con le maiuscole, o con troppi spazi, o con troppo pochi spazi, oppure capitalizzati senza un criterio logico e comprensibile.  
  
Spesso ancora i collegamenti vengono creati sul desktop (e quindi nel profilo) di **ogni singolo utente** , mentre a volte se ne starebbero più felici e contenti nel profilo **All Users** del computer.  
  
Come rimediare ad una situazione del genere, e dare un tocco di pulizia e professionalità ai ~~catorci~~ PC della nostra azienda?  
Con un poco di codice VBS, naturalmente!  
  
Casi di utilizzo  
Si potrebbero voler eliminare tutti i collegamenti non più necessari, quali ad esempio _\\\ServerAziendale\Documenti\Incentivi\_.  
  
Oppure si potrebbero voler aggiornare dei collegamenti con un nuovo percorso (ad es. i collegamenti ad [http://it.wikipedia.org](<http://it.wikipedia.org/>) potrebbero essere aggiornati alla nuova versione dell'enciclopedia all'indirizzo [http://nonciclopedia.wikia.com](<http://nonciclopedia.wikia.com/>)).  
  
In questo esempio farò molto di più!  
  
Vado alla ricerca del collegamento ad un determinato programma (\\\ServerAziendale\Programma\Start.exe) sul desktop di ogni singolo utente. Se almeno un utente dispone di questo link, li cancello tutti quanti, e ne ricreo uno solo (con il nome bello, pulito, e capitalizzato come si deve) nel desktop di All Users.  
  
The Code  

    
    Set Start_Exe = CreateObject("System.Collections.ArrayList")
    
  
_La variabile Start_Exe conterrà tutti gli eventuali link al nostro programma._  

    
    Rem *** Ricerca in tutti i profili ***
    
    Const HKEY_LOCAL_MACHINE = &H80000002
    Const HKEY_USERS         = &H80000003
    
    strComputer = "."
    
    Set objRegistry = GetObject("winmgmts:\\" & strComputer & "\root\default:StdRegProv")
    Set ws = CreateObject("Wscript.Shell")
    Set oFs = CreateObject("Scripting.FileSystemObject")
    
    strKeyPath = "SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList"
    objRegistry.EnumKey HKEY_LOCAL_MACHINE, strKeyPath, arrSubkeys
    
    For Each objSubkey In arrSubkeys
      strValueName = "ProfileImagePath"
      strSubPath = strKeyPath & "\" & objSubkey
      objRegistry.GetExpandedStringValue HKEY_LOCAL_MACHINE, strSubPath, strValueName, strProfile
      ScanDesktop(strProfile & "\Desktop") ' dispongo del percorso del desktop!
    Next
    
    strAllUsersDesktopPath = ws.SpecialFolders.Item("AllUsersDesktop")
    
    rem *** se si vogliono ricreare anche i collegamenti in AllUsers già esistenti,
    rem *** decommentare le seguenti righe
    'ScanDesktop(strAllUsersDesktopPath)
    
_Il codice precedente legge l'elenco dei percorsi dei profili, a partire dalla chiave HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList. Che fare se il desktop è stato redirezionato da qualche parte? Ci pensiamo un'altra volta, please!_  
_  
_  
_Ed ecco il codice che ricrea i collegamenti:_  

    
    Rem ***verifica se ci sono collegamenti al programma
    If Start_Exe.Count > 0 Then
      Set objShtCut = ws.CreateShortcut(strAllUsersDesktopPath & "\Programma Aziendale.lnk")
      objShtCut.TargetPath = "\\ServerAziendale.dominio.dc\Programma\Start.exe"
      objShtCut.WorkingDirectory = "\\ServerAziendale.dominio.dc\Programma\"
      objShtCut.Save
      Set objShtCut = Nothing
    
      For Each Lnk in Start_Exe
        Set fl = oFs.GetFile(Lnk)
        fl.Delete
        Set fl = Nothing
      Next
    End If
    
  
_Manca niente? Manca il codice che scansiona ogni singolo Despol:_  
  

    
    Sub ScanDesktop(profile)
      If oFs.FolderExists(profile) Then
        Set oFolder = oFs.GetFolder(profile)
    
        For Each oFile In oFolder.Files
        name = oFile.Name
        if len(name)>4 and UCase(right(name,4)) = ".LNK" then
          Set objShtCut = ws.CreateShortcut(profile & "\" & oFile.Name)
          Select Case UCase(objShtCut.TargetPath)
          Case "\\SERVERAZIENDALE\PROGRAMMA\START.EXE"
            Start_Exe.Add profile & "\" & oFile.Name
          Case "\\SERVERAZIENDALE.DOMINIO.DC\PROGRAMMA\START.EXE
            Start_Exe.Add profile & "\" & oFile.Name
          End Select
          Set objShtCut = Nothing
        End If
    End Sub
    
Naturalmente nella nostra sub ScanDesktop possiamo identificare i collegamenti tramite il metodo che preferiamo.  
E naturalmente, come al solito, sto allegramente ignorando molte condizioni d'errore. Ma la vita è troppo corta per la gestione adeguata degli errori. E in VBS non vi è traccia di Try Catch Finally... [oppure sì](<http://my.opera.com/Lee_Harvey/blog/2007/04/21/try-catch-finally-in-vbscript-sure>)?
