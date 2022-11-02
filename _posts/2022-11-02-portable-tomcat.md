---
layout: post
title: "Portable Tomcat"
date: 2022-11-02 22:40:00 +0200
categories: tomcat, java
---
In questi giorni avevo necessità di testare nella mia macchina di sviluppo
delle servlet utilizzando la versione portable di Apache Tomcat.

Di seguito la procedura che ho utilizzato.

## Predisposizione Ambiente

Ho scelto di installare tutti i software portable sotto la directory `sw` della cartella dei documenti:

    c:\Users\[username]\Documents\sw\

Ho quindi scaricato la versione 64-bit per Windows in formato zippato dalla sezione [Download di Tomcat 10](https://tomcat.apache.org/download-10.cgi)
ed ho estratto il contenuto in:

    c:\Users\[username]\Documents\sw\apache-tomcat-10\

Il sistema richiede che sia definita la variabile di ambiente `JRE_HOME`, nel caso
in cui non sia già configurata nel sistema è consigliabile predisporre il file
`setenv.bat` all'interno della cartella `bin` come segue:

    @echo off
    set "JRE_HOME=c:\Program Files\Java\jdk-17.0.2\"
    exit /b 0

## Attivazione del server

Dato che si tratta di un ambiente di test, non ho necessità che il sistema sia
sempre in esecuzione ma lo voglio lanciare quando necessario.

I comandi da lanciare sono pertanto, in sequenza, i seguenti:

    setenv.bat
    catalina.bat run

Mentre per terminare è sufficiente un normale `CTRL+C`.

## Configurazione CmdEr

Dato che utilizzo il sistema CmdEr ho pensato di predisporre il seguente script che configura l'ambiente e lancia il server.
L'ho chiamato `catalina.cmd` all'interno del seguente percorso
`C:\Users\[username]\Documents\sw\cmder\bin` in modo che sia disponibile direttamente dalla console:

    @echo on
    pushd "c:\Users\[username]\Documents\sw\apache-tomcat-10.0.27\bin\"
    call setenv.bat
    call catalina.bat %*
    popd

## Configurazione utente Tomcat

Sotto la directory `conf` di Tomcat si trova il file di configurazione `tomcat-users.xml` al quale ho aggiunto, nella sezione
users, la seguente definizione:

<user username="fthiella" password="secret" roles="standars,manager-script,manager-gui" />

## Accesso al sistema

Il sistema è accessibile quindi via browser al seguente indirizzo [http://localhost:8080/](http://localhost:8080/).

L'installazione in ambiente di produzione richiede naturalmente una configurazione più attenta e curata (gestione privilegi utente,
permessi file e directory, gestione servizi, gestione e rotazione log, ...) ma non è oggetto di questo articolo.
