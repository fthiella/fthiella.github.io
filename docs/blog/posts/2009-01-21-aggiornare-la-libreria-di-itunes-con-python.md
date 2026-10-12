---
date: 2009-01-21
categories:
  - itunes
  - python
title: Aggiornare la libreria di iTunes con Python
---
Visto che la mia libreria di iTunes non ne voleva sapere di aggiornarsi correttamente, nemmeno utilizzando il buon [iTunes Library Updater](<http://itlu.ownz.ch/>), ho pensato di approfittare di questo piccolo ma fastidioso inconveniente per iniziare a conoscere - finalmente - il linguaggio di programmazione Python.  

  

Prerequisito 1 - Python!
  

Quale distribuzione di Python utilizzare? Poiché l'ambiente ActiveState Perl mi è molto familiare, mi sembrava naturale utilizzare ActiveState Python, ma ho notato che gli sviluppatori di Google, nel canale [Google Code](<http://www.youtube.com/user/GoogleDevelopers>), preferiscono utilizzare la versione ufficiale tratta dal sito[ www.python.org](<http://www.python.org/>) e allora... non so ancora perché, non so se e quando lo scoprirò, ma ho deciso di iniziare anch'io con questa versione: migliaia di sviluppatori Google non possono sbagliare!
  

Prerequisito 2 - Apple SDK
  

iTunes per Windows è controllabile tramite interfaccia COM: sul sito della Apple è gentilmente messo a disposizione il software developer kit con la [documentazione per l'interfaccia COM di iTunes](<http://developer.apple.com/sdk/itunescomsdk.html>).
  

Prerequisito 3 - libreria tagging MP3
  

Dopo anni di programmazione in Perl, sono giunto alla conclusione che le mie prime impressioni erano fin troppo vere: si tratta di un linguaggio brutto, sporco e cattivo. Ciò che lo rende comodo, ma non solo comodo, oserei dire quasi meraviglioso, è la sterminata [libreria di moduli](<http://search.cpan.org/>), generalmente di ottima qualità, progettati da progettisti degni di tale nome, che risolvono praticamente tutti i problemi della terra.
E per Python, esiste qualcosa di analogo?
Non lo so ancora, e quindi vai con google: il modulo [Mutagen](<http://code.google.com/p/quodlibet/wiki/Mutagen>) sembra proprio quello che stavo cercando, e se lo utilizzano con successo gli amici della comunità [MusicBrainz](<http://musicbrainz.org/>), allora lo posso utilizzare con successo anch'io. Forse [code.google.com](<http://www.blogger.com/code.google.com>) è (o diventerà) il repository di riferimento per i moduli Python? Forse l'intenzione è proprio quella: vedremo!
  

Il codice!
  

Ho ancora molto dubbi, ma copiando un po' di qua un po' di là, ecco finalmente il mio primo programma Python bello e funzionante:
    
    import win32com.client
    
    from mutagen.mp3 import MP3
    from mutagen.easyid3 import EasyID3
    import mutagen.id3
    
    itunes = win32com.client.Dispatch("iTunes.Application")
    mainLibrary = itunes.LibraryPlaylist
    tracks = mainLibrary.Tracks
    numTracks = tracks.Count
    n = 1
    v = 0
    
    while n <= numTracks:
      currTrack = tracks.Item(n)
      location = win32com.client.CastTo(currTrack,'IITFileOrCDTrack').Location
    
      if ((currTrack.Artist == "") or (currTrack.Name == "")):
        try:
          m = MP3(location, ID3=EasyID3)
          if m.has_key('artist'):
            currTrack.Artist      = m['artist'][0]
          if m.has_key('album'):
            currTrack.Album       = m['album'][0]
          if m.has_key('title'):
            currTrack.Name        = m['title'][0]
          if m.has_key('tracknumber'):
            currTrack.TrackNumber = m['tracknumber'][0]
          if m.has_key('date'):
            currTrack.Year        = m['date'][0]
          if m.has_key('genre'):
            currTrack.Genre       = m['genre'][0]
          v += 1
        except mutagen.id3.error:
          print "Error"
          continue
      n+=1
    
    print "Brani aggiornati: " + str(v)
    
Questo programma scorre tutta la libreria iTunes, ed aggiorna il nome dell'artista, il titolo dell'album e del brano, il numero di traccia, la data (uhm forse bisogna estrarre solo l'anno), ed il genere musicale, leggendoli direttamente dai tag del file MP3, per tutti i brani in cui l'artista oppure il brano sono vuoti.
Brutto, sporco, probabilmente buggato, con un cast misterioso e ancora molti perché... ma funzionale: per adesso è un buon inizio!
