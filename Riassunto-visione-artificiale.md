---
title: Visione Artificiale
author: Francesco Carlucci
date: 08/06/2023
---

# Visione Artificiale

## Indice

- [Visione Artificiale](#visione-artificialei)
      - [Indice](#indice)
- [Visione Artificiale](#visione-artificiale)
  - [Indice- Visione Artificiale](#indice--visione-artificiale)
- [Immagini](#immagini)
  - [RGB](#rgb)
  - [HSV/HSL](#hsvhsl)
  - [Istogramma](#istogramma)
  - [Lookup Table (LUT)](#lookup-table-lut)
  - [Binarizzazione](#binarizzazione)
  - [Trasformazioni geometriche](#trasformazioni-geometriche)
  - [Trasformazioni affini](#trasformazioni-affini)
- [Calibrazione Telecamera](#calibrazione-telecamera)
  - [Parametri estrinseci](#parametri-estrinseci)
  - [Parametri intrinseci](#parametri-intrinseci)
  - [Calibrazione](#calibrazione)
- [Filtri](#filtri)
  - [Filtri lineari](#filtri-lineari)
    - [Normalizzazione](#normalizzazione)
    - [Filtri separabili](#filtri-separabili)
    - [Riduzione del rumore](#riduzione-del-rumore)
    - [Sharpening](#sharpening)
  - [Estrazione dei bordi](#estrazione-dei-bordi)
    - [Difference of Gaussians (DoG)](#difference-of-gaussians-dog)
    - [Filtri derivativi](#filtri-derivativi)
    - [Gradiente](#gradiente)
    - [Canny edge detector](#canny-edge-detector)
    - [Laplaciano](#laplaciano)
      - [Laplacian of Gaussian (LoG)](#laplacian-of-gaussian-log)
  - [Filtri non lineari](#filtri-non-lineari)
    - [Median filter](#median-filter)
- [Analisi di immagini binarie](#analisi-di-immagini-binarie)
  - [Immagini binarie](#immagini-binarie)
    - [Topologia digitale](#topologia-digitale)
    - [Metriche discrete](#metriche-discrete)
  - [Trasformata distanza](#trasformata-distanza)
  - [Estrazione del contorno](#estrazione-del-contorno)
  - [Componenti connesse](#componenti-connesse)
    - [Etichettatura delle componenti connesse](#etichettatura-delle-componenti-connesse)
  - [Morfologia matematica](#morfologia-matematica)
    - [Elemento strutturante](#elemento-strutturante)
    - [Operazioni morfologiche](#operazioni-morfologiche)
- [Movimento nei video](#movimento-nei-video)


# Immagini

## RGB
---
L'RGB è un modello di visualizzazione dei colori additivo, cioè si basa sulla somma della luminosità dei tre colori primari (rosso, verde, blu) per ottenere tutti i colori.

Ogni colore rappresenta un punto in uno spazio a tre dimensioni.

Non raggruppa spazialmente i colori percepiti come simili dall'uomo.

## HSV/HSL
---
L'HSV e l'HSL sono modelli basati su Hue, Saturation, Value o Lightness.

L'asse verticale comprende i grigi, dal nero (luminosità 0) al bianco (luminosità 1).

I colori si trovano attorno al cilindro con saturazione 1 e luminosità 1 (HSV) o 0.5 (HSL).

Sono utilizzati per riconoscere gli oggetti nelle immagini.

Raggruppano spazialmente i colori percepiti come simili dall'uomo.

Non sono adatti ad una descrizione quantitativa dei colori.

<img src="./Riassunto_res/HSV.png" width="250" title="HSV">
<img src="./Riassunto_res/HSL.png" width="250" title="HSL">

## Istogramma
---
L'istogramma di un'immagine grayscale indica il numero di pixel dell'immagine per ciascun livello di grigio.

Con un istogramma si può evincere se un immagine è poco contrastata (valori condensati in una zona) o se è chiara/scura.

L'equalizzazione dell'istogramma permette di migliorarne il contrasto.

E' utilizzato per schiarire/scurire un'immagine.

Può essere utilizzato anche per separare gli oggetti dallo sfondo, se entrambi sono omogenei.

## Lookup Table (LUT)
---
La lookup table è utilizzata se il numero di colori nell'immagine è inferiore al numero di pixel per aumentare l'efficienza delle operazioni sui pixel.

E' un array in cui si mappano i valori di output per ogni input.

Viene utilizzata per convertire immagini grayscale in RGB, binarizzazione, equalizzazione di un istogramma.

## Binarizzazione
---
La binarizzazione viene usata per separare gli oggetti dallo sfondo.

Si usa una soglia globale se oggetti e sfondo sono uniformi.

Altrimenti si utilizza una soglia locale determinata per ogni pixel in base al suo intorno.

## Trasformazioni geometriche
---
Le trasformazioni geometriche sono trasformazioni che si applicano alle coordinate e non ai valori dei pixel.

Con il mapping diretto si va a mappare ogni pixel della vecchia immagine nella nuova immagine.

Problemi e relative soluzioni:

- Valore dei pixel fuori dalla nuova immagine
  - Colore di background costante
  - Copia del pixel più vicino
  - Riflessione dei pixel dell'immagine
  - Replica dell'immagine
  - Lascia il valore già presente nell'immagine destinazione
- Valore dei pixel nelle coordinate non intere
  - Il metodo più semplice è scegliere il valore del pixel più vicino
  - L'interpolazione, invece, applicando una funzione ai pixel in un intorno, stima il valore del pixel
- Buchi nella nuova immagine

Il mapping inverso è una trasformazione che parte dalla destinazione e ritorna alla sorgente.

## Trasformazioni affini
---
Una trasformazione affine può essere rappresentata come moltiplicazione per una matrice (fattore di scala + rotazione) e somma di un vettore (traslazione).

Si possono combinare più trasformazioni affini moltiplicando fra loro le corrispondenti matrici.

A differenza delle trasformazioni proiettive preserva il parallelismo fra rette.

# Calibrazione Telecamera

## Parametri estrinseci
---
Poiché il sistema della camera non è lo stesso della scena 3D, è necessario trasformare le coordinate 3D della scena in coordinate 2D della camera.

Per farlo si utilizzano i parametri estrinseci:

- Una matrice di rotazione $R$ (ottenuta a partire da tre angoli)
- Un vettore di traslazione $t$ (tre componenti)

## Parametri intrinseci
---
Per determinare la trasformazione dalla scena 3D all'immagine è necessario conoscere i parametri intrinseci della camera:

- Lunghezza focale $f$
- Centro dell'immagine $(c_x, c_y)$
- La relazione tra la dimensione dei pixel e l'unità di misura fisica $(s_x, s_y)$ (quanto misura un pixel)

## Calibrazione
---
La calibrazione consiste nel determinare i parametri intrinseci ed estrinseci della camera.

Conoscendo un certo numero di corrispondenze tra punti 3D della scena e punti 2D dell'immagine è possibile calcolare i parametri estrinseci e intrinseci.

Di solito si utilizza una scacchiera per calibrare la camera poiché è semplice individuare al suo interno un insieme di punti noti.

# Filtri

Un filtro $F$ è una matrice quadrata $m$ ⨉ $m$ dispari.

## Filtri lineari
---
Un filtro lineare è un operazione locale (il valore del pixel risultato dipende da un intorno del pixel di partenza) in cui il valore di un pixel è calcolato come somma pesata dei valori dei pixel di un intorno del pixel di partenza.

### Normalizzazione
---
Se i coefficienti del filtro sono tutti $≥0$ e si vuole mantenere lo stesso range di valori dell'immagine di partenza si normalizza il filtro dividendo per la somma dei coefficienti.

### Filtri separabili
---
Un filtro è separabile se la matrice $F$ può essere scritta come prodotto di un vettore colonna per un vettore riga.

In questo modo si può applicare il filtro $F$<sub>$x$</sub> e successivamente $F$<sub>$y$</sub>.

>### Correlazione
>
>$I'=I⨂F$
>
>### Convoluzione
>
>$I'=I*F$
>
>Si ottiene ribaltando il filtro sui due assi.

### Riduzione del rumore
---

- **Box filter** → riduce il rumore "sfocando" l'immagine.
- **Gaussian filter** → risolve il problema della creazione di artefatti utilizzando pesi che diminuiscono spostandosi dal centro verso l'esterno del filtro.

### Sharpening
---
Un filtro di sharpening è un filtro che aumenta la nitidezza di un'immagine.

1. Si calcola l'immagine blurrata e la si sottrae all'originale.
    
    $M=I-I*F$<sub>$b$</sub>

2. Si moltiplica il risultato ottenuto per un parametro $k$ (che determina l'intensità dell'effetto) e si somma all'originale.

    $I'=I+k$ <sup>.</sup> $M$

## Estrazione dei bordi
---
Considerando un'immagine grayscale come una funzione $f$ i bordi si trovano in corrispondenza di cambiamenti rapidi di $f$.

### Difference of Gaussians (DoG)
---
Il filtro gaussiano riduce le alte frequenze nell'immagine:

$I*G$<sub>$σ$</sub> = basse frequenze

Sottraendo il risultato dello smooth dall'immagine originale si evidenziano le zone caratterizzate da cambiamenti rapidi fra pixel chiari e scuri:

$I-I*G$<sub>$σ$</sub> = alte frequenze

Dati due filtri $G$<sub>$σ1$</sub> e $G$<sub>$σ2$</sub>, la differenza $(I*G$<sub>$σ1$</sub>$)-(I*G$<sub>$σ2$</sub>$)$ evidenzia un certo range di frequenze nell'immagine e dopo aver normalizzato emergono i bordi.

Per la proprietà distributiva della [convoluzione](#convoluzione) il filtro DoG si può scrivere come:

> $I*(G$<sub>$σ1$</sub>$-G$<sub>$σ2$</sub>$)$

### Filtri derivativi
---
Facendo le derivate parziali sugli assi $x$ e $y$ si ottengono le alte frequenze nell'immagine, ma poiché sono influenzate dal rumore dell'immagine si applica prima un filtro di smooth.

I principali filtri derivativi $3$⨉$3$ (in ordine decrescente di errori) sono: 

- Prewitt → il più efficiente
- Sobel → il più usato
- Scharr → migliore invarianza rotazionale

### Gradiente
---
Il gradiente in un punto è il vettore che ha per componenti le sue derivate parziali.

In un punto di un'immagine indica la direzione di maggior variazione dell'immagine nel punto.

Osservando l'orientazione del gradiente si evince che il bordo di un'immagine è ortogonale al gradiente.

Si può impostare una soglia al modulo del gradiente per selezionare i bordi, ma ciò può produrre:

- bordi non connessi
- bordi spuri
- bordi spessi

### Canny edge detector
---
L'algoritmo di Canny è formato da quattro step:

1. Gaussian blur dell'immagine
2. Calcolo del gradiente per ogni pixel
3. Soppressione dei non massimi in direzione ortogonale al bordo
   
    Serve ad eliminare dall'immagine del modulo del gradiente i pixel in cui il modulo del gradiente non è massimo locale rispetto all'orientazione del gradiente.

4. Selezione dei bordi significativi con l'isteresi

    L'immagine viene binarizzata utilizzando due soglie T<sub>1</sub> e T<sub>2</sub> con T<sub>1</sub> > T<sub>2</sub>.

    Inizialmente si considerano solo i pixel con il modulo del gradiente superiore a T<sub>1</sub>.

    Successivamente si considerano anche quelli con modulo superiore a T<sub>2</sub> se adiacenti a pixel già considerati.

### Laplaciano
---
Il Laplaciano è la somma delle due derivate parziali seconde in un punto.

In un immagine con un bordo sufficientemente contrastato il laplaciano è:

- zero lontano dal bordo
- zero in corrispondenza del bordo
- positivo vicino al bordo (dal lato in cui i pixel diventano più chiari avvicinandosi al bordo)
- negativo vicino al bordo (dal lato opposto)

#### Laplacian of Gaussian (LoG)

---
Laplaciano a cui è stato applicato un smooth gaussiano e, grazie alla proprietà associativa, si può ottenere calcolando il laplaciano
della funzione gaussiana:

$I*(S$<sub>$G$ </sub>$*$ $F$<sub>$∆$</sub>$)$

## Filtri non lineari
---

### Median filter
---
Il median filter è un operazione locale in cui il valore di ciascun pixel è la mediana dei valori dei pixel nell'intorno del pixel considerato.

Può ridurre alcuni tipi di rumore preservando maggiormente i bordi rispetto ai filtri di smooth lineari.

# Analisi di immagini binarie

## Immagini binarie
---
Le immagini binarie sono immagini che hanno solo due colori:

- Foreground $F$ → **255**
- Background $F$* → **0**

Sono alla base del rilevamento e del riconoscimento di oggetti nelle immagini.

### Topologia digitale
---
La topologia digitale è una disciplina che studia le proprietà e le caratteristiche topologiche di un immagine (componenti connesse, bordi).

### Metriche discrete
---
Le metriche discrete stabiliscono quali pixel sono *vicini* di un pixel, ovvero quelli che hanno distanza 1 dal pixel considerato.

I tipi più comuni sono:

- Distanza d<sub>4</sub> (City block , Manhattan)
  
  ![distanza d4](./Riassunto_res/distanza_d4.png "distanza d4")

- Distanza d<sub>8</sub> (Chessboard , Chebyshev)
  
  ![distanza d8](./Riassunto_res/distanza_d8.png "distanza d8")

## Trasformata distanza
---
La trasformata distanza di $F$ rispetto a $F$* è un'immagine in cui il valore dei pixel è la distanza di ciascun pixel da $F$*.

Per calcolare la trasformata distanza con la metrica d<sub>4</sub> si effettuano due scansioni dell'immagine:

- Diretta → dall'alto verso il basso, da sinistra a destra
- Inversa → dal basso verso l'alto, da destra a sinistra

Durante le scansioni i pixel vengono trasformati e quelli dello sfondo messi a 0.

La trasformata distanza può essere utilizzata per:

- Misurazioni geometriche
- Generare lo scheletro di un oggetto
- Template matching
- Robotica (aggirare ostacoli)

## Estrazione del contorno
---
Il contorno di $F$ è dato dalla sequenza ordinata di pixel che hanno distanza unitaria da $F$*.

Per l'estrazione del contorno di solito si utilizza una tecnica di inseguimento che percorre il bordo nella stessa direzione fino a tornare al pixel di partenza.

## Componenti connesse
---
Un percorso di lunghezza $n$ è una sequenza di n pixel vicini.

Una componente connessa è un sottoinsieme di $F$ tale che, presi due qualsiasi dei suoi pixel, esiste un percorso tra di loro appartenente a $F$.

### Etichettatura delle componenti connesse
---
L'etichettatura delle componenti connesse consiste nell'individuare ed etichettare automaticamente le componenti connesse di un'immagine:

- Si scorre l'immagine e per ogni pixel di foreground `p` si considerano i pixel già visitati:
  - se nessuno è etichettato si assegna una nuova etichetta a `p`
  - se uno è etichettato si assegna la stessa etichetta a `p`
  - se più di uno è etichettato si annotano le equivalenze e si assegna a `p` una delle etichette
- Per ogni insieme di etichette equivalenti se ne definisce una e si effettua un'altra scansione

## Morfologia matematica
---
La morfologia matematica è una tecnica di analisi ed elaborazione di immagini binarie derivata dalla teoria degli insiemi.

Serve a:

- Estrarre informazioni per descrivere la forma degli oggetti
- Rimuovere particolari irrilevanti dagli oggetti

### Elemento strutturante
---
Immagine binaria con dimensioni dispari utilizzata come parametro nelle operazioni morfologiche.

### Operazioni morfologiche
---
Gli operatori principali delle operazioni morfologiche sono:

- Dilatazione: $F⨁S=\{q|(𝑆^r)$<sub>$q$</sub>$∩F≠∅\}$
  - La nuova immagine è l’insieme dei pixel tali che, traslando in essi $S^r$, almeno uno dei suoi
elementi è sovrapposto a $F$
- Erosione: $F$⊖$S=\{q|(𝑆)$<sub>$q$</sub>$⊆F\}$
  - La nuova immagine è l’insieme dei pixel tali che, traslando in essi $S$, l’intero elemento strutturante è contenuto in $F$

Da cui si ottengono le seguenti operazioni morfologiche:

- Apertura:  $F∘S=(F$⊖$S)⨁S$
  - Erosione seguita da dilatazione
  - Separa oggetti debolmente connessi e rimuove regioni piccole
  - Si utilizza come elemento strutturante un elemento che è più
grande della regione da eliminare.
- Chiusura: $F•S=(F⨁S)$ ⊖ $S$
  - Dilatazione seguita da erosione
  - Riempie buchi e rafforza la connessione di regioni unite debolmente
  - Si utilizza come elemento strutturante un elemento che è più
grande dei buchi da chiudere.

# Movimento nei video


