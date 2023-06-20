---
title: Visione Artificiale
author: Francesco Carlucci
date: 16/06/2023
---

# Esercizi esami

## Indice

- [Esercizi esami](#esercizi-esami)
  - [Indice](#indice)
  - [Cheat sheet](#cheat-sheet)
  - [2022/05/05](#20220505)
  - [2022/06/08](#20220608)
  - [2022/07/13](#20220713)
  - [2022/09/08](#20220908)
  - [2023 Prova esame](#2023-prova-esame)
  - [2013/01/15](#20130115)
  - [2013/06/26](#20130626)
  - [2013/09/17](#20130917)
  - [2014/01/29](#20140129)
  - [2017/06/13](#20170613)

## Cheat sheet

- `np.argmax(a, axis)` -> Returns the indices of the maximum values along an axis.
- `axis = 1` -> righe
- `np.where(condition, x, y)` -> Return elements chosen from x if the condition is satisfied, y otherwise. Folowed by: `.np.astype(uint8)`
- `np.argwhere(array)` -> Find the indices of array elements that are non-zero, grouped by element.
- `np.isin(element, test_elements)` -> Returns a boolean array of the same shape as element that is True where an element of element is in test_elements and False otherwise.
- `cv.merge(matrices*)` -> Creates one multi-channel array out of several single-channel ones.
- `np.count_nonzero(array)` -> Counts the number of non-zero values in the array.
- `enumerate(array)` -> Adds a counter to an iterable and returns it like an array of tuple [(index, value)].
- `clip(array, min, max)` -> limits the values in an array: values smaller than min become min, and values larger than max become max.


## 2022/05/05

1. Calcolare, per ciascuna riga dell’immagine, la somma dei valori dei pixel: sia Ym la coordinata y della riga dell’immagine con la somma minore.

    ```python
    Ym = np.argmin(img.sum(axis=1))
    # or
    Ym = np.argmin(np.sum(img, 1))
    ```

2. Binarizzare img, utilizzando come unica soglia la media dei livelli di grigio dei pixel con coordinata y minore o uguale a Ym.

    ```python
    _, img2 = cv.threshold(img, np.mean(img[:Ym+1,...]), 255, cv.THRESH_BINARY)
    ```

3. Applicare, al risultato del passo precedente, un’operazione morfologica di dilatazione con un cerchio di diametro 3 pixel come elemento strutturante e considerando il foreground pari a 255: sia img3 il risultato.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_ELLIPSE, (3,3))
    img3 = cv.morphologyEx(img2, cv.MORPH_DILATE, st_elem)
    ```

4. Costruire un’immagine contenente solo i bordi (con uno spessore di 2 pixel) delle componenti connesse dell’immagine ottenuta al punto precedente. Suggerimento: questo risultato può essere ottenuto con la differenza fra l’immagine e il risultato dell’erosione con un cerchio di diametro 2*2+1 pixel.
    
    ```python
    img4 = img3 - cv.morphologyEx(img3, cv.MORPH_ERODE, cv.getStructuringElement(cv.MORPH_ELLIPSE, (5,5)))
    ```

5. Determinare tutti i pixel di background di img3 con distanza maggiore di 4 pixel (secondo la metrica d8) dal foreground.
    
    ```python
    img5 = np.where(cv.distanceTransform(255-img3, cv.DIST_C, 3) > 4, 255, 0).astype(np.uint8)
    ```

6. Restituire un’immagine a colori in formato BGR in cui i pixel di bordo individuati al passo 4 sono blu, i pixel individuati al punto 5 verdi e i restanti pixel hanno un valore, nel solo canale R, pari alla metà (arrotondata all’intero inferiore) della corrispondente luminosità in
img.

    ```python
    return cv.merge(img4, img5, (img & ( ~ (img4 | img5))) // 2)
    ```

## 2022/06/08

1. Calcolare, per ogni pixel dell’immagine img, la media dei pixel in un intorno 7x7. Impostare a zero 3 pixel lungo tutti i bordi nell’immagine risultato.

    ```python
    img1 = cv.boxFilter(img, -1, (7,7))
    img1[:3], img[:,:3], img[-3:], img[:,-3:] = 0, 0, 0, 0
    ```

2. Calcolare l’immagine Diff in cui ciascun pixel è il valore assoluto della differenza fra il pixel corrispondente nell’immagine originale e il valore determinato al passo precedente.

    ```python
    Diff = np.abs(img - img1)
    ```

> 3. Eseguire l’operazione di contrast stretching su Diff.

```python
v_min, v_max = Diff.min(), Diff.max()
if v_max > v_min:
    img3 = (255 * (Diff.astype(float) - v_min)/(v_max - v_min)).astype(np.uint8)
```

4. Binarizzare Diff utilizzando come soglia globale il valore 128.

    ```python
    _, img4 = cv.threshold(Diff, 128, 255, cv.THRESH_BINARY)
    ```

5. Etichettare le componenti connesse del risultato ottenuto al passo precedente; eliminare quindi le componenti connesse con area inferiore a 15 pixel.

    ```python
        toRemove = np.argwhere(stats[:, cv.CC_STAT_AREA] < 15)
    ```

6. Restituire come output un’immagine grayscale in cui i pixel appartenenti alle componenti connesse rimanenti dopo il passo precedente hanno valore 255, mentre tutti gli altri 0.

    ```python
    return np.where(np.isin(cc, toRemove), 0, 255).astype(uint8)
    ```

## 2022/07/13

1. Calcolare, per ogni immagine nella lista (esclusa l’ultima), la differenza pixel-a-pixel fra l’immagine seguente e l’immagine stessa, memorizzandola come una matrice numpy di interi a 16 bit con segno. Conservare i riferimenti a tali immagini in una lista chiamata diff_frames.

    ```python
    diff_frames = []
    for i in len(frames)-1:
        diff_frames.append(frames[i+1].astype(np.int16) - frames[i])
    # or
    diff_frames = [frames[i+1].astype(np.int16) - frames[i] for i in len(frames)-1].
    ```

2. Costruire una lista masks di immagini binarie con un byte per pixel, con la stessa lunghezza di diff_frames. In ciascuna immagine i pixel che nella corrispondente immagine in diff_frames hanno un valore assoluto maggiore di 20 devono essere posti a 255, mentre tutti gli altri a zero.

    ```python
    masks = [(np.where(np.abs(diff_frames[i]) > 20, 255, 0)).astype(np.uint8) for i in diff_frames]
    ```

3. Modificare ciascuna immagine in masks come segue: etichettarne le componenti connesse ed eliminare quelle con area inferiore a 20 pixel, ponendo i corrispondenti pixel a zero.

    ```python
    for i in masks:
        n, cc, stats, centroids = cv.connectedComponentsWithStats(masks[i])
        toRemove = np.argwhere(stats[:, cv.CC_STAT_AREA] < 20)
        i[np.isin(cc, toRemove)] = 0
    ```

4. Trovare l’immagine in masks che contiene il maggior numero di pixel a 255.

    ```python
    index, mask = max(enumerate(masks), key = lambda k: np.count_nonzero(k[1]))
    #or
    temp_array = [np.count_nonzero(i) for i in masks]
    index, mask = max(enumerate(temp_array))
    ```

5. Restituire una tupla contenente l’immagine binaria trovata al punto precedente e la corrispondente immagine originale in frames.

    ```python
    return mask, frames[index]
    ```

## 2022/09/08

1. Convertire img in grayscale memorizzando il risultato in un’immagine chiamata gray_img.

    ```python
    gray_img = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
    ```

2. Applicare due filtri gaussiani a gray_img: il primo con dimensione 5×5 e σ=1, il secondo con dimensione 11×11 e σ=4. Siano img_g1 e img_g2, rispettivamente, le due immagini ottenute applicando tali filtri.

    ```python
    img_g1 = cv.GaussianBlur(gray_img, (5,5), 1)
    img_g2 = cv.GaussianBlur(gray_img, (11,11), 4)
    ```

3. Calcolare la differenza pixel-a-pixel sottraendo img_g2 da img_g1, memorizzando il risultato in un’immagine con un intero a 16 bit per ogni pixel (in modo da ottenere sia i valori negativi che quelli positivi), quindi conservare solo i valori positivi (ponendo a zero tutti gli altri), normalizzarli fra 0 e 255 e memorizzare il risultato in un’immagine di byte img_d.

    ```python
    img_d = img_g1.astype(np.int16) - img_g2
    img_d = np.where(img_d >= 0, img_d, 0).astype(np.uint8)
    img_d = normalize(img_d, None, 0, 255, cv.NORM_MINMAX)
    ```

4. Eseguire l’algoritmo di Canny, senza effettuare il primo passo (smooth Gaussiano), ma partendo direttamente dall’immagine img_g2; come soglie per l’isteresi utilizzare 30 e 70.

    ```python
    img_c = cv.Canny(img_g2, 30, 70)
    ```

5. Restituire un’immagine a colori in cui, per ciascun pixel, la componente blu è il valore corrispondente in gray_img, la componente verde è il valore corrispondente ottenuto con l’algoritmo di Canny e la componente rossa è il valore corrispondente in img_d.

    ```python
    return cv.merge(gray_img, img_c, img_d)
    ```

## 2023 Prova esame

1. Convertire img in grayscale, memorizzando il risultato in un’immagine chiamata img_g.

    ```python
    img_g = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
    ```

2. Se la larghezza di img_g è maggiore della sua altezza, ruotarla di 90 gradi in senso antiorario (equivale a scambiare le righe con le colonne).

    ```python
    h, w = img_g.shape[:2]
    if w > h
        img_g = img_g.T
    ```

3. Binarizzare l'immagine utilizzando l'algoritmo di Otsu.

    ```python
    _, img_b = cv.threshold(img_g, -1, 255, cv.THRESH_OTSU)
    ```

4. Applicare al risultato del passo precedente un'operazione morfologica di dilatazione con un elemento strutturante di forma quadrata e lato di 9 pixel.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_RECT, (9,9))
    img_d = cv.morphologyEx(img_b, cv.MORPH_DILATE, st_elem)
    ```

5. Trovare il più esteso gruppo di pixel di foreground contigui nell'immagine ottenuta al passo precedente: eliminare tutti gli altri pixel di foreground.

    ```python
    n, cc, stats, centroids = cv.connectedComponentsWithStats(img_d)
    max_cc = cv.argmax(stats[1:, cv.CC_STAT_AREA]) + 1
    img_d = cv.where(cv.isin(cc, max_cc), 255, 0).astype(cv.uint8)
    ```

6. Restituire una copia di img_g in cui è dimezzata la luminosità dei pixel le cui posizioni sono state individuate al passo precedente.

    ```python
    return cv.where(img_b != 0, img_g // 2, img_g).astype(cv.uint8)
    # or
    return img_g[img_d] = img_g // 2
    ```

## 2013/01/15

1. Calcolare, per ogni pixel di InputImage (esclusi eventualmente quelli di bordo), le componenti x e y del gradiente mediante convoluzione con opportuni filtri.

    ```python
    InputImage = cv.GaussianBlur(img, (3,3), 0)
    dx = cv.Sobel(InputImage, cv.CV_32F, 1, 0, scale=1/8)
    dy = cv.Sobel(InputImage, cv.CV_32F, 0, 1, scale=1/8)
    ```

2. Calcolare, per ogni pixel di InputImage (esclusi eventualmente quelli di bordo), l’orientazione del gradiente, esprimendola come angolo in radianti in [-π,π].

    ```python
    # mod = cv.magnitude(dx, dy)
    ang = cv.phase(dx, dy, angleInRadians=True)
    ```

3. Binarizzare InputImage utilizzando una soglia globale pari al livello medio di grigio dell’immagine.

    ```python
    img_b = cv.threshold(InputImage, np.mean(InputImage), 255, cv.THRESH_BINARY)
    ```

4. Etichettare le componenti connesse dell’immagine binarizzata, utilizzando 255 come valore di foreground.

    ```python
    n, cc, stats, centroids = cv.connectedComponentsWithStats(img_b)
    img_b[cc] = 255
    ```

5. Individuare eventuali componenti connesse che soddisfino la seguente condizione: almeno il 15% dei pixel ha orientazione del gradiente (così come calcolata su InputImage) compresa nell’intervallo (-π/4,π/4).

    ```python

    ```

6. Restituire come output un’immagine grayscale (Result) in cui i pixel appartenenti alle componenti connesse individuate al passo precedente corrispondono a quelli di InputImage, mentre tutti gli altri pixel hanno valore zero.

    ```python
    return np.where(cv.isin(img_c, InputImage), InputImage, 0).astype(np.uint8)
    ```

## 2013/06/26

1. Calcolare, per ogni pixel dell’immagine, la media dei pixel in un intorno 51x51, considerando eventuali pixel fuori dal bordo come aventi livello di grigio pari a zero.

    ```python
    img1 = cv.boxFilter(img, -1, (51,51))
    ```

2. Calcolare l’immagine Diff in cui ciascun pixel è il valore assoluto della differenza fra il pixel corrispondente nell’immagine originale e il valore medio determinato al passo precedente.

    ```python
    Diff = np.abs(img - img1)
    ```

3. Eseguire l’operazione di contrast stretching su Diff.

    ```python
    v_max, v_min = Diff.max(), Diff.min()
    if v_max > v_min:
        str = (255 * (Diff.astype(float) - v_min)/(v_max - v_min)).astype(np.uint8)
    ```

4. Binarizzare Diff utilizzando come soglia globale il valore 128.

    ```python
    img4 = cv.treshold(Diff, 128, 255, cv.THRESH_BINARY)
    ```

5. Etichettare le componenti connesse del risultato ottenuto al passo precedente usando la metrica D4; eliminare poi le componenti connesse con un perimetro superiore a 25 pixel.

    ```python
    n, cc, stats, centroids = cv.connectedComponentsWithStats(img4)
    toRemove = cv.argmax(stats[:, cv.STATS_AREA])
    img5 = cv.where(cv.isin(cc, toRemove), 0, 255).astype(np.uint8)
    ```

6. Restituire come output un’immagine grayscale (Result) in cui i pixel appartenenti alle componenti connesse individuate al passo precedente hanno valore 255, mentre tutti gli altri pixel hanno valore 0.

    ```python
    return img5
    ```

## 2013/09/17

1. Binarizzare InputImage utilizzando come soglia la media dei livelli di grigio dei pixel nell’immagine stessa che sono massimi locali di luminosità (considerando semplicemente l’intorno di 8 pixel di ciascuno). Qualora nessun pixel soddisfacesse tale proprietà, utilizzare il valore 128 come soglia.

    ```python
    if cv.boxFilter(img, -1, (8,8)) is not None
        img1 = cv.treshold(img, , 255, cv.TRESH_BINARY)
    ```

2. Eseguire, sul risultato del passo precedente, un’operazione morfologica di apertura con un quadrato di lato 7 pixel come elemento strutturante: sia C il risultato.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_RECT, (7,7))
    img2 = cv.morphologyEx(img1, cv.MORPH_OPEN, st_elem)
    ```

3. Estrarre i bordi di C utilizzando la morfologia matematica e un cerchio di diametro 3 pixel come elemento strutturante.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_ELLIPSE, (3,3))
    img3 = cv.morphologyEx(img1, cv.MORPH_ERODE, st_elem)
    ```

4. Determinare tutti i pixel di foreground di C con distanza maggiore di 9 pixel (secondo la metrica d8) dal background (valore 0).

    ```python
    img4 = cv.where(cv.distanceTransform(img3, cv.DIST_C, 3) > 9, 255, 0).astype(np.uint8)
    ```

5. Restituire come output un’immagine grayscale (Result) in cui i pixel di bordo determinati al passo 3 hanno luminosità pari a 255, i pixel determinati al punto 4 hanno luminosità pari a 128 e i restanti pixel hanno luminosità pari alla metà della corrispondente luminosità in InputImage.

    ```python
    img[img3], img[img4] = 255, 128
    return ((img & (~(img3 | img4))) // 2)
    ```

## 2014/01/29

1. Calcolare, per ciascuna riga dell’immagine, la somma dei valori dei pixel: sia Ym la coordinata y della riga dell’immagine con la somma maggiore (nel caso più righe soddisfino tale condizione, considerare quella con coordinata y minore).

    ```python
    Ym = np.argmax(np.sum(img, 1))
    ```

2. Binarizzare InputImage utilizzando come soglia la media dei livelli di grigio dei pixel con coordinata y maggiore o uguale a Ym.

    ```python
    img2 = cv.threshold(img, np.mean(img[Ym:,:]), 255, cv.THRESH_BINARY)
    ```

3. Eseguire, sul risultato del passo precedente, un’operazione morfologica di erosione con un cerchio di diametro 7 pixel come elemento strutturante e considerando il foreground pari a 255: sia C il risultato.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_ELLIPSE, (7,7))
    C = cv.morphologyEx(img1, cv.MORPH_ERODE, st_elem)
    ```

4. Estrarre i bordi di C utilizzando la morfologia matematica e un cerchio di diametro 3 pixel come elemento strutturante.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_ELLIPSE, (3,3))
    C = cv.morphologyEx(img1, cv.MORPH_DILATE, st_elem)
    ```

5. Determinare tutti i pixel di background di C con distanza minore di 9 pixel (secondo la metrica d8) dal foreground (valore 255).

    ```python
    img5 = cv.where(cv.distanceTransform(255-C, cv.DIST_C, 3) < 9, 255, 0).astype(np.uint8)
    ```

6. Restituire come output un’immagine di int (Result) in cui i pixel di bordo individuati al passo 4 hanno valore pari a -1, i pixel individuati al punto 5 hanno valore pari a 0 e i restanti pixel hanno valore pari al quadrato della corrispondente luminosità in img.

    ```python
    img[C], img[img5] = -1, 0
    return ((img & (~(C | img5)))^2)
    ```

## 2017/06/13

1. Binarizzare InputImage utilizzando come soglia la media dei livelli di grigio dei pixel nell’immagine stessa che sono minimi locali di luminosità (considerando semplicemente l’intorno di 8 pixel di ciascuno). Qualora nessun pixel soddisfacesse tale proprietà, utilizzare il valore 32 come soglia.

    ```python
    img1 = cv.treshold(InputImage, np.mean(), 255, cv.THRESH_BINARY)
    ```

2. Eseguire, sul risultato del passo precedente, un’operazione morfologica di apertura con un cerchio di diametro 3 pixel come elemento strutturante: sia A il risultato.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_ELLIPSE, (3,3))
    A = cv.morphologyEx(img1, cv.MORPH_OPEN, st_elem)
    ```

3. Eseguire, sul risultato del passo 2, un’operazione morfologica di chiusura con un cerchio di diametro 5 pixel come elemento strutturante: sia C il risultato.

    ```python
    st_elem = cv.getStructuringElement(cv.MORPH_ELLIPSE, (5,5))
    C = cv.morphologyEx(img1, cv.MORPH_CLOSE, st_elem)
    ```

4. Determinare l’intersezione fra tutti i pixel di foreground di A con distanza maggiore di 2 pixel (secondo la metrica d8) dal background e tutti i pixel di foreground di C con distanza maggiore di 5 pixel (sempre secondo d8) dal background.

    ```python
    imgA = cv.where(cv.distanceTransform(A, cv.DIST_C, 3) > 2, 255, 0).astype(np.uint8)
    imgC = cv.where(cv.distanceTransform(C, cv.DIST_C, 3) > 5, 255, 0).astype(np.uint8)
    img4 = np.where(imgA | imgC, 255, 0).astype(np.uint8)
    ```

5. Restituire come output un’immagine grayscale (Result) in cui i pixel determinati al punto 4 hanno luminosità pari a 128 e i restanti pixel hanno luminosità pari a un quarto della corrispondente luminosità in InputImage.

    ```python
    InputImage[img4] = 128
    return (InputImage & (~(img4)) // 4)
    ```

