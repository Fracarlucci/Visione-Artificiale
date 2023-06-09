---
title: Visione Artificiale
author: Francesco Carlucci
date: 08/06/2023
---

# Visione Artificiale

## Indice

- [Visione Artificiale](#visione-artificiale)
      - [Indice](#indice)
- [Immagini](#immagini)
  - [RGB](#rgb)
  - [HSV/HSL](#hsvhsl)
  - [Istogramma](#istogramma)
  - [Lookup Table (LUT)](#lookup-table-lut)
  - [Binarizzazione](#binarizzazione)
  - [Trasformazioni geometriche](#trasformazioni-geometriche)
  - [Trasformazioni affini](#trasformazioni-affini)
- [Calibrazione](#calibrazione)
  - [something](#something)

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

<img src="./Risposte_res/HSV.png" width="250">
<img src="./Risposte_res/HSL.png" width="250">

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

# Calibrazione

## something