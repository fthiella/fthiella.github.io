---
date: 2009-02-12
categories:
  - windows
  - xcacls
---
# Modifica ACL tramite script

Spesso mi capita di dover mantenere una struttura di cartelle di Windows che hanno delle ACL personalizzate con un livello di dettaglio molto elevato. Quando i permessi cominciano a diventare complessi lo strumento standard di Windows diventa scomodo ed è facile commettere degli errori.  
  
L'utility [Xcacls.exe](<http://www.microsoft.com/windows2000/techinfo/reskit/tools/existing/xcacls-o.asp>), che si scarica dal sito della Microsoft, può essere di grande aiuto in tutti questi casi. Ho preparato uno script batch che richiama questa utility, leggendo da un file di testo con l'elenco delle cartelle e delle relative ACL da impostare ad ognuna di esse:  
  
_esempio di elencoacl.txt:_  

    
    "c:\sede\ced";"DOMAIN\user1:C" "DOMAIN\user2:R"
    "c:\sede\personale";"DOMAIN\user3:C" "DOMAIN\user4:R"
    "c:\sede\personale\dirigenza";"DOMAIN\user3:C"
    "c:\sede\personale\pub";"DOMAIN\Everyone:R"
    "c:\sede\tecnico";"DOMAIN\user4:C" "DOMAIN\user5:C"
    "c:\sede\tecnico\progetti";"DOMAIN\user4:R"
    "c:\sede\tecnico\lavori";"DOMAIN\user5:R"
    
etc...   
con il seguente _impostaacl.cmd:_  

    
    set LETTURA="DOMAIN\leggi:R"
    set SCRITTURA="DOMAIN\fede:C"
    set FULL="BUILTIN\Administrators:F" "NT AUTHORITY\SYSTEM:F"
    
    for /F "tokens=1,2 delims=;" %%a in (elencoacl.txt) do (
      xcacls %%a /T /G %FULL% %LETTURA% %SCRITTURA% %%b /Y
      @if ERRORLEVEL 1 (
        echo Errore set ACL directory %%a
        pause
      )
    )
    
Questo script batch legge in sequenza tutte le righe del file _elencoacl.txt_ , ed imposta ricorsivamente ad ogni cartella i permessi associati. Fare attenzione alle cartelle innestate: questo script non è molto "furbo" e ripete le impostazioni più volte. Verificare che l'ordine delle cartelle sia corretto!
