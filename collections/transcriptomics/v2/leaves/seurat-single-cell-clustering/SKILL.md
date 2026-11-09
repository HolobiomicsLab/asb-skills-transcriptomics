---
name: seurat-single-cell-clustering
description: Use when you have two adjusted gene-barcode UMI matrices from paired
  single-cell processing pipelines and need to compare their cell-clustering outcomes
  quantitatively via ARI.
when_to_use_negative:
- The GBMs have not been adjusted to matching barcodes/genes (i.e. one matrix contains
  barcodes or genes absent from the other) — clustering will not be comparable
- The data are not UMI-count gene-barcode matrices from single-cell RNA-seq processing
  (e.g. bulk RNA-seq counts or full-length reads without UMIs)
- You need to compare processing pipelines for 5' scRNA-seq technologies other than
  Chromium, which UniverSC does not support
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: R package (version 4.1.1) used to perform clustering analysis on the adjusted
    gene-barcode matrices
- name: clues
  role: R package (version 0.6.2.2) used with stats to compute the Adjusted Rand Index
    between two clustering outcomes
- name: R
  role: Runtime environment (version 4.1.2) in which the clustering and ARI computation
    are executed
- name: UniverSC
  role: Produces one GBM of each comparison pair; the other GBM comes from the platform-specific
    counterpart pipeline
  repo: https://github.com/minoda-lab/universc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:03052886c31430785543e0fefed1e12cc6a363ff2ac7dc683e276495c26346d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - R
  - Seurat
  - UniverSC
  - clues
  - stats
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# seurat-single-cell-clustering

## Summary

Clustering analysis of gene-barcode matrices (GBMs) with Seurat, followed by computation of the Adjusted Rand Index (ARI) between two clustering outcomes, as used to benchmark UniverSC against platform-specific pipelines (Cell Ranger, dropSeqPipe, CogentAP, zUMIs).

## When to use

Use when you have two adjusted gene-barcode UMI matrices from paired single-cell processing pipelines and need to compare their cell-clustering outcomes quantitatively via ARI.

## When NOT to use

- The GBMs have not been adjusted to matching barcodes/genes (i.e. one matrix contains barcodes or genes absent from the other) — clustering will not be comparable
- The data are not UMI-count gene-barcode matrices from single-cell RNA-seq processing (e.g. bulk RNA-seq counts or full-length reads without UMIs)
- You need to compare processing pipelines for 5' scRNA-seq technologies other than Chromium, which UniverSC does not support

## Inputs

- Adjusted gene-barcode matrix (GBM) from pipeline A (barcodes subset to those shared with pipeline B, missing genes padded with 0 UMIs)
- Adjusted gene-barcode matrix (GBM) from pipeline B, matched to pipeline A
- Platform identifier for each pair (Chromium, Drop-seq, ICELL8, SmartSeq3)

## Outputs

- Clustering assignments (cluster label per barcode) for each GBM from Seurat
- Adjusted Rand Index value per GBM pair
- Pearson correlation per GBM pair (supporting metric)
- Results table (ari_results.csv) with one ARI value per platform comparison

## How to apply

First adjust each GBM pair so they share matching sets of barcodes and genes: keep only barcodes found in both GBMs and pad genes missing from one matrix with 0 UMIs. Load each adjusted GBM into R (version 4.1.2) and run clustering with the Seurat package (version 4.1.1) separately for each matrix in the pair. Compute the Adjusted Rand Index between the two clustering outcomes using the R packages stats (4.1.2) and clues (0.6.2.2). Optionally compute the Pearson correlation between each GBM pair, treating each gene-by-cell UMI-count pair as one data point unless both counts are zero. Assemble per-platform ARI values (Chromium, Drop-seq, ICELL8, SmartSeq3) with supporting Pearson correlations into a single results table (e.g. ari_results.csv).

## Related tools

- **Seurat** (R package (version 4.1.1) used to perform clustering analysis on the adjusted gene-barcode matrices)
- **clues** (R package (version 0.6.2.2) used with stats to compute the Adjusted Rand Index between two clustering outcomes)
- **R** (Runtime environment (version 4.1.2) in which the clustering and ARI computation are executed)
- **UniverSC** (Produces one GBM of each comparison pair; the other GBM comes from the platform-specific counterpart pipeline) — https://github.com/minoda-lab/universc

## Examples

```
library(Seurat); library(clues); seu <- CreateSeuratObject(counts = gbm); seu <- NormalizeData(seu); seu <- FindClusters(seu); ari <- adj.rand.index(clusters_a, clusters_b)
```

## Evaluation signals

- ARI for the UniverSC/Chromium (Cell Ranger) pair should be approximately 1.0
- ARI for the UniverSC/Drop-seq (dropSeqPipe) pair should be approximately 0.78
- ARI for the UniverSC/ICELL8 (CogentAP) pair should be approximately 0.87
- ARI for the UniverSC/SmartSeq3 (zUMIs) pair should be approximately 0.78
- Both GBMs in each pair must have identical barcode and gene sets before clustering (barcodes kept only if present in both; missing genes padded with 0 UMIs)

## Limitations

- Seurat clustering outcomes depend on the adjusted GBM alignment step; unadjusted matrices give non-comparable clusterings
- ARI values are dataset- and downsampling-dependent (e.g. the ICELL8 dataset was downsampled to 250 K reads), so exact reproduction requires the same test datasets and chromosome 21 (GRCh38/hg38) reference
- Later Cell Ranger versions than 3.0.2 (the version UniverSC uses) are available; version differences may shift results slightly

## Evidence

- [methods] The adjusted pair of GBMs was then used to carry out clustering analysis with an R package Seurat: "The adjusted pair of GBMs was then used to carry out clustering analysis with an R package Seurat"
- [methods] Pearson correlation and ARI computation: "the Pearson correlation between the GBMs and ARI between the two clustering outcomes were calculated"
- [methods] adjusting GBM pairs: "only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [other] Validation against reported benchmark ARI values: [evidence span withheld — anchor status: not_found]
