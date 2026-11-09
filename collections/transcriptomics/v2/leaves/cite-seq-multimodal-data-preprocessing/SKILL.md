---
name: cite-seq-multimodal-data-preprocessing
description: Use when raw CITE-seq data (paired scRNA-seq and ADT counts from platforms
  like 10X Chromium) must be filtered, normalized, and packaged for joint multimodal
  clustering benchmarking.
when_to_use_negative:
- Data are single-omics only (scRNA-seq without ADT) — use a single-modal preprocessing
  pipeline instead
- Input is already normalized h5/aecessible object prepared for scMDC; re-normalizing
  would double-process the counts
- Analyzing SMAGE-seq (scRNA+scATAC) data, which requires Signac GeneActivity mapping
  of scATAC reads to gene regions before preprocessing rather than direct mRNA/ADT
  packaging
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scMDC
  role: Downstream multimodal deep clustering model consuming the preprocessed h5
    files; defines the required input schema (X1/X2/Y/Batch)
- name: Scanpy
  role: Python package used to filter, size-factor normalize, log-transform, and scale
    mRNA and ADT data separately with the same procedure
- name: SeuratData / Seurat
  role: Source of the BMNC (bmcite, GSE128639) dataset with cell-type labels used
    in preprocessing
  repo: https://github.com/satijalab/seurat
- name: BREM-SC
  role: Competing joint-clustering method that also works directly on raw CITE-seq
    counts; benchmark dataset files must be consumable by it
  repo: https://github.com/tarot0410/BREMSC
- name: CiteFuse
  role: Competing R/Bioconductor method for pre-processing and clustering CITE-seq
    data; shares the same paired mRNA/ADT input structure
  repo: https://github.com/SydneyBioX/CiteFuse
- name: Specter
  role: Competing multimodal spectral clustering method; consumes mRNA/ADT matrices
    preprocessed via Seurat (NormalizeData, CLR for ADT)
  repo: https://github.com/canzarlab/Specter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:a4644dadd641cf7705788447e1105f3fd98ab2f61a8744ff0f3f7fc3e5892324
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat
  tools:
  - BREM-SC
  - CiteFuse
  - IDEC
  - SC3
  - SCVIS
  - SKlearn
  - Scanpy
  - Seurat V4
  - SeuratData / Seurat
  - Specter
  - TotalVI
  - Tscan
  - scMDC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cite-seq-multimodal-data-preprocessing

## Summary

Preprocess paired CITE-seq mRNA and ADT count matrices (e.g., into h5 files with X1, X2, ground-truth Y, and Batch indicator) so they can serve as inputs to multimodal clustering methods such as scMDC, using Scanpy-based normalization applied separately to each modality.

## When to use

Use when raw CITE-seq data (paired scRNA-seq and ADT counts from platforms like 10X Chromium) must be filtered, normalized, and packaged for joint multimodal clustering benchmarking.

## When NOT to use

- Data are single-omics only (scRNA-seq without ADT) — use a single-modal preprocessing pipeline instead
- Input is already normalized h5/aecessible object prepared for scMDC; re-normalizing would double-process the counts
- Analyzing SMAGE-seq (scRNA+scATAC) data, which requires Signac GeneActivity mapping of scATAC reads to gene regions before preprocessing rather than direct mRNA/ADT packaging

## Inputs

- Raw paired CITE-seq mRNA count matrix (cells x genes) from GEO or 10X sources (e.g., GSE100866, GSE128639, GSE150599 SLN208/SLN111, 10X PBMC)
- Raw ADT (antibody-derived tag) UMI count matrix (cells x proteins)
- Ground-truth cell-type label vector Y (with 'Unknown' cells flagged for removal)
- Batch indicator for multi-batch datasets

## Outputs

- h5 file containing X1 (normalized mRNA), X2 (normalized ADT), Y (cell-type labels), and Batch indicator
- Filtered gene and ADT feature lists (zero-count features removed)

## How to apply

Prepare each dataset in h5 format with slots X1 (mRNA), X2 (ADT), Y (cell-type labels), and a Batch indicator, following the 'dataset' folder readme of scMDC. Filter out genes and ADTs with zero counts across cells, and remove cells with 'Unknown' labels (e.g., in GSE100866). Normalize mRNA and ADT data separately but with the same Scanpy-based procedure (filter zero-count genes, size-factor normalization, log transform, scaling), since the two modalities have different count scales but the paper's ZINB modeling requires comparable preprocessing. Benchmark datasets include seven single-batch CITE-seq sets (GSE100866 CBMN, GSE128639 BMNC/bmcite from SeuratData, GSE150599 SLN208 and SLN111, 10X PBMC) and two multi-batch sets, where the Batch field enables scMDC's conditional autoencoder to correct batch effects.

## Related tools

- **scMDC** (Downstream multimodal deep clustering model consuming the preprocessed h5 files; defines the required input schema (X1/X2/Y/Batch))
- **Scanpy** (Python package used to filter, size-factor normalize, log-transform, and scale mRNA and ADT data separately with the same procedure)
- **SeuratData / Seurat** (Source of the BMNC (bmcite, GSE128639) dataset with cell-type labels used in preprocessing) — https://github.com/satijalab/seurat
- **BREM-SC** (Competing joint-clustering method that also works directly on raw CITE-seq counts; benchmark dataset files must be consumable by it) — https://github.com/tarot0410/BREMSC
- **CiteFuse** (Competing R/Bioconductor method for pre-processing and clustering CITE-seq data; shares the same paired mRNA/ADT input structure) — https://github.com/SydneyBioX/CiteFuse
- **Specter** (Competing multimodal spectral clustering method; consumes mRNA/ADT matrices preprocessed via Seurat (NormalizeData, CLR for ADT)) — https://github.com/canzarlab/Specter

## Examples

```
python run_scMDC.py --data_file dataset/bmcite.h5 --X1 X1 --X2 X2 --Y Y --encodeLayer [256,64,32,16] --pretrain_epochs 400 --gamma 0.1 --tol 0.001 --sigma1 2.5 --sigma2 1.5 --phi1 0.001 --phi2 0.001
```

## Evaluation signals

- h5 file contains all four required slots (X1, X2, Y, Batch) with matching cell counts across X1 and X2
- No genes or ADTs with zero total counts remain after filtering
- mRNA and ADT matrices are normalized separately (not jointly), and distributions are consistent with Scanpy size-factor normalization + log transform + scaling
- Cells labeled 'Unknown' (e.g., in GSE100866) are absent from the final object
- Downstream scMDC run converges under CITE-Seq defaults (encodeLayer [256,64,32,16], pretrain_epochs 400, tol 0.001), indicating the input schema and scale are correct

## Limitations

- The pipeline is CITE-seq specific; SMAGE-seq data first require mapping scATAC-seq reads to gene regions with Signac GeneActivity before this preprocessing applies
- The paper does not report explicit feature-selection defaults for Genes/ADT (scMDC --filter1/--filter2 default to No), so datasets where feature selection matters may behave differently
- Filtering of 'Unknown' cell types assumes pre-existing ground-truth labels; fully unsupervised datasets cannot use this cleaning step

## Evidence

- [discussion] Preprocessing and normalization of mRNA and ADT data with SCANPY: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [discussion] Filter zero-count features: "Specifically, the genes and ADTs with no count are filtered out."
- [discussion] Filter Unknown cells in GSE100866: [evidence span withheld — anchor status: not_found]
- [methods] h5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [intro] Batch-effect correction motivation for multi-batch inputs: "by employing a conditional autoencoder framework, scMDC can correct batch effects when analyzing multi-batch data."
- [methods] Feature selection flags: "--filter1: if do feature selection on Genes. Default: No."
