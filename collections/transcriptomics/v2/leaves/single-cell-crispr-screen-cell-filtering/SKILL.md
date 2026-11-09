---
name: single-cell-crispr-screen-cell-filtering
description: Use when preprocessing single-cell CRISPR screening data (Perturb-seq/CROP-seq
  expression profile plus perturbation information) to remove invalid-edit cells and
  low-cell-count perturbations before model building.
when_to_use_negative:
- Input is fastq data or raw sequencing output — MUSIC's starting point is a gene
  expression profile, not fastq files
- Data has not passed cell QC (genes >500, UMIs >1000, mito % < threshold) — Cell_filtering
  operates on QC-passed, imputed data
- Analysis goal is perturbation effect prioritization/topic modeling — this skill
  is only the preprocessing stage before Get_high_varGenes/Get_topics
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: MUSIC
  role: Provides Input_preprocess(), Cell_qc(), and Cell_filtering() functions that
    implement the QC, invalid-edit filtering, and minimal-cells-per-perturbation filtering
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Performs expression recovery/imputation of the sparse single-cell RNA-seq
    matrix prior to cell filtering
  repo: https://github.com/mohuangx/SAVER
- name: R
  role: Runtime environment (R>=3.4.1) with devtools used to install MUSIC from GitHub
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:69fb1da0c3dd908c39a7ab51e7f1729aa0dfbf427151c028c764a587a204cdc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - CRISPRseek
  - MUSIC
  - R
  - SAVER
  - clusterProfiler
  - devtools
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-crispr-screen-cell-filtering

## Summary

MUSIC's data-preprocessing filtering workflow that removes perturbed cells carrying invalid sgRNA edits and drops under-represented perturbations before topic-model building, using SAVER imputation followed by Cell_filtering with a 20-cell threshold, a 90% filtered-rate rule, and a 30-cell minimum per perturbation.

## When to use

Use when preprocessing single-cell CRISPR screening data (Perturb-seq/CROP-seq expression profile plus perturbation information) to remove invalid-edit cells and low-cell-count perturbations before model building.

## When NOT to use

- Input is fastq data or raw sequencing output — MUSIC's starting point is a gene expression profile, not fastq files
- Data has not passed cell QC (genes >500, UMIs >1000, mito % < threshold) — Cell_filtering operates on QC-passed, imputed data
- Analysis goal is perturbation effect prioritization/topic modeling — this skill is only the preprocessing stage before Get_high_varGenes/Get_topics

## Inputs

- Gene expression profile matrix (e.g., expression_profile.txt from crop_stimulated, genes x cells)
- Perturbation information vector/data frame (e.g., perturb_information.txt, control labeled 'CTRL')
- QC-passed expression profile and perturbation information from Cell_qc()
- SAVER-imputed expression profile from Data_imputation()

## Outputs

- Filtered expression profile and perturbation information (cells with invalid edits removed)
- Per-cell keep/filter table from Cell_filtering()
- Invalid_rate diagnostic plot (Invalid_rate.png)
- Perturbation-level table with dropped perturbations (<30 perturbed cells or >=90% filtered)

## How to apply

Load the expression profile and perturbation annotation (e.g., crop_stimulated's expression_profile.txt and perturb_information.txt) with Input_preprocess() (control cells labeled 'CTRL'), then run Cell_qc(species='Hs') retaining cells with >500 detected genes, >1000 UMIs, and mitochondrial percentage below the default threshold. Impute the QC-passed matrix with Data_imputation (SAVER-based, set.seed(234), cpu_num=30) to reduce sparsity, because Cell_filtering compares differential-expression profiles on the imputed data. Run Cell_filtering with cell_num_threshold=20 and cpu_num=30: cells whose DE profile is more similar to control cells than to other same-perturbation cells are filtered (motivated by evidence that only 70–80% of sgRNAs generate true loss-of-function), perturbations with >=90% of cells filtered are dropped, and perturbations with fewer than 30 perturbed cells are excluded. Inspect the returned per-cell keep/filter table and the Invalid_rate plot to confirm the thresholds were honored.

## Related tools

- **MUSIC** (Provides Input_preprocess(), Cell_qc(), and Cell_filtering() functions that implement the QC, invalid-edit filtering, and minimal-cells-per-perturbation filtering) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Performs expression recovery/imputation of the sparse single-cell RNA-seq matrix prior to cell filtering) — https://github.com/mohuangx/SAVER
- **R** (Runtime environment (R>=3.4.1) with devtools used to install MUSIC from GitHub)

## Examples

```
crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Cell_qc retains only cells with genes detected >500, UMIs >1000, and mitochondrial gene percentage below the threshold
- Per-cell keep/filter table and Invalid_rate plot are produced by Cell_filtering
- Filtering honored the 90% rule: perturbations with >=90% of influenced cells filtered are dropped
- Perturbations with fewer than 30 perturbed cells (cell_num_threshold=20 passed to Cell_filtering) are excluded from downstream analysis
- Cross-dataset QC-filtered proportion averages ~6% and sgRNA-filtered proportion ~41% across the 14 screening datasets

## Limitations

- Only 70–80% of sgRNAs generate true loss-of-function of the targeted gene, motivating but limiting the invalid-edit filter
- The article acknowledges future improvements are expected to refine and update such filtering steps
- Cell filtering and imputation can take a long time without parallel computation (cpu_num=30 recommended)

## Evidence

- [discussion] if the differentially expressed gene profile of a perturbed cell is more similar to the control cells than that of other same perturbed cells, this cell will be filtered: "if the differentially expressed gene profile of a perturbed cell is more similar to the control cells than that of other same perturbed cells, this cell will be filtered"
- [discussion] the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [methods] Cell quality control
    crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F): "crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F)"
- [readme] MUSIC addresses two specific considerations... (1) Filtering perturbed cells with invalid edits; and (2) Filtering perturbations according to a minimal number of cells per perturbation.: "MUSIC addresses two specific considerations that should be taken into account for such novel data type: **(1)** Filtering perturbed cells with invalid edits; and **(2)** Filtering perturbations"
- [discussion] Only cells with the first two factors above the thresholds and the third factor below the threshold are retained: "Only cells with the first two factors above the thresholds and the third factor below the threshold are retained"
