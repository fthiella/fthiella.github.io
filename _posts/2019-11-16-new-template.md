---
layout: post
title:  "New Template"
date:   2019-11-16 23:59:00 +0200
categories: jekyll, github, markdown
---

Quando ho iniziato questo blog su GitHub, oltre 2 anni fa, mi sono basato semplicemente sul modulo
[Jekyll Now](https://github.com/barryclark/jekyll-now) che, una volta clonato, permette di disporre
di un template subito pronto per la pubblicazione degli articoli del blog, scritti con sintassi
markdown.

Inoltre, ho effettuato solo minime modifiche al template (un po' di css per gestire le tabelle,
aggiunta del modulo disqus e di recente mathjax, configurazione, etc.).

Le modifiche "importanti" sono però difficili da applicare, in quanto non disponendo di un ambiente
Jekyll installato sul mio PC per vedere una preview è necessario fare il push delle modifiche su GitHub,
attendere qualche secondo/minuto e vedere il risultato direttamente online.

Di recente ho installato la `bash` sulla mia postazione Windows, ed ho quindi voluto installare anche
l'ambiente `Jekyll` in modo da poter vedere le modifiche in tempo reale.

Ho seguito questa guida [Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/) e mi
sono basato sulla soluzione `bash`. Riassumento ho lanciato i seguenti comandi:

{% highlight bash %}
bash

sudo apt-get update -y && sudo apt-get upgrade -y

sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.5 ruby2.5-dev build-essential dh-autoreconf

gem update

gem install jekyll bundler
{% endhighlight %}

modificando quando necessario il file `.bashrc`

{% highlight bash %}
export GEM_HOME=$HOME/gems
export PATH=$HOME/gems/bin:$PATH
{% endhighlight %}

tutti i dettagli sono nella guida. Non sono esperto della piattaforma Ruby (e non lo voglio diventare) per cui ho
sostanzialmente copiato passo passo tutti i comandi.

Ho poi seguito la guida per creare un blog di test:

{% highlight bash %}
gem install jekyll bundler
jekyll new myblog
cd myblog
bundle exec jekyll serve
{% endhighlight %}

ho copiato il `GemFile` generato nella cartella dove tengo il blog GitHub,
modificando contemporaneamente il mio `.gitignore`, e da questa cartella
posso lanciare il comando `bundle exec jekyll serve` ed il mio blog è
pubblicato in tempo reale su localhost.

Ho risolto alcuni errori. In particolare alcuni tag di Jekyll venivano
erroneamente attivati nel mio codice, ad esempio per rendere correttamente
il seguente codice:

{% highlight html %}
{% raw %}
{% include mathjax.html %}
{% endraw %}
{% endhighlight %}

è in realtà necessario aprire il codice con `{% raw %}{% raw %}{% endraw %}` e chiuderlo con `{% raw %}{% end raw %}{% endraw %}` . Naturalmente
va tolto lo spazio tra `end` e `raw`! Il codice lo evidenzio con `{% raw %}{% highlight python %}{% endraw %}` e `{% raw %}{% endhighlight %}{% endraw %}`.

Ho aggiornato il tema prendendolo da [jekyll-solarized-theme](https://github.com/rphlmr/jekyll-solarized-theme) con qualche piccolo aggiustamento
(tabelle, e larghezza da 500px a 800px)

Ora devo creare una pagina dove visualizzare i post per tag.