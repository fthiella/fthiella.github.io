---
layout: post
title: "Advanced Cron"
date: 2020-12-16 22:14:00 +0200
categories: bash, sh, linux
---
Per gestire l'emergenza Covid-19 è necessario elaborare e trasmettere diversi flussi informativi a diverse piattaforme
(piattaforme aziendali, bucket regionali, in formato csv o excel, applicativi di gestione casi, dashboard di aggregazione
o analisi dei dati, etc.).

Ho quindi predisposto diversi task che:
- richiamano dei piccoli applicativi Perl per estrarre via SQL le informazioni necessarie, le trasformano, e le salvano in file in formato CSV
- comprimono i dati, eventualmente con password complesse
- trasmettono i dati verso bucket aws, su altri server Qlik o Pentaho
- generano mail puntuali o di riepilogo

Il testo che segue non è un "manuale" su come fare le cose, ma un elenco di appunti che mi aiutano a ricordare quali operazioni
ho fatto e qual è la logica sottostante.

La schedulazione avviene con il servizio `crond`. Tuttavia nel corso del tempo il numero di task è diventato numeroso,
la configurazione della `crontab` è diventa complessa, e modificare o monitorare il corretto funzionamento dei task è diventato
poco agevole.

Ho quindi voluto riorganizzare il servizio con il seguente sistema. La tabella con i task è minimale, questo un task che necessita di essere eseguito ogni ora:

````crontab
0  *  *  *  * /home/fede/scripts/hourly_tasks.sh
````

Questo task a sua volta si occupa di lanciare tutti gli script che devono essere eseguiti ogni ora, gestendone correttamente l'output stdout e stderr nei file di log opportuni:

````sh
#!/bin/bash

LOG_FILE=/var/log/covid_out.log
LOG_ERROR=/var/log/covid_err.log

exec 1>>"$LOG_FILE" 2>>"$LOG_ERROR"

./task01.sh &
./task02.sh &
./task03.sh &
...
````

In questo caso i vari task vengono eseguiti in parallelo, ed ereditano gli output verso i file di log.

Tuttavia per ogni task voglio fare in modo che il log contenga la data e l'ora, ed il nome del task eseguito:

````sh
#!/bin/bash
PROGNAME=$(basename $0)

exec  > >(sed --unbuffered "s/^/$(date +'%Y-%m-%d %H:%M:%S'),$PROGNAME: /")
exec 2> >(sed --unbuffered "s/^/$(date +'%Y-%m-%d %H:%M:%S'),$PROGNAME: /" >&2)

...
````

Il parametro `--unbuffered` assicura che la singola riga viene processata immediatamente, senza utilizzare
un buffer, in modo che data e l'ora presentate nel file di output siano quelle effettive di
quando l'output è stato realizzato e che i vari output siano sequenziali.

Inoltre per gestire correttamente eventuali errori, ho predisposto una funzione come segue:

````sh
error_exit()
{
        echo "${1:-"Unknown Error"}" 1>&2
        exit 1
}
````

La struttura completa del task risulta essere questa:

````sh
#!/bin/bash
PROGNAME=$(basename $0)

error_exit()
{
        echo "${1:-"Unknown Error"}" 1>&2
        exit 1
}

exec  > >(sed --unbuffered "s/^/$(date +'%Y-%m-%d %H:%M:%S'),$PROGNAME: /")
exec 2> >(sed --unbuffered "s/^/$(date +'%Y-%m-%d %H:%M:%S'),$PROGNAME: /" >&2)

echo "Inizio elaborazione"

perl "extract.pl" -prepare || error_exit "$LINENO: Errore preparazione dati."
perl "extract.pl" -extract > $output || error_exit "$LINENO: Errore estrazione dati."

...
````

Come scaricare un file più recente da aws via cli:

````sh
latest=`$AWS s3 ls s3://bucket/path/documento | sort | tail -1 | awk '{ print $4 }'`                   
$AWS s3 cp --no-progress s3://bucket/path/$latest /home/fede/csv/documento.csv
````

Per scaricare il file più recente da un altro unix, in questo caso utilizzando sshpass (da valutare ovviamente se è la soluzione opportuna):

````sh
latest=$(sshpass -p $pass ssh fede@server 'ls -t /var/export/documento* | head -1')
sshpass -p $pass scp fede@server:/$latest $output_dir/$output_file
````
