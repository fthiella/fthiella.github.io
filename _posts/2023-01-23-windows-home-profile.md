---
layout: post
title: "Windows Home Profile"
date: 2023-01-23 15:09:00 +0200
categories: cmd
---

## Percorso Profilo Utente di Windows

La variabile `%HOMEPROFILE%` contiene il percorso del profilo corrente, tipicamente:

    c:\Users\username

mentre la cartella dei documenti è *di default* la sua sottocartella `Documents` quindi:

    c:\Users\username\Documents

tuttavia il nome della cartella Documents è configurabile e potrebbe essere stato impostato a piacere,
per cui non si può dare per scontato che il percorso `%HOMEPROFILE%\Documents` sia sempre quello corretto.

Il percorso corretto è registrato nella seguente posizione del registro di Windows:

    λ reg query "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v Personal
    
    HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
        Personal    REG_SZ    C:\Users\username\Documents

Alcuni linguaggi di programmazione mettono a disposizione librerie e funzioni apposite per interrogare il registro,
mentre con la shell è necessario utilizzare uno script apposito.

## Script di lettura Percorso

Il seguente script legge il percorso richiesto, e lo memorizza nella variabile `%value%`:

    @echo OFF
    setlocal ENABLEEXTENSIONS
    
    set KEY_NAME="HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders"
    set VALUE_NAME="Personal"
    
    for /F "tokens=2*" %%A IN ('reg query %KEY_NAME% /v %VALUE_NAME%') do (
        set value=%%B
    )
    echo %value%

questo script sembra funzionare sia con le versioni più recenti di Windows che con le precedenti, e supporta percorsi con gli spazi.
