---
layout: post
title: "Pandas da SQL"
date: 2020-05-22 23:15:00 +0200
categories: postgresql, python, pandas
---
Pandas è una libreria per Python, che offre strutture dati per manipolare tabelle ed effettuare
analisi. Si tratta di uno strumento molto flessibile: io lo considero, facendo un paragone forse
un po' azzardato, una specie di Excel per Python.

Dopo aver lavorato ad un sistema data engineering/ETL di pulizia e caricamento dati COVID
(composto da query SQL Oracle e Postgresql, script bash, export in perl, pentaho data integration,
export in csv ed excel, integrazioni aws, etc.) ho pensato fosse interessante fare qualche
approfondimento ed analisi "scientifica" dei dati.

Pandas può leggere un dataframe da diverse origini, tra cui Excel o CSV, ma anche da una query
SQL, e questo è estremamente comodo. Ecco un esempio:

{% highlight python %}
from sqlalchemy import create_engine
import pandas as pd

import matplotlib as mpl
import matplotlib.pyplot as plt

engine = create_engine('postgresql://covid:12345678@pgsqlserver:5432/covid')
df=pd.read_sql_query('select * from covid.stage_covid',con=engine)

df.groupby(['data_prelievo']).size().plot(kind='line')

plt.title('Numero tamponi per giorno')
plt.ylabel('Tamponi')
plt.xlabel('Giorno')

plt.show()
{% endhighlight %}
