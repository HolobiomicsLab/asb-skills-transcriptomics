---
name: perturbation-correlation-analysis
description: Use when Perturb-seq/CROP-seq expression data plus perturbation labels
  (control labeled 'CTRL') have been topic-modeled with MUSIC and you need to quantify
  how two gene knockouts (e.g., Cebpb vs Rela) relate.
when_to_use_negative:
- The perturbation of interest has fewer than 30 perturbed cells after filtering —
  MUSIC drops such perturbations and no correlation can be computed
- Input is raw fastq data rather than a gene expression profile — MUSIC's starting
  point is expression-profile input, not fastq
- You need off-target information for CRISPRi experiments — MUSIC only provides off-target
  analysis for knockout screens
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: MUSIC
  role: Provides Correlation_perturbation() and the topic-model pipeline (Input_preprocess,
    Cell_qc, Data_imputation, Cell_filtering, Get_topics, Diff_topic_distri) that
    produces the perturbation relationship quantification
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Expression imputation applied via Data_imputation() to the QC-filtered matrix
    before topic modeling and correlation computation
  repo: https://github.com/mohuangx/SAVER
- name: clusterProfiler
  role: Used in Topic_func_anno(species='Hs') to annotate topic functions via enrichment,
    contextualizing the perturbation correlations
- name: devtools
  role: Installs MUSIC from GitHub via install_github('bm2-lab/MUSIC') under R>=3.4.1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:eb033ab00dfcb5873e9bdb8cab988cd6675b67346547a1a6cfc71e6a82fea9fa
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# perturbation-correlation-analysis

## Summary

Quantifies pairwise relationships between gene perturbations in single-cell CRISPR screens using MUSIC's topic-model cell profiles, reproducing published correlations such as cor(Cebpb,Rela)=0.99 and cor(TP53,MLH1)=0.99.

## When to use

Use when Perturb-seq/CROP-seq expression data plus perturbation labels (control labeled 'CTRL') have been topic-modeled with MUSIC and you need to quantify how two gene knockouts (e.g., Cebpb vs Rela) relate.

## When NOT to use

- The perturbation of interest has fewer than 30 perturbed cells after filtering — MUSIC drops such perturbations and no correlation can be computed
- Input is raw fastq data rather than a gene expression profile — MUSIC's starting point is expression-profile input, not fastq
- You need off-target information for CRISPRi experiments — MUSIC only provides off-target analysis for knockout screens

## Inputs

- Gene expression profile matrix (e.g., crop_stimulated expression_profile.txt, 36722 genes x 3259 cells) with matched perturbation_information vector labeled 'CTRL' for controls
- 10X-format directory (barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv, cbc_gbc_dict_grna.tsv) such as perturb_GSM2396857
- Per-cell topic distributions from MUSIC's Diff_topic_distri() step

## Outputs

- Pairwise perturbation correlation scores per dataset (e.g., cor(Cebpb,Rela)=0.99, cor(TP53,MLH1)=0.99, cor(LCK,ZAP70)=0.93)
- perturbation_analysis_results.csv containing ranks, correlations, and TPDS values (PERK=94.0, IRE1alpha=23.2, ATF6=11.0)

## How to apply

After preprocessing the expression profile and perturbation information with Input_preprocess() or Input_preprocess_10X(), run Cell_qc (genes detected > 500, UMIs > 1000, mitochondrial percentage below threshold), Data_imputation (SAVER, cpu_num=30), and Cell_filtering (cell_num_threshold=20; perturbations with fewer than 30 perturbed cells dropped). Then identify highly dispersion differentially expressed genes with Get_high_varGenes, fit topic models over topic_number=c(4:6), select the optimal topic number, and compute per-cell topic distributions. Call Correlation_perturbation() on those topic distributions to derive pairwise perturbation-relationship scores (e.g., cor(Cebpb,Rela)=0.99, cor(TP53,MLH1)=0.99, cor(LCK,ZAP70)=0.93) and write the results to perturbation_analysis_results.csv for validation against paper-reported values.

## Related tools

- **MUSIC** (Provides Correlation_perturbation() and the topic-model pipeline (Input_preprocess, Cell_qc, Data_imputation, Cell_filtering, Get_topics, Diff_topic_distri) that produces the perturbation relationship quantification) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Expression imputation applied via Data_imputation() to the QC-filtered matrix before topic modeling and correlation computation) — https://github.com/mohuangx/SAVER
- **clusterProfiler** (Used in Topic_func_anno(species='Hs') to annotate topic functions via enrichment, contextualizing the perturbation correlations)
- **devtools** (Installs MUSIC from GitHub via install_github('bm2-lab/MUSIC') under R>=3.4.1)

## Examples

```
library(MUSIC); crop_seq_filtered <- Cell_filtering(crop_seq_imputation$expression_profile, crop_seq_imputation$perturb_information, cell_num_threshold=20, plot=T, cpu_num=30)
```

## Evaluation signals

- Reproduced pairwise correlations match paper values: cor(Cebpb,Rela)=0.99, cor(TP53,MLH1)=0.99, cor(LCK,ZAP70)=0.93
- Perturbations with fewer than 30 perturbed cells are excluded, and perturbations whose affected cells are >=90% filtered are dropped
- All perturbed cells retained after Cell_qc pass thresholds (genes detected > 500, UMIs > 1000, mitochondrial percentage below default)
- Results table perturbation_analysis_results.csv is populated with ranks, correlations, and TPDS values consistent with MUSIC's overall/topic-specific outputs
- Control cells are labeled 'CTRL' in perturbation_information, as required by MUSIC's input convention

## Limitations

- The recommended topic number scope is narrow (4 to 6), which may constrain model fit on other datasets
- Off-target analysis is only provided for knockout, not CRISPRi perturbations
- Current filtering steps for data noise may need future refinement, per the authors' acknowledged limitation
- Data imputation with SAVER and Cell_filtering may take a long time without parallel computation (cpu_num=30 used)

## Evidence

- [abstract] MUSIC prioritizes gene perturbation effects overall, in a functional topic-specific way, and quantifies relationships between different perturbations: "prioritizing the gene perturbation effect as an overall perturbation effect, in a functional topic-specific way, and quantifying the relationships between different perturbations"
- [methods] perturbation cell-count filter: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [methods] MUSIC install via devtools: "You can install the MUSIC package from Github using devtools packages with R>=3.4.1"
- [discussion] off-target limitation: "In the current version MUSIC only provides the off-target information of the knockout"
- [readme] control label requirement: "The label of the control sample needs to be "CTRL"."
