---
name: single-cell-crispr-screen-data-preprocessing
description: Use when raw input is a gene-by-cell expression profile (e.g. expression_profile.txt)
  plus per-cell perturbation labels (perturb_information.txt), or 10X barcodes/genes/matrix
  files with a cbc_gbc_dict, and cells must be QC'd, imputed, and filtered before
  perturbation ranking.
when_to_use_negative:
- Input is fastq data — MUSIC starts from gene expression profile data, not fastq
  files
- The analysis targets off-target effects for CRISPRi experiments — MUSIC only provides
  off-target information for knockout screens
- Control cells are labelled something other than 'CTRL' — labels must be renamed
  first or preprocessing will fail
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: MUSIC
  role: Provides Input_preprocess/Input_preprocess_10X, Cell_qc, Data_imputation,
    and Cell_filtering functions that implement the preprocessing steps
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: R package used internally by Data_imputation to recover true gene expression
    profiles from noisy sparse counts
  repo: https://github.com/mohuangx/SAVER
- name: devtools
  role: Used to install the MUSIC package from GitHub (install_github('bm2-lab/MUSIC'))
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:33c2acfd612f03b911026fdbb0c6e4926075f63fc61f06ed372e00205617b6c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  - devtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-crispr-screen-data-preprocessing

## Summary

Preprocessing pipeline from MUSIC for combined single-cell RNA-seq and pooled CRISPR screening data (Perturb-Seq/CRISP-seq/CROP-seq): input formatting, cell quality control, SAVER imputation, and perturbation/cell-level filtering that feed a topic model for perturbation effect prioritization.

## When to use

Use when raw input is a gene-by-cell expression profile (e.g. expression_profile.txt) plus per-cell perturbation labels (perturb_information.txt), or 10X barcodes/genes/matrix files with a cbc_gbc_dict, and cells must be QC'd, imputed, and filtered before perturbation ranking.

## When NOT to use

- Input is fastq data — MUSIC starts from gene expression profile data, not fastq files
- The analysis targets off-target effects for CRISPRi experiments — MUSIC only provides off-target information for knockout screens
- Control cells are labelled something other than 'CTRL' — labels must be renamed first or preprocessing will fail

## Inputs

- expression_profile.txt (gene x cell count matrix, tab-separated, row.names=1)
- perturb_information.txt (per-cell perturbation/gene target labels, control labelled 'CTRL')
- 10X directory with barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv, cbc_gbc_dict_grna.tsv
- sgRNA_information.txt (optional, for off-target evaluation via CRISPRseek)
- RData-formatted examples such as crop_unstimulated.RData or the crop_stimulated dataset

## Outputs

- QC'd expression profile and perturbation information (Cell_qc output object)
- SAVER-imputed expression profile (Data_imputation output object)
- Filtered expression profile and perturbation information with invalid-perturbed-cell filtering (Cell_filtering output object, e.g. crop_seq_filtered)
- Invalid_rate plot when Cell_filtering(plot=T)
- Preprocessed list suitable for downstream Get_topics / Select_topic_number / Rank_overall steps

## How to apply

Format inputs with Input_preprocess() (or Input_preprocess_10X() for 10X barcodes.tsv/genes.tsv/matrix.mtx plus cbc_gbc_dict files), ensuring the control sample label is exactly 'CTRL'; the preprocess step also filters mitochondrial ribosomal protein (^MRP) and ribosomal protein (^RP) genes. Apply Cell_qc with species='Hs' retaining cells with genes detected > 500, UMIs > 1000, and mitochondrial percentage < 10%. Optionally impute with Data_imputation backed by SAVER (cpu_num=30, set.seed(234) for reproducibility), then run Cell_filtering with cell_num_threshold=20 to drop perturbed cells whose DE profile is closer to control cells than to same-perturbation cells; perturbations whose targeted gene is zero in all cells, whose filtered cells reach the 90% default proportion, or with fewer than 30 perturbed cells are excluded. The rationale is that sparse, noisy Perturb-Seq data require both conventional scRNA-seq QC and perturbation-specific filtering before topic modeling with Get_topics (topic_number 4:6) and ranking via Rank_overall/Rank_specific.

## Related tools

- **MUSIC** (Provides Input_preprocess/Input_preprocess_10X, Cell_qc, Data_imputation, and Cell_filtering functions that implement the preprocessing steps) — https://github.com/bm2-lab/MUSIC
- **SAVER** (R package used internally by Data_imputation to recover true gene expression profiles from noisy sparse counts) — https://github.com/mohuangx/SAVER
- **devtools** (Used to install the MUSIC package from GitHub (install_github('bm2-lab/MUSIC')))

## Examples

```
crop_seq_list<-Input_preprocess(expression_profile,perturb_information); crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F); crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30); crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Control sample label must be 'CTRL' after Input_preprocess; otherwise downstream comparisons are invalid
- Cell_qc output should exclude cells failing genes detected > 500, UMIs > 1000, or mitochondrial percentage >= 10% thresholds
- Zero-rate check: perturbations whose targeted gene expression is zero in all cells (e.g. CDKN2A, PTPRD, IER3IP1 across the 14-dataset screen set) must be flagged and filtered, while retained perturbation count stays within the >= 30 perturbed cells rule
- Expression values must be normalized and rounded to fit the topic model; verify ^MRP and ^RP genes were removed during preprocessing
- Downstream sanity: topic number selection restricted to the 4:6 scope and ranking via Rank_overall should reproduce expected rank-order correlations (e.g. ~0.6 mean similarity across ablation settings)

## Limitations

- Imputation and Cell_filtering can take a long time without parallel computation; use cpu_num=30
- Authors acknowledge current filtering steps for data noise may need future refinement
- Only 70-80% of sgRNAs generate true loss-of-function of the targeted gene, and off-target analysis is limited to knockout (not CRISPRi)
- Recommended topic number is restricted to the narrow 4:6 scope

## Evidence

- [discussion] Cell QC thresholds: "MUSIC evaluates cell quality based on three factors29, i.e., number of genes detected (default 500), number of unique molecular identifiers induced (default 1000)"
- [discussion] Only cells with the first two factors above the thresholds and the third factor below the threshold are retained: "Only cells with the first two factors above the thresholds and the third factor below the threshold are retained"
- [discussion] MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation: "MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation"
- [discussion] Zero-expression perturbation filtering: "If the corresponding gene expression values of the perturbation are all zero among all the cells, this perturbation will be filtered directly"
- [readme] Control label requirement: "The label of the control sample needs to be "CTRL"."
- [readme] Preprocess filters ribosomal genes: "this function integrates the input data and filters mitochondrial ribosomal protein(^MRP) and ribosomal protein(^RP)."
