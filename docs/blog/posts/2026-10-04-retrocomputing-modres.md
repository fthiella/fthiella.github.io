---
title: "MODRES: reverse engineering di un TSR audio negli anni '90"
date: 2026-10-04
categories: [retrocomputing, storia]
---

> MODRES era la libreria audio di MODEDIT, un tracker per MOD
> molto diffuso in ambiente DOS. Non aveva documentazione
> pubblica, l'ho quindi ricostruita con il Turbo Debugger ed
> inviata a Ralf Brown, ed oggi nel 2026 tali voci sono ancora
> presenti nella Interrupt List.

## Cos'è MODRES

MODRES era un programma per DOS di tipo TSR (*Terminate and Stay
Resident*) che permetteva di riprodurre in background i moduli
musicali in formato MOD, quelle a quattro tracce dei tracker Amiga.

Il suo uso principale era all'interno di **MODEDIT**: MODRES era la
libreria audio che permetteva al tracker di ascoltare i moduli ed
i campioni mentre li si editava, e di continuare a suonarli
anche quando il tracker era in background.

Supportava:

- PC speaker
- D/A converter su porte parallele LPT1-LPT4
- Sound Blaster (porta 02x0h)
- Disney Sound Source
- Configurazioni stereo su due porte parallele
- Stereo-on-1

Esponeva le sue funzioni attraverso l'interrupt `INT 2F`, con
`AX=8220h` come base.

## Il reverse engineering

MODRES non aveva documentazione pubblica. Tramite il
**Turbo Debugger** della Borland ne ho seguito l'esecuzione
istruzione per istruzione, mettendo breakpoint ed esaminando
registri e memoria. Con molta pazienza, ho ricostruito
le sue logiche interne di funzionamento.

Le funzioni che avevo documentato sono sette. Le strutture dati due:
`MODPARM` e `SAMPARM`, più una tabella di output device e una dei
pitch.

## La documentazione

### MODRES - PLAY MODULE

    AX = 8220h
    DX:CX -> MODPARM structure (see #2646)

    Return:
    AX = status
      5722h successful
      2000h parameters out of range
      other MODRES not installed

    See Also: AX=8221h - AX=8223h - AX=8225h - AX=8227h - AX=8200h"RESPLAY"

### Format of MODPARM Structure (Table 2646)

    Offset  Size    Description
    00h     WORD    signature 504Dh ("MP" = Modparm)
    02h     BYTE    output device (see #2648 at INT 2F/AX=8221h)
    03h     WORD    segment of start of main module (pattern) data
    05h  31 WORDs   segment of start of sample numbers 1-31
    43h     BYTE    pattern at which to start playing (00h to 7Fh)
    44h     BYTE    function
                    00h play from pattern [offset 43h] until end of the song
                    01h play indicated pattern [offset 43h] only
    45h     BYTE    Machine speed
                    00h 10-12Mhz
                    01h 12-25Mhz (default)
                    02h 25Mhz+
                    03h mix speed 10kHz (fast 8Mhz machines)
                    04h mix speed 12kHz (10Mhz machines)
                    05h mix speed 13kHz
                    06h mix speed 8kHz (test for 8Mhz machines)
    46h     BYTE    allow >64k sample playing
                    80h MOD has samples >64k in it
                    else all samples in MOD are <64k

    Notes: Main module data and all samples must start on segment
    boundaries. In version 2.00 (ONLY) this function carries on
    playing (works in the background).

    See Also: #2647

### MODRES - INSTALLATION CHECK

    AX = 8221h

    Return:
    AX = status
      5722h successful
      other MODRES not installed
    BX = BCD version number (BH = major, BL = minor)
    DX:CX -> Output Device structure (read-only) (see #2647)

    See Also: AX=8220h - AX=8222h - AX=8225h - AX=8227h

### Format of Output Device structure [array] (Table 2647)

    Offset  Size    Description
    00h 20 BYTEs   ASCIZ name of the output device
                   (end of list if first char is FFh)
    14h    WORD    apparently always FFFFh
    16h    WORD    0000h if output device not available
                   else first I/O port for the output device
    18h    WORD    second I/O port for the output device (for example
                   if it is stereo)
                   0000h if only one port used or device is not available
    1Ah  7 BYTEs   ???

    See Also: #2646 - #2648

### Values for MODRES v1.52 output device index (Table 2648)

    00h    PC speaker
    01h    D/A Converter on LPT1
    02h    D/A Converter on LPT2
    03h    D/A Converter on LPT3
    04h    D/A Converter on LPT4
    05h    D/A Converter on LPT1&LPT2 (stereo)
    06h    D/A Converter on LPT1&LPT2 (mono)
    07h    Sound Blaster (port 02x0h)
    08h    User Defined D/A (mono)
    09h    User Defined D/A (stereo)
    0Ah    Stereo-on-1
    0Bh    Disney SS su LPT1
    0Ch    Disney SS su LPT2
    0Dh    Disney SS su LPT3
    0Eh    Disney SS su LPT4

    Note: This list may vary between versions of MODRES

### MODRES - UNINSTALL

    AX = 8222h

    Return:
    AX = code segment of the program

    Note: This function does not release the TSRs memory; the caller
    must do so

    See Also: AX=8220h - AX=8221h - AX=8223h

### MODRES - PLAY SAMPLE

    AX = 8223h
    DX:CX -> SAMPARM structure (see #2649)

    Return:
    AX = status
      5722h successful
      2000h parameters out of range
      other MODRES not installed

    See Also: AX=8221h - AX=8224h - AX=8225h - AX=8226h

### Format of SAMPARM Structure (Table 2649)

    Offset  Size    Description
    00h     WORD    signature 5053h ("SP" = SAMPARM)
    02h     WORD    segment of start of sample to play
    04h     WORD    length of sample (IN WORD)
    06h     BYTE    output device (see #2648 at INT 2F/AX=8221h)
    07h     WORD    pitch to play (see #2650)
    09h     BYTE    volume (from 00h to 40h)
    0Ah     WORD    loop start
    0Ch     WORD    loop length
    0Eh     BYTE    machine speed (see INT 2F/AX=8220h)

    See Also: #2646

### Values for Pitch to play (Table 2650)

    C 0 is 06B0h
    C#0 is 06B0h / 2^(1/12)
    D 0 is (06B0h / 2^(1/12)) / 2^(1/12)
    ...

    Note: C 1 is 06B0h / 2. C 2 is 06B0h / 4. Etc.

    See Also: #2649

### MODRES - ???

    AX = 8224h
    DX:CX -> ???

    Return:
    ???

    See Also: AX=8221h - AX=8223h - AX=8224h

### MODRES v2.00+ - GET LOCATION IN MOD

    AX = 8225h

    Return:
    AL = status
      00h playing
      01h reached end or stopped
    AH = speed of MOD
    BX = position within pattern 0000h-0400h
    CL = position within the song (track number)

    See Also: AX=8220h - AX=8221h - AX=8223h - AX=8226h

### MODRES v2.00+ - STOP PLAYING

    AX = 8226h

    Return:
    AX = status
      5722h successful
      other MODRES not installed

    Desc: Stops playing the MOD file before performing critical
    operations such as disk accesses

    See Also: AX=8220h - AX=8221h - AX=8223h - AX=8225h - AX=8227h

### MODRES - CONFIGURE

    AX = 8227h
    BX = function
      0001h set default playing speed (06h)
      0002h select output device
        CL = output device (see #2648 at INT 2F/AX=8221h)

    Return:
    AX = status
      5722h successful
      2000h parameters out of range
      other MODRES not installed

    Note: Function 0001h should be called every time a new module
    is loaded

    See Also: AX=8220h - AX=8221h - AX=8222h - AX=8223h

## Note sulla documentazione

Alcune cose che vale la pena notare, rileggendola oggi:

- La `MODPARM` inizia con una **signature** (`504Dh` = "MP"), come
  quasi tutte le strutture dati di quei tempi. Serviva a verificare
  che il puntatore passato alla funzione fosse davvero una struttura
  valida.
- Il campo **machine speed** non è un valore in MHz, ma un indice
  che il TSR usa per scegliere la frequenza di mixaggio. La voce
  `06h` è "test for 8MHz machines": il programma si adattava alla
  macchina su cui girava.
- La tabella dei pitch parte da **C 0 = 06B0h** e calcola le note
  successive come divisioni per `2^(1/12)`. È il temperamento equabile
  applicato ai registri del timer. Quasi certamente però il programma
  utilizzava una tabella precompilata per evitare di calcolare i
  valori a runtime, che però non sono riuscito a recuperare.
- Non sono riuscito a capire il significato della funzione `AX=8224h`
  che è rimasta **non documentata**.

## Dove è finita

La documentazione è nella **Ralf Brown's Interrupt List**, alla voce
`INT 2F/AX=8220h` e seguenti. È ancora consultabile su
[ctyme.com/rbrown.htm](http://www.ctyme.com/rbrown.htm).

## Il riconoscimento

Nella sezione CREDITS della Release 55 (28 settembre 1997) della
Ralf Brown's Interrupt List, c'è questa riga:

    12/96 A Federico Thiella fthiella@stud32.math.unipd.it  MODRES

Dicembre 1996. Università di Padova. MODRES.
