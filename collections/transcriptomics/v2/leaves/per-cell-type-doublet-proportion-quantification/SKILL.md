---
name: per-cell-type-doublet-proportion-quantification
description: Use when Chord (or another doublet caller) has output predicted doublet
  barcodes for annotated scRNA-seq samples and you need per-sample doublet rates and
  per-cell-type doublet proportions to identify doublet-enriched populations.
when_to_use_negative:
- When the dataset has experimental doublet labels (e.g., cell hashing, MULTI-seq,
  or demuxlet calls) — direct experimental annotation is preferable to summarizing
  computational Chord calls.
- When cell-type annotations are absent or untrusted — cross-tabulating doublets against
  unvalidated annotations yields misleading per-type proportions.
- When the goal is doublet detection itself rather than downstream summarization —
  use the chord() detection workflow instead.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: Ensemble doublet detector whose 'outname_doublets.csv' barcode calls are the
    input to the per-cell-type cross-tabulation
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Built-in method whose scores feed Chord's GBM integration before doublets
    are called and summarized
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Built-in method (cxds/bcds) whose scores feed Chord's overkill/training pipeline
    before summarization
  repo: https://github.com/kostkalab/scds
- name: R
  role: Environment in which chord(), creatSCE(), and the cross-tabulation of doublets
    vs cell types are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:31ec390cd4e1f9cf4e35405f6bc9a720d183e8e1a0edebe801cfdf7a63ea93e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# per-cell-type-doublet-proportion-quantification

## Summary

Cross-tabulates Chord-predicted doublet barcodes against cell-type annotations and per-sample totals to quantify doublet burden per cell type and per sample, revealing which populations (e.g., myeloid vs fibroblasts) are enriched for doublets in real-world scRNA-seq data.

## When to use

Use when Chord (or another doublet caller) has output predicted doublet barcodes for annotated scRNA-seq samples and you need per-sample doublet rates and per-cell-type doublet proportions to identify doublet-enriched populations.

## When NOT to use

- When the dataset has experimental doublet labels (e.g., cell hashing, MULTI-seq, or demuxlet calls) — direct experimental annotation is preferable to summarizing computational Chord calls.
- When cell-type annotations are absent or untrusted — cross-tabulating doublets against unvalidated annotations yields misleading per-type proportions.
- When the goal is doublet detection itself rather than downstream summarization — use the chord() detection workflow instead.

## Inputs

- Chord output file 'outname_doublets.csv' containing predicted doublet barcodes
- Seurat objects with cell-type annotations for each sample (e.g., NSCLC tumour samples 11, 13, 17, 18, 22)
- Per-sample cell counts (e.g., 24,280 cells across the 5 NSCLC samples) and the estimated doublet rate (~0.9% per 1,000 cells)

## Outputs

- Per-sample doublet rates (e.g., 3.81–4.74% in the NSCLC reference application)
- Per-cell-type doublet proportion table (e.g., myeloid 8.15%, fibroblasts 1.69%)
- Report of doublet-enriched clusters (e.g., cluster-10 doublet enrichment)

## How to apply

After running chord() on each sample's Seurat object (with doubletrate set by the 10X rule of ~0.9% per 1,000 cells and overkill=T so scds/DoubletFinder scores are integrated via the GBM algorithm), read the predicted doublet barcodes from the output file 'outname_doublets.csv'. Cross-tabulate these called doublets against the existing cell-type annotations to obtain per-cell-type doublet proportions, and compute per-sample doublet rates as called doublets divided by total cells. In the NSCLC reference application (samples 11, 13, 17, 18, 22), per-sample rates of 3.81%, 4.68%, 4.74%, 4.20%, and 4.42% were estimated, with myeloid cells showing the highest doublet proportion (8.15%) and fibroblasts the lowest (1.69%); cluster-10 doublet enrichment was also reported. Interpret cell-type-specific proportions cautiously: apparent enrichment may reflect doublet-prone states rather than misannotation.

## Related tools

- **Chord** (Ensemble doublet detector whose 'outname_doublets.csv' barcode calls are the input to the per-cell-type cross-tabulation) — https://github.com/13308204545/Chord
- **DoubletFinder** (Built-in method whose scores feed Chord's GBM integration before doublets are called and summarized) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Built-in method (cxds/bcds) whose scores feed Chord's overkill/training pipeline before summarization) — https://github.com/kostkalab/scds
- **R** (Environment in which chord(), creatSCE(), and the cross-tabulation of doublets vs cell types are executed)

## Examples

```
chord(seu="input seurat object",doubletrate=0.009,overkill=T,outname="sample11"); doublets <- read.csv("sample11_doublets.csv"); table(celltype[doublets$barcodes])/table(celltype)
```

## Evaluation signals

- Per-cell-type doublet table reproduces the reported reference values (myeloid 8.15%, fibroblasts 1.69%) on the NSCLC dataset.
- Per-sample doublet rates fall within the reported 3.81–4.74% range for samples 11, 13, 17, 18, and 22.
- 'outname_doublets.csv' exists and contains a barcode vector consistent with the estimated doubletrate (~0.9% per 1,000 cells scaled to sample size).
- Sum of per-cell-type doublet counts equals the total number of called doublets per sample (cross-tabulation consistency check).
- Cluster-level enrichment (e.g., cluster-10) is reproducibly identified when doublets are projected onto the clustering.

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods, so per-sample summarization is gated on a slow ensemble run.
- Called doublets reflect computational estimates at an assumed doubletrate; no gold-standard labels exist for real-world unlabeled data, so per-type proportions are approximate.
- Ambient mRNA removal and other pre-processing steps are not incorporated by Chord and could affect doublet calls and thus the derived proportions.

## Evidence

- [readme] The doublets' barcodes are in the file "outname_doublets.csv": "A:The doublets' barcodes are in the file "outname_doublets.csv""
- [readme] 10X doublet rate rule of thumb: "10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [results] Per-sample and per-cell-type doublet summary from the NSCLC application: [evidence span withheld — anchor status: not_found]
- [methods] Cross-tabulation step of doublets against cell types: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime limitation of the ensemble approach: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
