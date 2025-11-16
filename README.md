# Discovering Rare Genomic Subtypes from RNA-seq via Unsupervised Autoencoder Embeddings and Cluster Stability Analysis

This repository contains the code and notebooks for the project:

> **Discovering Rare Genomic Subtypes from RNA-seq via Unsupervised Autoencoder Embeddings and Cluster Stability Analysis**
>  *Mezghiche Alaa*, 2025. 

The goal is to use unsupervised deep learning on RNA-seq data to:
1. Learn low-dimensional representations with an autoencoder,
2. Cluster patients in the latent space,
3. Identify **rare but stable** molecular subtypes within a cancer type,
4. Characterize these subtypes via differential expression.

The main case study is **clear cell renal cell carcinoma (KIRC)** using the UCI *Gene Expression Cancer RNA-Seq* dataset.

---

##  Project overview

### Data

- **Dataset:** UCI Machine Learning Repository – *Gene Expression Cancer RNA-Seq* (ID 401).  
- **Samples:** 801 tumors  
- **Genes:** 20,531  
- **Cancer types (Class label):**
  - BRCA – Breast invasive carcinoma  
  - COAD – Colon adenocarcinoma  
  - KIRC – Kidney renal clear cell carcinoma  
  - LUAD – Lung adenocarcinoma  
  - PRAD – Prostate adenocarcinoma  

In the repo, the data is expected in a simple CSV format, e.g.:

- `data.csv` – gene expression matrix (samples × genes)  
- `labels.csv` – sample IDs and cancer type labels  

(If your filenames are different, update the notebooks accordingly.)

---

### Methods (high level)

1. **Pan-cancer negative control**
   - Use all 801 samples.
   - Log-transform, select top 2,000 highly variable genes, z-score.
   - Train a feed-forward autoencoder and cluster the latent space with KMeans.
   - Observe that clusters align almost perfectly with tissue-of-origin  
     → Cramér’s V ≈ 0.887.
   - Conclusion: pan-cancer clustering mainly rediscovers tissue, not novel subtypes.

2. **Within-cancer analysis (KIRC only)**
   - Subset to KIRC samples (n = 146).
   - Select top 2,000 highly variable genes within KIRC, z-score.
   - Train an autoencoder with a 128-dimensional latent space.
   - Run KMeans for k = 2…10 on the latent codes.
   - Compute:
     - Silhouette score,
     - Davies–Bouldin index (DBI),
     - Cluster stability (Jaccard index across 20 seeds, after Hungarian alignment).

3. **Rare + stable subtype discovery**
   - Define a pre-specified “discovery rule”:
     - **Rare:** prevalence < 10% of samples  
     - **Stable:** Jaccard ≥ 0.60 across 20 KMeans runs  
   - Scan all k ∈ {2,…,10} and all clusters.
   - Identify a simple and interpretable solution at **k = 5**:
     - Silhouette = 0.129  
     - DBI = 2.045  
     - Cluster sizes: [10, 19, 31, 53, 33]  
     - Rare cluster **C0:** 10 / 146 = 6.85% of KIRC patients  
     - Jaccard(C0) = 0.787 (highly stable)

4. **Interpretability: differential expression**
   - Cluster-vs-rest analysis on standardized expression:
     - Welch’s t-test,
     - Benjamini–Hochberg FDR correction.
   - Identify a small set of marker genes for C0, e.g.:
     - Strongly **down-regulated**: `gene_11713`, `gene_13678`, `gene_16402`, `gene_3777`, …  
     - Strongly **up-regulated**: `gene_751`, `gene_17397`, `gene_2760`, …
   - Visualizations:
     - UMAP of latent space with C0 highlighted,
     - Volcano plot,
     - Heatmap of top up-/down-regulated markers.

