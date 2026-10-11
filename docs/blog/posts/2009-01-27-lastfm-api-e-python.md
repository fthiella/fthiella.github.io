---
date: 2009-01-27
categories:
  - python
  - itunes
  - last.fm
---
# Last.fm API e Python

Il mio primo programma Python non è certamente un esempio da inserire in un manuale di stile, ma non era nemmeno quella l'intenzione; ci tenevo a scrivere qualcosa che fosse utile e che avesse risultati pratici fin da subito.  
Con piccole variazioni al codice ho potuto individuare i brani doppi, ho potuto correggere alcuni titoli errati, ho potuto individuare gli artisti che comparivano "sdoppiati" a causa del tag artistsortorder mancante.  
Ed ora, che cosa resta?  
Ho pensato che sarebbe stato interessante sistemare anche il genere musicale... 
## Last.fm
Last.fm è un social network di utenti accomunati dalla passione per la musica. Una caratteristica che lo rende interessante è la funzione di scrobbling: di ogni utente viene creato un profilo dettagliato che comprende tutti i brani ascoltati tramite l'apposita radio oppure tramite un plugin installato nel proprio player.   
Gli utenti inoltre possono aggiungere dei "tag", ovvero delle etichette, agli artisti, agli album, alle canzoni che ascoltano. Da questi tag è possibile ottenere molte informazioni, tra cui il genere musicale di un artista. Ma come si possono estrarre queste informazioni?   

## Last.fm API
Fortunatamente Last.fm mette a disposizione dei Web Services attraverso i quali è possibile estrarre tutte le informazioni presenti sul sito. Dato per scontato che non è sempre precisa (il "tag" è un'etichetta generica, che non sempre coincide col genere musicale), la funzione artist.getTopOfTags sembra proprio avvicinarsi il più possibile a quanto stavo cercando.   

## Il codice
    
    import sys
    import win32com.client
    import httplib
    import urllib
    import xml.dom.minidom
    
    dic = dict()
    
    def artistGetTopOfTags(artista):
      artist_url = u"http://ws.audioscrobbler.com/2.0/?method=artist.gettoptags&artist=%s&api_key=YOUR_KEY"
      url = artist_url % urllib.quote(artista.encode('utf8'))
      dom=xml.dom.minidom.parse(urllib.urlopen(url))
      genre=""
      for tag in dom.getElementsByTagName('tag'):
        name = ""
        count = ""
        for child in tag.childNodes:
          if child.nodeName == "name":
            name = child.firstChild.nodeValue
          if child.nodeName == "count":
         count = child.firstChild.nodeValue
        if count=="100":
          genre = name
      return genre.title()
    
    def genere(artista):
      if not artista in dic:
        dic[artista]=artistGetTopOfTags(artista)
        print artista
      return dic[artista]
    
    def main(*args):
      itunes = win32com.client.Dispatch("iTunes.Application")
      mainLibrary = itunes.LibraryPlaylist
      tracks = mainLibrary.Tracks
      numTracks = tracks.Count
      n = 1;
    
      while n <= numTracks:
        currTrack = tracks.Item(n)
        if genere(currTrack.Artist) != "":
          try:
            currTrack.Genre = genere(currTrack.Artist)
          except:
            print "eccezione!"
        n+=1
    
    if __name__ == '__main__':
      sys.exit(main(*sys.argv))
    
va bene, le eccezioni dovranno essere gestite meglio, e va bene, il genere musicale non è sempre corretto. Però i portali Web che mettono a disposizione delle funzioni tramite Web Services sono sempre più numerosi (es. Google, Yahoo Weather, etc...) e mi pareva interessante porre le basi per iniziare ad utilizzarli. Sembra che il metodo più pythoniano di parserizzare un file XML consiste nell'utilizzare la libreria ElemenTree, e non la libreria DOM standard... bene vedremo di approfondire anche questo!
