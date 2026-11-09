---
name: citeseq-rna-adt-count-data-preparation
description: Use when raw paired CITE-seq (or SMAGE-seq) mRNA + ADT count matrices
  must be converted into the structured H5 input (X1 = RNA, X2 = ADT) that scMDC expects
  before deep clustering.
when_to_use_negative:
- Input is already an scMDC-format H5 file with X1/X2 slots populated — re-preprocessing
  would double-normalize the data
- The data are single-omics (RNA only, no paired ADT modality) or unpaired multi-omics
  where cell-by-cell correspondence between modalities is absent
- Cells with 'Unknown' cell type labels are needed for the analysis — these are filtered
  out in the paper's supervised-accuracy experiments, so a different preparation path
  is required
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCANPY
  role: Preprocesses and normalizes raw CITE-seq mRNA and ADT data (filter zero-count
    features, size-factor normalization, log transform, scaling) before H5 export
- name: scMDC
  role: Consumes the prepared H5 input (X1 RNA, X2 ADT); provides the dataset-folder
    readme specifying the required H5 structure and --filter1/--filter2 feature-selection
    flags
  repo: https://github.com/xianglin226/scMDC
- name: Signac
  role: Maps scATAC-seq reads to gene regions (GeneActivity) to build the gene-activity
    matrix needed for SMAGE-seq preparation
- name: h5py
  role: Writes the paired modalities into the H5 file structure required by scMDC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:7caff6b6b0b07ff19b8c2e4cedc2fb103f8efd9ae56c142264c76429fd261016
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - SCANPY
  - Scanpy
  - Signac
  - SymSim
  - h5py
  - scMDC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# citeseq-rna-adt-count-data-preparation

## Summary

Prepare paired CITE-seq mRNA and ADT count matrices into the scMDC-required H5 input format, including SCANPY-based normalization of each modality and removal of zero-count genes/ADTs.

## When to use

Use when raw paired CITE-seq (or SMAGE-seq) mRNA + ADT count matrices must be converted into the structured H5 input (X1 = RNA, X2 = ADT) that scMDC expects before deep clustering.

## When NOT to use

- Input is already an scMDC-format H5 file with X1/X2 slots populated — re-preprocessing would double-normalize the data
- The data are single-omics (RNA only, no paired ADT modality) or unpaired multi-omics where cell-by-cell correspondence between modalities is absent
- Cells with 'Unknown' cell type labels are needed for the analysis — these are filtered out in the paper's supervised-accuracy experiments, so a different preparation path is required

## Inputs

- Raw CITE-seq mRNA count matrix (cells x genes) for a paired single-cell experiment (e.g. GSE100866 CBMN, GSE128639 BMNC, 10X PBMC)
- Raw ADT (antibody-derived tag) UMI count matrix (cells x ADTs) for the same cells
- For SMAGE-seq: raw scATAC-seq fragment files for gene-activity computation via Signac GeneActivity

## Outputs

- H5-format dataset file with scMDC-required structure: X1 (RNA) and X2 (ADT), per the readme in the scMDC 'dataset' folder
- SCANPY-normalized (size-factor normalized, log-transformed, scaled) mRNA and ADT matrices with zero-count genes/ADTs filtered out

## How to apply

Start from raw count matrices for mRNA and ADT measured on the same set of cells, and preprocess each modality separately but with the same SCANPY-based procedure: filter out genes and ADTs with no count, apply size-factor normalization, log transform, and scaling. For SMAGE-seq data, first map scATAC-seq reads to gene regions with Signac GeneActivity to obtain the gene-activity count matrix before the same preprocessing. Export the normalized modalities into H5 format following the readme in the scMDC 'dataset' folder, with X1 holding RNA and X2 holding ADT. Optionally apply scMDC's built-in feature-selection flags (--filter1 for Genes, --filter2 for ATAC; default off) if feature selection is desired. Correctly prepared inputs let the ZINB-reconstruction autoencoder model zero inflation and overdispersion in the count data.

## Related tools

- **SCANPY** (Preprocesses and normalizes raw CITE-seq mRNA and ADT data (filter zero-count features, size-factor normalization, log transform, scaling) before H5 export)
- **scMDC** (Consumes the prepared H5 input (X1 RNA, X2 ADT); provides the dataset-folder readme specifying the required H5 structure and --filter1/--filter2 feature-selection flags) — https://github.com/xianglin226/scMDC
- **Signac** (Maps scATAC-seq reads to gene regions (GeneActivity) to build the gene-activity matrix needed for SMAGE-seq preparation)
- **h5py** (Writes the paired modalities into the H5 file structure required by scMDC)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('raw.h5ad'); sc.pp.filter_genes(adata, min_cells=1); sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.scale(adata)
```

## Evaluation signals

- The output H5 file contains datasets X1 (RNA) and X2 (ADT) with matching cell dimensionality, matching the structure documented in the scMDC 'dataset' folder readme
- No genes or ADTs with all-zero counts remain in either modality after filtering
- mRNA and ADT are normalized separately but with the same SCANPY procedure (size-factor normalization, log transform, scaling)
- scMDC trains without shape errors on the prepared file and the ZINB reconstruction loss decreases during the 400-epoch pretraining (reconstruction for first 200 epochs, KL loss added for remaining 200)
- For SMAGE-seq inputs, gene-activity counts were derived via Signac GeneActivity after ATAC filtering by peak region fragments, nucleosome signal, and TSS enrichment

## Limitations

- The number of clusters K is not estimated automatically; users must specify it after pretraining
- All scMDC experiments in the study were run on an Nvidia Tesla P100 (16 G) GPU, so CPU-only preparation-plus-clustering may not be reproducible
- scATAC-seq-derived modalities require quality filtering (peak region fragments, nucleosome signal, TSS enrichment) before preprocessing; skipping it degrades the gene-activity matrix

## Evidence

- [discussion] The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method.: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [discussion] Specifically, the genes and ADTs with no count are filtered out.: "Specifically, the genes and ADTs with no count are filtered out."
- [methods] 1) Prepare the input data in h5 format. (See readme in 'dataset' folder): "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28"
- [discussion] after filtering the reads by ATAC peak region fragments, nucleosome signal, and TSS enrichment: "after filtering the reads by ATAC peak region fragments, nucleosome signal, and TSS enrichment"
