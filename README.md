# Image Segmentation and Puzzle Matching in Computer Vision

Progetto accademico per il corso di **Computer Vision 2024–25** dedicato all'estrazione, segmentazione e ricomposizione automatica di pezzi di puzzle su un'immagine di riferimento (*City Example*).

---

## Panoramica del Progetto

Il progetto implementa una pipeline completa di computer vision per risolvere e ricomporre un puzzle a partire da immagini dei singoli pezzi e dall'immagine target originale:

1. **Pre-processing:** Ottimizzazione e normalizzazione della luminosità nello spazio colore HSV.
2. **Segmentazione dei singoli pezzi:** Clustering dei colori tramite **K-Means**, generazione di maschere binarie, raffinamento morfologico (operazioni di *Canny edge detection*, *dilation*, *erosion*) ed estrazione dei componenti connessi.
3. **Feature Detection & Description:** Estrazione di punti chiave e descrittori invarianti mediante **SIFT** (*Scale-Invariant Feature Transform*).
4. **Feature Matching & Filtering:** Corrispondenza con **BFMatcher** (L2-norm + cross-check) e filtraggio euristico con *ratio thresholding*.
5. **Stima Geometrica & Ricostruzione:** Calcolo della trasformazione affine parziale (**RANSAC**) per ogni pezzo e rendering finale tramite *affine warping* sul canvas di riferimento.

---

## Pipeline Dettagliata

### 1. Pre-processing & Miglioramento della Luminosità
* Conversione delle immagini dal formato standard OpenCV BGR a RGB.
* Aumento controllato del canale $V$ (Value) nello spazio colore HSV per enfatizzare i contrasti e normalizzare l'illuminazione tra il riferimento e i frammenti.

### 2. Segmentazione basata su K-Means e Morfologia
* **Downsampling:** Ridimensionamento delle immagini ad alta risoluzione per velocizzare il clustering.
* **K-Means Clustering ($k = 4$):** Raggruppamento dei vettori colore dei pixel e selezione mirata dei cluster rappresentanti i pezzi del puzzle (filtrando lo sfondo).
* **Raffinamento dei contorni:**
  * Rilevamento dei bordi con **Canny Edge Detector**.
  * Chiusura dei contorni tramite **Dilation** (kernel rettangolare $10 \times 10$).
  * Separazione dei pezzi adiacenti mediante **Erosion**.
* **Component Extraction:** Riconoscimento dei singoli pezzi tramite `connectedComponentsWithStats` e filtraggio basato sull'area minima/massima del pezzo.

### 3. Feature Extraction & Matching (SIFT)
* Individuazione di keypoints e calcolo dei descrittori sia sull'immagine target che su ciascun pezzo segmentato.
* Matching robusto tramite **Brute-Force Matcher** (`NORM_L2`, `crossCheck=True`).
* Eliminazione delle corrispondenze a distanza nulla e applicazione di una soglia di distanza relativa:
  $$d_{\text{match}} < \text{ratio} \cdot \min(d)$$

### 4. Trasformazione Affine e Ricostruzione
* Selezione di almeno 3 punti corrispondenti per componente.
* Stima della trasformazione affine euclidea (rototraslazione e scala uniforme) tramite `cv.estimateAffinePartial2D` con filtro **RANSAC**.
* Deformazione (`cv.warpAffine`) e fusione sul canvas nero finale per ricostruire la scena completa.

---

## Struttura del Repository

```plaintext
├── data/
│   ├── city.JPG          # Immagine di riferimento intera
│   ├── pieces1.JPG       # Scatti dei frammenti (set 1)
│   ├── pieces2.JPG       # Scatti dei frammenti (set 2)
│   ├── pieces3.JPG       # Scatti dei frammenti (set 3)
│   └── pz.jpg
├── notebooks/
│   └── final_project.ipynb  # Notebook Jupyter con l'intero flusso di esecuzione
├── requirements.txt      # Dipendenze Python
└── README.md             # Documentazione del progetto
```

