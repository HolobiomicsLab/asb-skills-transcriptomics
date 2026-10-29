---
name: single-cell-rnaseq-dataset-acquisition
description: Use when a benchmark or classification task requires downloading publicly
  available scRNA-seq datasets identified by GEO accession numbers (e.g., GSE104276,
  GSE108989) or supplement tables, and normalizing them per assay type before cell-type
  identification.
when_to_use_negative:
- The input is already a normalized expression matrix or feature table — re-normalizing
  would double-transform the data
- The task requires single-cell data generated in-house rather than datasets available
  under public accession numbers listed in the paper's supplement tables
- Cross-species gene mapping is not needed and HomoloGene-based conversion would drop
  genes unnecessarily
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: kallisto
  role: Estimates gene-level expression abundance from raw FASTQ of Mock HCA datasets
    using the hg19 human genome reference
- name: Seurat v3
  role: Provides default UMI count preprocessing (size-factor 10,000, add one, log
    normalization) for UMI-based datasets
- name: Python
  role: Used with sklearn for the stratified train/test splitting applied after datasets
    are acquired and normalized
- name: SciBet
  role: Downstream consumer of the acquired and normalized datasets; its GitHub repo
    hosts the scripts producing all quantitative results
  repo: https://github.com/PaulingLiu/scibet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:5c54b4f13a060732e13bedada33eec3842e6e18c31f94a92da9b34113324d421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - Python
  - SciBet
  - Seurat v3
  - kallisto
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-rnaseq-dataset-acquisition

## Summary

Retrieve the public scRNA-seq datasets cited in the SciBet benchmark (e.g., the 14 datasets of Supplementary Table 1, plus GSE104276, GSE108989, and the 42 human datasets of the Mock Human Cell Atlas) via their accession numbers, and prepare them for classification by the normalization scheme matching each assay (TPM + log for full-length reads; Seurat v3 UMI preprocessing with size-factor 10,000, add-one, log normalization).

## When to use

Use when a benchmark or classification task requires downloading publicly available scRNA-seq datasets identified by GEO accession numbers (e.g., GSE104276, GSE108989) or supplement tables, and normalizing them per assay type before cell-type identification.

## When NOT to use

- The input is already a normalized expression matrix or feature table — re-normalizing would double-transform the data
- The task requires single-cell data generated in-house rather than datasets available under public accession numbers listed in the paper's supplement tables
- Cross-species gene mapping is not needed and HomoloGene-based conversion would drop genes unnecessarily

## Inputs

- Dataset accession numbers from Supplementary Table 1 (14 benchmarking scRNA-seq datasets) and Supplementary Table 5 (16 datasets for 10 training-test pairs)
- Raw FASTQ files for 26 of the 42 Mock Human Cell Atlas human datasets
- Processed count matrices for full-length (TPM-based) and UMI-based assays
- NCBI HomoloGene Build 68 database for human-mouse gene conversion
- UCSC hg19 human genome reference for kallisto quantification

## Outputs

- Normalized expression matrices per dataset (TPM + log-normalized for full-length; Seurat v3 size-factor 10,000, add-one, log-normalized for UMI)
- Homologous-gene-mapped training sets with converted human gene symbols for cross-species use
- Gene-level expression abundance tables from kallisto for raw-FASTQ datasets

## How to apply

Enumerate the datasets from the paper's supplement tables — the 14 benchmarking datasets (Supplementary Table 1), the 16 datasets forming 10 training-test pairs (Supplementary Table 5), and the 42 human datasets of the Mock Human Cell Atlas — and fetch each via its public accession number. For datasets provided only as raw FASTQ (26 of the 42 HCA datasets), quantify gene-level expression with kallisto against the UCSC hg19 human genome reference. Then normalize according to assay technology: for full-length read count data compute TPM, add a pseudo-value of one to handle zeros, and log-normalize; for UMI data apply Seurat v3 preprocessing with size-factor 10,000, add one, and log normalization. For cross-species training (e.g., Tabula Muris mouse to human query), convert gene names using NCBI HomoloGene Build 68 one-to-one human-mouse orthologs and intersect genes between training set and look-up table. The GitHub repository's scripts directory contains the scripts that produced all quantitative results, which can be consulted to reproduce retrieval and preprocessing.

## Related tools

- **kallisto** (Estimates gene-level expression abundance from raw FASTQ of Mock HCA datasets using the hg19 human genome reference)
- **Seurat v3** (Provides default UMI count preprocessing (size-factor 10,000, add one, log normalization) for UMI-based datasets)
- **Python** (Used with sklearn for the stratified train/test splitting applied after datasets are acquired and normalized)
- **SciBet** (Downstream consumer of the acquired and normalized datasets; its GitHub repo hosts the scripts producing all quantitative results) — https://github.com/PaulingLiu/scibet

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- All 14 datasets from Supplementary Table 1 are retrieved and each normalizes without error under its assay-appropriate scheme (TPM for full-length, Seurat v3 UMI pipeline for UMI data)
- Log-normalized matrices contain no zero values after the add-one pseudo-value step
- Cross-species training sets contain only HomoloGene Build 68 one-to-one ortholog gene symbols shared between training set and query
- Downstream classification (scmapCluster, 7:3 stratified splits repeated 50 times) on the acquired datasets reproduces the accuracy ordering reported in Fig. 2a
- Retrieved data match the dataset descriptions (e.g., GSE104276 comprises cells from human prefrontal cortex; GSE108989 contains T cells)

## Limitations

- Numerous datasets are cited only by accession numbers with full enumeration deferred to Supplementary Tables 4 and 5, which may not be included in the synthesized document
- No seed or randomization parameters are reported for the 50x repeated splits, complicating exact reproduction of downstream validation
- Explicit version numbers for key software (Seurat v3 minor version, kallisto, scikit-learn, imbalanced-learn) are not provided in the article body

## Evidence

- [discussion] TPM normalization for full-length data: "we calculated Transcript Per Million (TPM)20, added pseudo value one to handle 0 values and performed log-normalization"
- [discussion] Seurat v3 UMI preprocessing: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [discussion] kallisto quantification of raw FASTQ: "estimated the gene-level expression abundance with kallisto23 and human genome reference hg19"
- [discussion] HomoloGene cross-species mapping: "We used the HomoloGene databases provided by NCBI (Build 68) to identify homologous genes between human and mouse"
- [discussion] Example dataset accession and composition: "For dataset GSE104276, which comprised cells from human prefrontal cortex, we randomly sampled 70% of the cells as reference"
- [readme] Reproduction scripts hosted in repository: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
