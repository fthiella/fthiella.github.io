---
layout: post
title: "Appunti su Pandoc"
date: 2024-05-17 15:00:00 +0200
categories: markdown, pandoc
---

Alcune note riguardo pandoc

## Documenti Word

Generare un documento con gli stili di riferimento:

    pandoc -o custom-reference.docx --print-default-data-file reference.docx

Si possono quindi modificare gli stili all'interno del file "custom-reference" con le normali modalità di Microsoft Word.

Introdurre nuovi stili:

    <div custom-style="Super big">My super big text</div>
    Normal text. <span custom-style="Highlighted text">This is highlighted</span>

(devono essere esistenti nel file custom-reference.docx)

Si può quindi compilare il documento con:

    pandoc documento.md -o documento.docx --reference-doc=custom-reference.docx

Per quanto riguarda le tabelle non è purtroppo possibile modificare lo stile standard. Si può però effettuare un passaggio con python:

    import docx
    document = docx.Document('/tmp/xxx.docx')
    for table in document.tables:
        table.style = document.styles['custom_style']
    document.save("target.docx") 