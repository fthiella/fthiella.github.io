---
layout: post
title: "Dynamic Markdown"
date: 2023-09-11 22:06:00 +0200
categories: markdown
---

Markdown è un sistema di markup che può essere utilizzato per aggiungere elementi di formattazione a
dei normali file di testo. È molto apprezzato dagli sviluppatori, e non a caso è il sistema di riferimento
per [GitHub](https://github.com/) oppure per [StackOverflow](https://stackoverflow.com/) ma può essere
utilizzato per qualsiasi tipo di documento, compresi report, libri, manuali, testi, etc.

Uno strumento indispensabile per lavorare, tra le altre cose, su documenti markdown è
[pandoc](https://pandoc.org/) che viene definito come il coltellino svizzero per i documenti di testo.
Può essere infatti utilizzato per convertire documenti markdown in documenti Word, Open Office, LaTeX, PDF, etc.

Ad esempio:

    pandoc test.md -s -o test.odt

oppure:

    pandoc test.md -s -o test.pdf

(necessario aver installato pdflatex, come ad es. portable MikTex, e configurato il `PATH` in maniera adeguata).

È possibile inoltre applicare dei template di alta qualità, come il
[template Eisvogel](https://github.com/Wandmalfarbe/pandoc-latex-template)

    pandoc test.md -s -o test.pdf --template eisvogel

## Markdown Dinamico

Ho pensato che sarebbe estremamente comodo poter disporre di un documento markdown "dinamico". Ad esempio, per inserire
delle tabelle in un documento Markdown posso utilizzare il
[plugin MarkDown per Adminer](https://fthiella.github.io/markdown-plugin-for-adminer) che ho scritto qualche tempo
fa, in modo molto semplice e con i risultati spesso migliori di un copia-incolla da Excel su documento Word!

Tuttavia sarebbe meglio ancora poter pescare i dati direttamente dal database di origine (così come da un CSV, XLSX, o
quant'altro).

Un sistema che potrebbe funzionare è [Jinga2](https://pypi.org/project/Jinja2/) ovvero uno dei più utilizzati framework
per template di Python. Questo sistema però, per precisa scelta architetturale, non prevede la possibilità di mescolare
codice di programmazione con condice markup. Avendo vissuto in prima persona gli anni '90 dello scorso millennio, dove il
codice HTML si mischiava al codice di programmazione senza capire bene il limiti di dove finiva uno ed iniziava l'altro,
posso dire che tale scelta è perfettamente condivisibile. Tuttavia, pur essendo una opzione sempre valida, non è quello
che stavo cercando!

## Linguaggi di scripting?

Si potrebbe usare un Makefile con qualche script bash, magari integrato con dei sistemi che generano output in formato markdown come il mio
[perl-Sql-Textify](https://github.com/fthiella/perl-Sql-Textify).

Oppure PHP che, nel bene e nel male, permette di inserire del codice all'interno di un documento e può quindi essere utilizzato per
rendere un documento Markdown dinamico.

Oppure ancora il buon vecchio [HTML::Mason](https://github.com/houseabsolute/HTML-Mason), che mi ha dato grandi soddisfazioni
all'inizio del millennio permettendo di integrare codice *HTML* con il linguaggio *perl*. Purtoppo né *HTML::Mason* né *Mason2*
sono attivamente sviluppati da anni, anche se proprio nel momento in cui ho iniziato a scrivere questo post (ovvero il giorno
11 febbraio 2023 - scrivo molto lentamente!) è stato pubblicato *HTML::Mason* versione 1.60 che contiene un piccolo bugfix.
Non ci sono poi aggiornamenti successivi.

## Script Python?

Ho pensato di risolvere il problema con dei piccoli script Python. Un documento Markdown sarà composto come segue, con del codice
Python inserito all'interno del documento:


````
---
title: "Report Dinamico"
author: [Federico Thiella]
date: "2023-09-11"
subject: "Report Dinamico"
keywords: [codice, python, esempio]
lang: "it"
table-use-row-colors: True
book: True
...
^ import mktools as mk
^ c={'host':'myhost.mshome.net', 'database':'mydatabase', 'user':'myuser', 'password':'mypassword'}
# Report Dinamico creato con Markdown

^ q=mk.query_db("select * from sezioni", c)
^ for r in q['rows']:

## Titolo: <& r[2] &>

<& r[3] &>

Inserisci una tabella:

^   w=mk.query_db("""
^      select
^        col1,
^        col2,
^        col3
^      from
^        progetti
^      where
^        progetto_id={}
^      order by
^        col1, col2, col3
^      """.format(str(r[0])), c)
^   w['head'][1]['name']='Colonna 1'
^   w['head'][1]['align']='right'
^   w['head'][2]['name']='Colonna 2'
^   w['head'][2]['align']='right'
^   w['head'][3]['name']='Colonna 3'
^   w['head'][3]['align']='left'
^   mk.markdown_table(w['head'], w['rows'])
^^
\pagebreak
````

1. questo codice Markdown dinamico verra compilato dallo script `pp.py` in uno script puro python,
   che può essere eseguito e che genererà in output il codice Markdown statico finale, pronto da
   poter essere utilizzato e/o convertito in altri formati;
2. la libreria `mktools.py` contiene delle funzioni utili per eseguire velocemente delle operazioni
   su database:
   - mk.query_db esegue una query e restituisce intestazione w['head'] e righe w['rows'];
   - mk.markdown_table che converte la tabella descritta da intestazione e righe in formato markdown.

La compilazione del documento avviene in questo modo:

    python pp.py -i documento.md | python | pandoc -o documento.pdf --template eisvogel.latex -s

(lo script dinamico e la libreria di utilities saranno pubblicate e descritte su GitHub quanto prima.
Non si tratta di un sistema "professionale" ).

## Mako Templates

Un'ultima opzione può essere il sistema [Mako Templates](https://www.makotemplates.org/),
la cui filosofia _"Don't reinvent the wheel...your templates can handle it!"_ contrasta con
il mio paragrafo precedente. Lo approfondirò appena mi sara possibile!
