---
name: topic-model-input-preprocessing
description: Use when preparing pooled CRISPR screening combined with single-cell
  RNA-seq data (Perturb-seq/CROP-seq/CROP-seq expression profile + sgRNA perturbation
  assignments) for topic modeling, and you must remove cells with invalid sgRNA edits
  and under-represented perturbations.
when_to_use_negative:
- Input is raw FASTQ or fastq-level data — MUSIC's starting point is a gene expression
  profile, not fastq data
- Data is already filtered/QC-passed and has fewer than 30 cells per perturbation
  — the 30-cell minimum would incorrectly drop intended perturbations
- CRISPRi (CRISPR interference) screens — MUSIC's off-target analysis only supports
  knockout data
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: MUSIC
  role: Integrated pipeline providing Input_preprocess, Cell_qc, Cell_filtering, and
    Get_offtarget for preprocessing single-cell CRISPR screening data before topic
    model building
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: R package used by MUSIC's Data_imputation to recover noisy and sparse single-cell
    RNA-seq expression prior to cell filtering
  repo: https://github.com/mohuangx/SAVER
- name: CRISPRseek
  role: Predicts candidate off-targets from sgRNA sequences (CFDscore, max.mismatch=3)
    to flag perturbations with invalid edits
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:3abdfe0bb9269ad5b6fb6bcc9f12855edd507acf9be81abf15a95aad95512ea7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - CRISPRseek
  - MUSIC
  - R
  - SAVER
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# topic-model-input-preprocessing

## Summary

MUSIC's data preprocessing step that cleans single-cell CRISPR screening data (e.g. CROP-seq expression_profile.txt plus perturb_information.txt) before topic model building by filtering perturbed cells with invalid edits and filtering perturbations represented by too few cells.

## When to use

Use when preparing pooled CRISPR screening combined with single-cell RNA-seq data (Perturb-seq/CROP-seq/CROP-seq expression profile + sgRNA perturbation assignments) for topic modeling, and you must remove cells with invalid sgRNA edits and under-represented perturbations.

## When NOT to use

- Input is raw FASTQ or fastq-level data — MUSIC's starting point is a gene expression profile, not fastq data
- Data is already filtered/QC-passed and has fewer than 30 cells per perturbation — the 30-cell minimum would incorrectly drop intended perturbations
- CRISPRi (CRISPR interference) screens — MUSIC's off-target analysis only supports knockout data

## Inputs

- expression_profile.txt gene-by-cell count matrix (e.g. 36722 genes x 3259 cells)
- perturb_information.txt per-cell sgRNA/perturbation assignment vector (control label must be 'CTRL')
- sgRNA_information.txt and sgRNA fasta (e.g. crop_stimulated_sgrna.fa) — optional, for off-target analysis
- 10X directory: barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv, cbc_gbc_dict_grna.tsv (via Input_preprocess_10X)

## Outputs

- QC- and filtering-passed expression profile and perturbation information ready for MUSIC topic modeling
- Per-cell keep/filter table from Cell_filtering
- Invalid_rate plot showing sgRNA knockout efficiency
- offTarget_Info per-perturbation off-target report (if CRISPRseek/Get_offtarget run)

## How to apply

Load expression_profile.txt and perturb_information.txt from a crop_stimulated-style dataset with Input_preprocess() (or Input_preprocess_10X() for 10X 'barcodes.tsv', 'genes.tsv', 'matrix.mtx' directories), then run Cell_qc(species='Hs') retaining cells with >500 genes detected, >1000 UMIs, and <10% mitochondrial reads. Optionally impute with Data_imputation (SAVER-based, set.seed(234)) and run Cell_filtering with cell_num_threshold=20: cells whose DE profile is more similar to control cells than to other same-perturbation cells are filtered (evidence that only 70-80% of sgRNAs generate true loss-of-function), perturbations with >=90% of cells filtered are dropped, and perturbations with fewer than 30 perturbed cells are excluded. Control sample labels must be 'CTRL'; mitochondrial (^MRP) and ribosomal protein (^RP) genes are filtered during preprocessing. If off-target checking is desired, additionally run CRISPRseek's offTargetAnalysis on the sgRNA fasta (BSgenome.Hsapiens.UCSC.hg38, CFDscore, min.score=1, max.mismatch=3) and correlate with Get_offtarget().

## Related tools

- **MUSIC** (Integrated pipeline providing Input_preprocess, Cell_qc, Cell_filtering, and Get_offtarget for preprocessing single-cell CRISPR screening data before topic model building) — https://github.com/bm2-lab/MUSIC
- **SAVER** (R package used by MUSIC's Data_imputation to recover noisy and sparse single-cell RNA-seq expression prior to cell filtering) — https://github.com/mohuangx/SAVER
- **CRISPRseek** (Predicts candidate off-targets from sgRNA sequences (CFDscore, max.mismatch=3) to flag perturbations with invalid edits)

## Examples

```
crop_seq_filtered <- Cell_filtering(crop_seq_imputation$expression_profile, crop_seq_imputation$perturb_information, cell_num_threshold=20, plot=T, cpu_num=30)
```

## Evaluation signals

- Per-cell keep/filter table is produced by Cell_filtering with cell_num_threshold=20 honored
- No perturbation with fewer than 30 perturbed cells survives into the downstream topic model input
- Any perturbation with >=90% of its cells filtered is dropped
- Invalid_rate plot is generated showing the proportion of cells with wild-type phenotype per sgRNA (expected 20-30% invalid rate motivation)
- Control cells retain the exact label 'CTRL' in the output perturbation information

## Limitations

- Only 70-80% of sgRNAs generate true loss-of-function of the targeted gene, so 20-30% of sgRNA-detected cells may show a wild-type phenotype and filtering thresholds may need tuning
- Current filtering steps for data noise may need future refinement, as acknowledged by the authors
- Off-target analysis is only provided for knockout, not CRISPRi
- Recommended topic number is restricted to a narrow scope (4 to 6) downstream of this preprocessing

## Evidence

- [readme] Filtering perturbed cells with invalid edits and filtering perturbations according to a minimal number of cells per perturbation: "Filtering perturbed cells with invalid edits; and (2) Filtering perturbations according to a minimal number of cells per perturbation"
- [discussion] Cell filtering similarity rule: "if the differentially expressed gene profile of a perturbed cell is more similar to the control cells than that of other same perturbed cells, this cell will be filtered"
- [discussion] 90% filter proportion and 30-cell minimum thresholds: "if the influenced cells filtered are amount to a high proportion (default 90%) among all, such perturbation is filtered"
- [discussion] 30-cell minimum perturbation threshold: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [discussion] sgRNA loss-of-function rate motivating invalid-edit filtering: "but only 70–80% of them will generate true loss-of-function of the targeted gene30,31"
- [methods] Concrete Cell_filtering invocation from README: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
