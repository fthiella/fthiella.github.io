---
date: 2010-10-27
categories:
  - windows
  - python
---
# Process Killer

Quando un programma va in crash la colpa viene attribuita sempre al programmatore. Mentre quando un programma si blocca, o è troppo lento, è di certo per qualche misfatto dell'amministratore di sistema. Anche se magari il programma è scritto con i piedi ed è propenso all'inchiappettamento di tutti i server su cui gira.  
  
Per ristabilire un po' di giustizia divina, ho pensato di killare senza pietà tutti i processi che utilizzano ferocemente la CPU per più di un determinato tempo. Metodo barbaro, ma molto in voga negli Unix dei vecchi tempi... e non solo.  
  
Naturalmente nel mio caso si tratta di processi che dovrebbero essere molto leggeri ma per qualche strana ragione si incastrano e finiscono per utilizzare il processore a pieno regime, infastidendo anche tutti gli utenti di programmi buoni e gentili.  
  
Come punto di partenza, direi di dare un'occhiata all'output del comando _pslist_ , che lista tutti i processi in esecuzione su di una macchina (locale o remota, a patto di avere i privilegi necessari):  
  

    
    pslist v1.29 - Sysinternals PsList
    Copyright (C) 2000-2009 Mark Russinovich
    Sysinternals
    
    Process information for ER-MEJO:
    
    Name                Pid Pri Thd  Hnd   Priv        CPU Time    Elapsed Time 
    Idle                  0   0   2    0      0    23:05:50.623     0:00:00.000
    System                4   8 114 4338     52     0:11:32.114    28:42:03.487
    smss                252  11   3   30    340     0:00:00.171    28:42:03.487
    csrss               348  13   9 1033   2744     0:00:06.645    28:41:51.132
    wininit             416  13   3   79    992     0:00:00.093    28:41:50.414
    explorer           4448   8  36 1252  69768     0:07:39.797    28:41:04.882
    iexplore           3772   8  13  382   7172     0:00:04.539     3:14:05.766
    iexplore           5788   8  22  704  97632     0:05:51.345     3:14:02.648
    PsList             4136  13   1  151   1980     0:00:00.187     0:00:00.172
    
  
le cui colonne interessanti sono le prime due (nome del process e PID) e le ultime due:  
  
**Elapsed Time:** tempo totale di esecuzione del processo  
**CPU Time:** tempo effettivamente utilizzato dal processo  
  
ad esempio _explorer_ è in esecuzione da 28 ore e 41 minuti, e durante questo periodo ha utilizzato la cpu per 7 minuti e 39 secondi. Proviamo a rilanciare il _psexec_ e a vedere che cosa succede:  
  

    
    Name                Pid Pri Thd  Hnd   Priv        CPU Time    Elapsed Time 
    Idle                  0   0   2    0      0    23:06:42.540     0:00:00.000
    System                4   8 114 4342     52     0:11:32.426    28:42:33.493
    smss                252  11   3   30    340     0:00:00.171    28:42:33.493
    csrss               348  13   9 1050   2744     0:00:06.645    28:42:21.138
    wininit             416  13   3   79    992     0:00:00.093    28:42:20.420
    explorer           4448   8  42 1295  71484     0:07:40.234    28:41:34.888
    iexplore           3772   8  13  389   7600     0:00:04.617     3:14:35.772
    iexplore           5788   8  23  716  95520     0:05:53.513     3:14:32.654
    PsList             1792  13   1  151   1988     0:00:00.171     0:00:00.156
    
  
il comando è stato lanciato circa 30 secondi dopo il primo, ed infatti tutto gli _Elapsed Time_ sono aumentati circa di 30 secondi, processo _PsList_ escluso ma se osserviamo bene si tratta di un nuovo processo con un nuovo PID.  
  
Anche la colonna _CPU Time_ è variata per alcuni processi, e se dividiamo il delta del _CPU Time_ per il delta dell'_Elapsed Time_ possiamo ottenere... **la percentuale di utilizzo del processore** di questo processo relativamente la periodo di tempo intercorso tra i due lanci del comando _psexec_.  
  
Da notare che se la macchina dispone di più processori nel _Task Manager_ di Windows vedremo la percentuale riferita a tutte le CPU: un processo che gira su di un singolo processore non potrà quindi superare il 25% di CPU se la macchina dispone di 4 processori.  
  
Con il comando psexec invece non ci dobbiamo preoccupare del numero di processori che ha la macchina: il tempo di CPU utilizzato diviso il tempo di CPU avuto è disposizione può variare da zero a uno ed è l'utilizzo effettivo del signolo processore.  
  
Ho scritto questo codice in Python che controlla ad intervalli regolari un elenco di server, e che giustizia tutti i processi che durante questo periodo di tempo hanno consumato più del 90% della propria CPU:  
  

    
    from __future__ import division
    import subprocess
    import re
    import time
    
    word = dict()
    pausa = 30
    servers = ('citrix01', 'citrix02', 'citrix03', 'server', 'server-2')
    
    def milliSeconds(time):
      tm = re.match("(\d*)\:(\d*)\:(\d*)\.(\d*)", time)
      if tm:
        return long(tm.group(1)) * 60*60*1000 + long(tm.group(2)) * 60*1000 + long(tm.group(3)) * 1000 + long(tm.group(4))
      else:
        return 0
    
    def getProcess(server, nome):
      pslist = subprocess.Popen('"c:\program files\utils\pslist.exe" \\\\' + server + ' ' + nome, shell=False, stdout=subprocess.PIPE)
    
      for line in pslist.stdout:
        proc = re.match("^(\w+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+([\w\:\.]+)\s+([\w\:\.]+)", line)
        if proc:
          serverPid = server + '.' + proc.group(2)
          if serverPid in word:
            if (milliSeconds(proc.group(7))-milliSeconds(word[serverPid][1]))/(milliSeconds(proc.group(8))-milliSeconds(word[serverPid][2]))>0.90:
              subprocess.Popen('"c:\program files\utils\pskill.exe" \\\\' + server + ' ' + word[serverPid][4])
          else:
            if proc.group(1) <> "Idle" and proc.group(1) <> "System":
              word[serverPid] = [proc.group(1), proc.group(7), proc.group(8), server, proc.group(2)]
    
    def main():
      while True:
        for server in servers:
          getProcess(server, 'winword')
          time.sleep(pausa)
    
    if __name__ == "__main__":
        main()
    
  
da sistemare per bene... e da usare con opportuna cautela!
