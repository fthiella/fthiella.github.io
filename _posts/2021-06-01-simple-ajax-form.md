---
layout: post
title: "Simple Ajax Form"
date: 2021-01-01 13:51:00 +0200
categories: php, ajax, html
---
## Vaccinazioni senza prenotazione

La campagna vaccinazioni in Italia è partita in ritardo ed in modo disorganizzato.
Nei mesi di Marzo ed Aprile 2021 diverse sedute sono state aperte al pubblico
senza appuntamento, e centinaia di anziani si sono trovati in fila fuori dai centri
vaccinali, tutti nello stesso momento, e sono stati costretti ad attendere diverso
tempo prima di poter fare l'accettazione ed ottenere il loro turno.

Per poter accedere alla vaccinazione è necessario compilare il "Modulo di Consenso Informato"
ed una scheda con l'anamnesi. Dato che le sedute erano senza appuntamento, la maggior
parte delle persone in fila non disponevano dei moduli che dovevano quindi essere
distribuiti e compilati a mano.

Abbiamo pensato che la predisposizione automatica di un modulo stampato con i dati
del paziente avrebbe potuto mettere un po' di ordine ed aiutato a velocizzare la fase
di accettazione.

## Modulo automatico

L'anagrafe dei Pazienti è a disposizione in diversi formati, anche su Postgresql e le
postazioni di accettazione dispongono tutte di collegamento VPN verso la rete aziendale.

L'idea iniziale era quindi di leggere tramite penna ottica il codice a barre della tessera
sanitaria, e tramite la predisposizione di una stampa-unione Word si sarebbe potuto
predisporre e stampare il documento corretto.

![Modulo Consenso]({{ site.baseurl }}/images/consenso.png)

Tuttavia questa soluzione richiede l'installazione del pacchetto Microsoft Word
oppure Microsoft Office, e l'installazione e configurazione dei driver ODBC per Postgresql.

Ho pensato invece di utilizzare una macro Word e di esporre il codice tramite Web API

## Codice Word

Per visualizzare gli strumenti per sviluppatori è necessario abilitare il relativo menù:

- Nella scheda *File* passare a *Opzioni*> *Personalizzazione barra multifunzione*.
- In Personalizzazione della barra multifunzione e Schede principali selezionare la casella di controllo *Sviluppo*.

A questo punto è disponibile il menù attraverso il quale sarà possono aggiungere dei controlli
direttamente nel testo del documento:

![Menù sviluppatore]({{ site.baseurl }}/images/consenso_sviluppatore.PNG)

e tramite il pulsante proprietà andiamo ad assegnare un nome ad ogni controllo:

![Proprietà controllo]({{ site.baseurl }}/images/consenso_word.PNG)

tramite codice VBA andremo a leggere i dati dalla WEB API ed a caricare il risultato
in ogni singolo controllo.

{% highlight vba %}
Sub GetPatientData(cf As String)
    Dim hReq As Object
    Dim response As String
    Dim Result
    
    On Error Resume Next

    Dim strUrl As String
    strUrl = "http://webserver/anagrafe/api?cf=" & cf

    Set hReq = CreateObject("MSXML2.XMLHTTP")
    With hReq
        .Open "GET", strUrl, False
        .Send
    End With

    response = hReq.ResponseText
    Result = Split(response, "::")

    Dim cc As ContentControl
    For Each cc In ActiveDocument.ContentControls
        If cc.Title = "codice_fiscale" Then
            cc.Range.Text = Result(0)
        End If
        If cc.Title = "nome" Then
            cc.Range.Text = Result(1)
        End If
    Next cc

    On Error Goto 0
End Sub


Sub LeggiDatiPaziente()
    Dim cf As String

    cf = InputBox("Inserire il codice fiscale", "Codice fiscale")
    
    LoadPatientData cf
End Sub
{% endhighlight %}

L'idea iniziale era di esporre i dati via JSON ma in VBA non mi sembrava così immediato
senza dover ricorrere a delle librerie esterne ed ho quindi ritenuto di esporre i dati
separati da un "banale" separatore ::

Lato server avevo a disposizione, già configurato e pronto all'uso, un Apache con PHP ed
ho quindi predisposto un semplice script che assomiglia a quanto segue:

{% highlight php %}
<?php
  header('Content-Type: text/plain; charset=UTF-8');

  $cf = $_GET['cf'];

  $conn = new PDO("pgsql:host=x.y.z.k;dbname=anagrafe", '...', '...');
  $stmt = $conn->prepare("select ... where codice_fiscale=:cf");
  $stmt->execute([ 'cf' => $cf ]);
  $patient = $stmt->fetch(PDO::FETCH_ASSOC);

  echo ...
?>
{% endhighlight %}

non sono necessari particolari controlli, l'utilizzo di una prepared statement
è sufficiente e se per caso qualche cosa non va a buon fine pazienza, uscirà errore 500
oppure un risultato vuoto. Generalmente è necessario qualche cosa di più sofisticato
ma in questo caso non serve niente di più di questo.

## JQuery ed Ajax

Il passo successivo è stato di sostituire la parte Word direttamente da un HTML con JQuery
che tramite AJAX va a leggere i dati dalla Web Api.

Di seguito le parti più interessanti del codice, ovvero come configurare la pagina
in formato A4, con la sezione "header" invisibile in fase di stampa, e con l'impostazione
delle interruzioni di pagina:

{% highlight css %}
@page {
  size: A4;
  margin: 0;
}
@media print, screen {
        html, body {
                width: 210mm;
                height: 297mm;
                padding-left: 20px;
                padding-right: 40px;
        }
        h1 {
                font: 16px Arial, sans-serif;
                font-style: normal;
                text-align: center;
        }
        h2 {
                ...
        }
        ...
}
@media print {
        #header {
                display: none;
        }
        #consenso1 {
                page-break-after: always;
        }
}
{% endhighlight %}

nella pagina HTML sarà presente una semplica form dove caricare il codice fiscale ed altri dati fissi,
e dove saranno presenti due pulsanti uno per la ricerca, uno per la stampa:

{% highlight html %}
<form id='codice'>
Telefono: <input type='text' name='telefono' value=''><br/>
Codice fiscale: <input type='text' name='cf'>
<button type='submit' value='cerca'>Cerca</button><br/>
<button type='submit' value='stampa'>Stampa</button>
</form>
{% endhighlight %}

il codice JavaScript / JQuery è il seguente:

{% highlight javascript %}
jQuery(document).ready(function(){
        $( "#codice button" ).click(function( event ) {
                if($(this).attr("value")=="cerca") {
                        # copia dei valori fissi
                        $('span#telefono1').text($("#codice :input[name='telefono']").val());
                        $('span#telefono2').text($("#codice :input[name='telefono']").val());

                        # sbianca i campi
                        $('span#cognome').text('');
                        $('span#nome').text('');
                        ...

                        # chiama la web api
                        $.ajax({
                                data: {
                                        cf: $("#codice :input[name='cf']").val()
                                },
                                type: "GET",
                                url: "anagrafe",
                                cache: false,
                                dataType: "text",
                                success: function(data) {
                                        var res = data.split("::");
                                        $('span#codice_fiscale').text('*' + res[0] + '*');
                                        $('span#cognome').text(res[1]);
                                        ...
                                }
                        });
                } else if($(this).attr("value")=="stampa") {
                        window.print();
                }
                event.preventDefault();
        });
});
{% endhighlight %}

## Codice a Barre

I codici a barre stampati nella tessera sanitaria utilizzano il formato "Code 39".
Per stampare il codice con lo stesso stile è necessario caricare ed il relativo font,
disponibile anche tra i font di google:

{% highlight css %}
@import url('https://fonts.googleapis.com/css2?family=Libre+Barcode+39+Text&display=swap');
span#codice_fiscale {
        font-family: 'Libre Barcode 39 Text', cursive;
        font-size: 36px;
}
{% endhighlight %}

Necessario ricordarsi che il codice, per essere letto correttamente dalla penna ottica,
deve essere preceduto e seguito da asterisco.

## Conclusioni

Dopo le prime settimane in cui regnava la confusione, ora la campagna vaccinale
è partita a pieno ritmo, gli appuntamenti sono gestiti tramite un portale di buona qualità
che viene messo a disposizione dalla regione (durante i periodi di picco aggiungono diversi
server virtuali, grazie al cloud ma anche grazie ad una buona progettazione) ed i
pazienti si congratulano per la buona organizzazione.

Il modulo consenso descritto in questo articolo e sviluppato e messo a punto nel giro di due
pomeriggi è ancora attivo, e rimane comunque un valido supporto al servizio.