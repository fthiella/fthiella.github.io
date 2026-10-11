---
date: 2009-07-02
categories:
  - python
---
# Inviare Check Passivi a Nagios

Nagios è un programma di monitoraggio di computer o risorse di rete che permette di inviare degli avvisi quando un nodo od un servizio non risulta attivo, oppure al ripristino del suo funzionamento.  
  
Per il monitoraggio degli apparati è possibile utilizzare uno dei numerosi comandi predefiniti (es. ping, snmp, etc) oppure è possibile scrivere un comando personalizzato - uno script bash, perl, python o altro va benissimo, l'importante è che tale script restituisca un valore che indica se il servizio che stiamo monitorando è attivo oppure se presenta qualche problema.  
  
Il sistema si occupa in automatico di schedulare i test, di raccogliere i risultati, e di inviare eventuali avvisi.  
  
Oltre a questa modalità di monitoraggio in cui il Nagios effettua dei test ed attende la risposta dagli host remoti, che è detta **attiva** , è possibile utilizzare una seconda modalità, detta **passiva** , in cui sono gli host o i servizi remoti che inviano il loro stato al Nagios.  
  
L'utilizzo più tipico consiste nel monitoraggio delle operazioni pianificate, ad esempio backup, allineamento tabelle, etc.  
Un task viene eseguito, svolge il backup, l'allineamento, o quant'altro, ed al termine invia al Nagios una notifica con l'esito dell'operazione. Se il Nagios non riceve tale esito entro un certo periodo di tempo (perché il task si blocca prima di inviare l'avviso, o perché il task dura troppo a lungo) può segnalare comunque di non aver ricevuto alcuna informazione e attivare un allarme.  
  
Il servizio può essere definito in questo modo (i parametri possono essere personalizzati a piacere):  

    
    define service{
      use                    service-template
      host_name              myhost
      service_description    logs
      freshness_threshold    93600
      notification_period    24x7
      check_command          check_dummy!3 "Dati non ricevuti"
      active_checks_enabled  0
      passive_checks_enabled 1
      notification_interval  0
      check_freshness        1
      check_period           24x7
      max_check_attempts     1
      contact_groups         contatti
      notification_options   c,u,w,r
      notification_interval  0
      notification_period    24x7
    }
    
  
In cui il comando check_dummy è definito nel seguente modo:  

    
    define command {
      command_name    check_dummy
      command_line    $USER1$/check_dummy $ARG1$
    }
    
in questo esempio, Nagios attende il ricevimento di una notifica passiva, e se non riceve una notifica da più di 93600 secondi, allora esegue il comando _check_dummy_ , che imposta lo stato del servizio ad _UNKNOWN_ ed inserisce l'avviso che non riceve i log. Un amministratore di sistema si dovrà dunque preoccupare di approfondire il problema.  
  
Come inviare i dati al server Nagios centrale? Esistono molti sistemi, ma il più semplice ed il mio preferito consiste nell'inviare l'esito dell'operazione direttamente al **cmd.cgi** che lo mette in coda agli altri eventi e lo fa processare.  
  
Basta inviare un semplice POST HTTP al cgi che si trova in cgi-bin/cmd.cgi e siamo a posto! Come al solito il codice seguente è decisamente semplificato, e non tiene conto di ogni possibile errore o circostanza. In alternativa, è sempre possibile lanciare un buon wget --post="parametri" http://nagios/nagios/cgi-bin/cmd.cgi.  

    
    import httplib
    import urllib
    import sys
    
    argc = len(sys.argv)
    
    if (argc < 6):
      print "Nagios Passive Check Submit"
      print "Esempio:"
      print "nagiospcs Server \"Trasferimento Dati\" 3 \"Errore Import\" \"\""
    else:
      data = urllib.urlencode(
        {"cmd_typ" : "30",
         "cmd_mod" : "2",
         "host" : sys.argv[1],
         "service" : sys.argv[2],
         "plugin_state" : sys.argv[3],
         "plugin_output" : sys.argv[4],
         "performance_data" : sys.argv[5]})
      f = urllib.urlopen(
        "http://user:password@servernagios/nagios/cgi-bin/cmd.cgi",
        data)
      s = f.read()
      f.close()

Ave.
