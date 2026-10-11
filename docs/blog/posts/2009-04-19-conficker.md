---
date: 2009-04-19
categories:
  - sicurezza
---
# Conficker

Mentre molte aziende sono state colpite in maniera massiccia dall'infezione del worm Conficker, nella mia realtà il problema è passato quasi del tutto inosservato: d'altra parte i requisiti di sicurezza per evitare il propagarsi di un'infezione di questo tipo sono decisamente di base, e dovrebbe far riflettere il fatto che molte aziende che trattano dati personali o sensibili non siano state adeguatamente preparate ad una tale evenienza.
  

In ogni caso, è bene non abbassare la guardia, visto che il Conficker è un worm che non perdona alcuna leggerezza!
Per tutte le informazioni relative al worm vedere questo articolo: "[Considerazioni per un efficace contenimento dell'infezione Conficker.B](<http://blogs.technet.com/feliciano_intini/archive/2009/01/20/considerazioni-per-un-efficace-contenimento-dell-infezione-conficker-b.aspx>)".
  

Uno dei problemi principali consiste nell'individuare quali siano le macchine infette. Fortunatamente sono disponibili numerosi tool di scansione della rete in grado di rilevare eventuali sistemi infetti (vedere [Conficker Remote Scanners](<http://www.dshield.org/diary.html?storyid=5860>)), tra i quali è presente anche [nmap](<http://www.insecure.org/>).
La sintassi per controllare una macchina od una rete remota è la seguente:
    
    nmap -PN -T4 -p139,445 -n -v --script smb-check-vulns,smb-os-discovery --script-args safe=1 [targetnetworks]
Bisogna comunque tenere in considerazione il fatto che molti pc potrebbero essere spenti od essere collegati alla rete solo saltuariamente, e purtroppo avere un inventario esatti di tutti i pc che "mancano all'appello" non è sempre facile. Ho pensato quindi di far girare saltuariamente il seguente script su di un domain controller:  

    
    @echo off  
    for /f "tokens=4 delims=: " %%a in ('netstat -na^|find "ESTABLISHED"') do (  
    nmap -PN -T4 -p139,445 etc. %%a >> scan_%%a  
    )
Questo script estrae tutti gli indirizzi IP che hanno una connessione correntemente aperta con il domain controller, ed effettua una scansione remota sugli indirizzi rilevati.
  

Certo il rischio è quello di effettuare numerose scansioni allo stesso indirizzo IP, ed il rischio è che alcune connessioni possano venire allegramente ignorate (meglio un windump piuttosto di un netstat, se non vogliamo farci sfuggire nulla). Naturalmente se sarà necessario, sono pronto a modificare lo script per renderlo più funzionale!
  

Ah un po' di informazioni sul comando for si trovano a questa pagina <http://www.ss64.com/nt/for.html>.
