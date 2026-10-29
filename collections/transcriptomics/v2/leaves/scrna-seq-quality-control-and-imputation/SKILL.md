---
name: scrna-seq-quality-control-and-imputation
description: Use when input is a raw gene-by-cell expression profile plus per-cell
  perturbation annotations (e.g. crop_stimulated or 10X perturb_GSM2396857 format)
  that must be QC'd and imputed before MUSIC topic modeling.
when_to_use_negative:
- Input is already quality-controlled and imputed (e.g. a pre-filtered crop_unstimulated.RData
  object past the preprocessing stage)
- Starting point is raw FASTQ data — MUSIC expects a gene expression profile, not
  sequencing reads
- The analysis goal is CRISPRi off-target prediction — MUSIC only provides off-target
  information for knockout screens
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: MUSIC
  role: Provides Input_preprocess/Input_preprocess_10X, Cell_qc, and Cell_filtering
    for preprocessing and quality control of single-cell CRISPR screening data
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Performs regularized regression prediction and empirical Bayes expression
    recovery (Data_imputation) on the noisy sparse expression matrix
  repo: https://github.com/mohuangx/SAVER
- name: devtools
  role: Used to install MUSIC from GitHub via install_github('bm2-lab/MUSIC') under
    R>=3.4.1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:3db7e10ddeeafeba5b11c6b6bf158e53d1a56a4b80dab602837daf21bb653a34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  - clusterProfiler
  - devtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# scrna-seq-quality-control-and-imputation

## Summary

Preprocess single-cell CRISPR screening (Perturb-seq/CROP-seq/CRISP-seq) expression matrices with MUSIC's cell quality control and SAVER-based data imputation so sparse, noisy count data are suitable for downstream topic modeling and perturbation effect quantification.

## When to use

Use when input is a raw gene-by-cell expression profile plus per-cell perturbation annotations (e.g. crop_stimulated or 10X perturb_GSM2396857 format) that must be QC'd and imputed before MUSIC topic modeling.

## When NOT to use

- Input is already quality-controlled and imputed (e.g. a pre-filtered crop_unstimulated.RData object past the preprocessing stage)
- Starting point is raw FASTQ data — MUSIC expects a gene expression profile, not sequencing reads
- The analysis goal is CRISPRi off-target prediction — MUSIC only provides off-target information for knockout screens

## Inputs

- Gene-by-cell expression profile table (e.g. crop_stimulated expression_profile.txt, 36722 genes x 3259 cells)
- Per-cell perturbation_information vector with control cells labeled 'CTRL'
- Optional sgRNA_information table (sgRNA sequences) for off-target evaluation
- 10X-format directory (barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv, cbc_gbc_dict_grna.tsv) for Input_preprocess_10X()

## Outputs

- QC-passed expression profile and perturbation_information (crop_seq_qc)
- SAVER-imputed expression matrix (crop_seq_imputation)
- Filtered cell/perturbation set with invalid edits and low-cell-count perturbations removed (crop_seq_filtered)
- Invalid_rate quality-control plot when Cell_filtering is run with plot=T

## How to apply

Load the expression profile and perturbation_information (control cells labeled exactly 'CTRL') and preprocess with Input_preprocess() for crop-style tables or Input_preprocess_10X() for 10X directories containing barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv and cbc_gbc_dict_grna.tsv; Input_preprocess also filters mitochondrial ribosomal protein (^MRP) and ribosomal protein (^RP) genes. Run Cell_qc(expression_profile, perturb_information, species='Hs') which retains cells with more than 500 genes detected, more than 1000 UMIs, and mitochondrial gene percentage below the threshold (10% default). Impute the filtered matrix with Data_imputation(..., cpu_num=30) using SAVER's regularized regression and empirical Bayes expression recovery (set.seed(234) for reproducibility); imputation is optional but improves data quality for downstream modeling. Feed the imputed object into Cell_filtering(cell_num_threshold=20) to drop perturbed cells whose DE profile resembles controls and perturbations with fewer than 30 perturbed cells, which are not considered by MUSIC.

## Related tools

- **MUSIC** (Provides Input_preprocess/Input_preprocess_10X, Cell_qc, and Cell_filtering for preprocessing and quality control of single-cell CRISPR screening data) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Performs regularized regression prediction and empirical Bayes expression recovery (Data_imputation) on the noisy sparse expression matrix) — https://github.com/mohuangx/SAVER
- **devtools** (Used to install MUSIC from GitHub via install_github('bm2-lab/MUSIC') under R>=3.4.1)

## Examples

```
library(MUSIC); crop_seq_list<-Input_preprocess(expression_profile,perturb_information); crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F); set.seed(234); crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30)
```

## Evaluation signals

- QC output retains only cells passing all three criteria: genes detected > 500, UMIs > 1000, and mitochondrial percentage below the 10% threshold
- Imputed matrix dimensions match the QC-passed cell set and set.seed(234) reproduces the SAVER result deterministically
- Cell_filtering with cell_num_threshold=20 drops perturbations whose perturbed-cell count falls below 30 and perturbations whose gene expression values are all zero across cells
- Downstream Get_high_varGenes/Get_topics(topic_number=c(4:6)) runs without error on the imputed, filtered matrix, indicating the object is in MUSIC's expected format
- The Perturb-seq UPR application reproduces the paper's TPDS ordering (PERK=94.0, IRE1alpha=23.2, ATF6=11.0), confirming preprocessing did not distort perturbation signals

## Limitations

- Cell_filtering without parallel computation can take a long time; the same applies to SAVER imputation (cpu_num parameter parallelizes both)
- Future improvements are expected to refine and update such filtering steps, as acknowledged by the authors
- Only 70-80% of sgRNAs generate true loss-of-function of the targeted gene, so filtering cannot fully remove invalid perturbation signals

## Evidence

- [discussion] MUSIC evaluates cell quality based on three factors29, i.e., number of genes detected (default 500), number of unique molecular identifiers induced (default 1000), and percentage of mitochondrial: "MUSIC evaluates cell quality based on three factors29, i.e., number of genes detected (default 500), number of unique molecular identifiers induced (default 1000), and percentage of mitochondrial"
- [discussion] To improve the quality of data, MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation: "To improve the quality of data, MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation"
- [readme] SAVER (Single-cell Analysis Via Expression Recovery) implements a regularized regression prediction and empirical Bayes method to recover the true gene expression profile in noisy and sparse single-cell RNA-seq data.: [evidence span withheld — anchor status: external_doc]
- [readme] The label of the control sample needs to be "CTRL".: "The label of the control sample needs to be "CTRL"."
- [methods] crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F): "crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F)"
