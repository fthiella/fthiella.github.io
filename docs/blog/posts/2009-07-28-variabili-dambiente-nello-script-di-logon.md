---
title: Variabili d'ambiente nello script di Logon
date: 2009-07-28
categories: [windows, vbs, sysadmin, scripting]
---

Come settare le variabili di ambiente in uno script di logon?  
In VBScript si può fare così:  
  

    
    Set WshShell = WScript.CreateObject("WScript.Shell")
    Set WshUEnv = WshShell.Environment("User")
    
    WshUEnv("INIAPP") = "%USERPROFILE%\APP.INI"
    
  
(in questo esempio ho settato alla variabile INIAPP, relativa all'ambiente dell'utente, il valore "%USERPROFILE%\APP.INI"). Per rendere la modifica permanente, aggiungo anche la seguente riga:  

    
    WshShell.RegWrite "HKCU\Environment\INIAPP", "%USERPROFILE%\APP.INI", "REG_EXPAND_SZ"
    
  
Capita spesso inoltre di dover modificare il PATH di ricerca. La soluzione che utilizzo è la seguente, e mi permette di evitare di avere PATH doppi:  
  

    
    Set WshShell = WScript.CreateObject("WScript.Shell")
    Set WshUEnv = WshShell.Environment("User")
    Set pp = CreateObject("Scripting.Dictionary")
    
    for each p in Split(WshUenv("PATH"), ";")
      AddPath p
    next
    
    AddPath "C:\PROGRAMMA1"
    AddPath "C:\ORACLE"
    AddPath "C:\TEST"
    
    WshUEnv("PATH") = Join(pp.Keys, ";")
    WshShell.RegWrite "HKCU\Environment\PATH", Join(pp.Keys, ";"), "REG_EXPAND_SZ"
    
    pp.RemoveAll
    Set pp = Nothing
    Set WshShell = Nothing
    Set WshUEnv = Nothing
    
    Sub AddPath(path)
      If Not pp.Exists(path) And Trim(path) <> "" Then
        pp.Add path, ""
      End If
    End Sub
  
Suddivido il percorso originario in un array con i percorsi singoli, e con l'oggetto Scripting.Dictionary aggiungo ogni singolo percorso, senza duplicati. Poi ricompongo la stringa.  
Semplice ma elegante!
