---
name: single-cell-multiomics-count-preprocessing
description: Use when raw or minimally processed CITE-seq or SMAGE-seq h5 count matrices
  need per-modality normalization (mRNA vs. ADT/ATAC) before feeding a deep clustering
  model such as scMDC.
when_to_use_negative:
- The downstream method expects already-scaled, variable-feature-selected inputs (e.g.,
  a Seurat WNN workflow) rather than the size-factor/log-transformed matrices scMDC
  expects
- Input is a joint embedding or latent representation rather than raw per-modality
  count data
- The modality combination is not supported by the pipeline (e.g., methylation or
  spatial data not among the paper's CITE-seq/SMAGE-seq modalities)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMDC
  role: Target deep clustering model whose h5 input format (X1, X2, Y) and per-modality
    preprocessing requirements define the output of this skill
  repo: github.com/xianglin226/scMDC
- name: Scanpy
  role: Python package used to filter zero-count genes/ADTs, size-factor normalize,
    and log transform mRNA and ADT data separately with the same method
- name: Signac
  role: Used for SMAGE-seq inputs to map scATAC-seq reads to gene regions (GeneActivity)
    and filter reads by peak region fragments, nucleosome signal, and TSS enrichment
- name: h5py
  role: Used to load and write the h5 input files with X1/X2/Y structure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:bb51906798ce9ff586c1b0766b7a5b189f74ace62dabe3f1d212afec5339a163
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - Scanpy
  - Signac
  - h5py
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-multiomics-count-preprocessing

## Summary

Preprocess paired scRNA-seq and CITE-seq/SMAGE-seq count matrices (h5 inputs with X1=RNA, X2=ADT/ATAC, Y=labels) separately per modality using Scanpy — filtering zero-count features, size-factor normalizing, and log transforming — so the data are ready for deep multimodal clustering with scMDC.

## When to use

Use when raw or minimally processed CITE-seq or SMAGE-seq h5 count matrices need per-modality normalization (mRNA vs. ADT/ATAC) before feeding a deep clustering model such as scMDC.

## When NOT to use

- The downstream method expects already-scaled, variable-feature-selected inputs (e.g., a Seurat WNN workflow) rather than the size-factor/log-transformed matrices scMDC expects
- Input is a joint embedding or latent representation rather than raw per-modality count data
- The modality combination is not supported by the pipeline (e.g., methylation or spatial data not among the paper's CITE-seq/SMAGE-seq modalities)

## Inputs

- h5 input files with X1 = scRNA-seq count matrix, X2 = ADT or ATAC count matrix, Y = cell type labels (CITE-seq datasets: GSE100866, GSE128639/BMNC, GSE150599 SLN208/SLN111, 10X PBMC; SMAGE-seq: PBMC3K, PBMC10K, E18 mouse brain)
- scATAC-seq fragment files (for Signac GeneActivity mapping in SMAGE-seq preprocessing)

## Outputs

- Normalized, log-transformed RNA and ADT/ATAC matrices aligned on the same cells, formatted as scMDC h5 inputs
- Filtered feature sets (zero-count genes and ADTs removed) and label vectors with 'Unknown' cells excluded

## How to apply

Load the h5 input files structured as X1 (RNA), X2 (ADT or ATAC), and Y (cell type labels). Preprocess mRNA and ADT/ATAC data separately but with the same Scanpy pipeline: filter out genes/ADTs with no count, apply size-factor normalization, then log transform. For SMAGE-seq (scATAC-derived) inputs, first map scATAC-seq reads to gene regions with Signac GeneActivity, after filtering reads by ATAC peak region fragments, nucleosome signal, and TSS enrichment; optional feature selection is toggled by --filter1 (Genes) and --filter2 (ATAC) flags, both defaulting to No. Filter cells with 'Unknown' cell types where labels are uninformative (e.g., GSE100866). Keep modalities in separate normalized matrices because scMDC characterizes each data source explicitly with its own ZINB reconstruction loss.

## Related tools

- **scMDC** (Target deep clustering model whose h5 input format (X1, X2, Y) and per-modality preprocessing requirements define the output of this skill) — github.com/xianglin226/scMDC
- **Scanpy** (Python package used to filter zero-count genes/ADTs, size-factor normalize, and log transform mRNA and ADT data separately with the same method)
- **Signac** (Used for SMAGE-seq inputs to map scATAC-seq reads to gene regions (GeneActivity) and filter reads by peak region fragments, nucleosome signal, and TSS enrichment)
- **h5py** (Used to load and write the h5 input files with X1/X2/Y structure)

## Examples

```
python scmdc/src/main.py --dataset_file data/citeseq_bmnc.h5 --pretrain_epochs 400 --gamma 0.1 --prediction_file out/bmnc_preds.csv
```

## Evaluation signals

- No genes or ADTs with zero counts remain in either normalized matrix
- RNA and ADT/ATAC matrices contain the same cells in the same order and match the h5 X1/X2/Y structure expected by scMDC
- Both modalities were normalized separately but with the identical Scanpy recipe (size-factor normalization + log transform)
- For SMAGE-seq, ATAC reads passed the peak-region fragment, nucleosome signal, and TSS enrichment filters before GeneActivity quantification
- Downstream scMDC variants trained on the preprocessed data attain higher mean ARI/NMI than scDeepCluster on the 13 datasets, as in Fig. 7

## Limitations

- Number of clusters K is not estimated automatically; the user must specify it after pretraining
- All scMDC experiments were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- Feature selection (--filter1 on Genes, --filter2 on ATAC) defaults to No; enabling it changes downstream clustering behavior and must be matched across compared methods

## Evidence

- [discussion] Scanpy preprocessing of mRNA and ADT with separate normalization: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [discussion] Zero-count feature filtering: "Specifically, the genes and ADTs with no count are filtered out."
- [discussion] ATAC-to-gene mapping and read filtering for SMAGE-seq: "we map all the reads from scATAC-seq to the gene regions"
- [methods] h5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [methods] Feature selection flags: "--filter1: if do feature selection on Genes. Default: No."
