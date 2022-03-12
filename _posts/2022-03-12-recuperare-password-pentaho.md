---
layout: post
title: "Recuperare una Password cifrata da Pentaho"
date: 2022-03-12 22:11:00 +0200
categories: pentaho, java
---
## Connessioni Database

Se si utilizza un repository centralizzato su database,
Pentaho registra le informazioni legate alle connessioni (`host_name`, `database_name`, ..) nella tabella
`r_database`, assieme alle credenziali in formato "offuscato".
Non si tratta di un hash monodirezionale ma di un formato che, pur non direttamente leggibile, può
essere facilmente decifrato.

````sql
select
  id_database,
  name,
  username,
  password
from
  r_database
````

## Recupero password Offuscate

Le password offuscate sono precedute dalla stringa `Encrypted`. Se prendiamo come esempio la password `prova`, questa verrà salvata in tabella con la stringa `Encrypted 2be98afc86aa7f2e4cb79ce60cc9db9db`.

Quale metodo viene utilizzato per codificare e decodificare le password? Pentaho utilizza la libreria `Encr.java` che richiama diverse funzioni presenti in `KettleTwoWayPasswordEncoder.java`. 

Un sistema per decifrare la password è dimostrato nella trasformazione [Pentaho-Kettle-Password-Decrypt](https://github.com/RHeijmann/Pentaho-Kettle-Password-Decrypt) che lo trovo interessante perché permette di
integrare la libreria Java che gestisce l'offuscamento delle password all'interno di una trasformazione,
importanto la libreria e definendo la funzione da richiamare:

````java
import org.pentaho.di.core.encryption.Encr;
Encr.decryptPassword(encrypted_password)
````

Tuttavia non è strettamente necessario utilizzare questo sistema.

Facendo una breve analisi del codice sorgente, si riesce facilmente ad individuare la logica del funzionamento, ovvero la definizione di una chiave "segreta":

````java
public KettleTwoWayPasswordEncoder() {
    String envSeed = Const.NVL( EnvUtil.getSystemProperty( Const.KETTLE_TWO_WAY_PASSWORD_ENCODER_SEED ), "0933910847463829827159347601486730416058" ); // Solve for PDI-16512
    Seed = envSeed;
}
````

(che corrisponde alla costante `KETTLE_TWO_WAY_PASSWORD_ENCODER_SEED` nel caso essa sia configurata,
oppure al valore fisso `0933910847463829827159347601486730416058`)

e l'algoritmo di codifica/decodifica che, senza sorpresa, avviene semplicemente tramite OR esclusivo. Il codice che decifra la password, semplificando al massimo e togliendo diversi controlli, è sostanzialmente questo:

````java
import java.math.BigInteger;

public class Main
{
        public static void main(String[] args) {
                String envSeed = "0933910847463829827159347601486730416058";
                String encrypted = "2be98afc807ce808daa1dab65d281bc8e";
                int RADIX = 16;

                BigInteger bi_confuse = new BigInteger( envSeed );
                BigInteger bi_r1 = new BigInteger( encrypted, RADIX );
                BigInteger bi_r0 = bi_r1.xor( bi_confuse );

                System.out.println(new String( bi_r0.toByteArray() ));
        }
}
````