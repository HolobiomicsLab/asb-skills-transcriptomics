---
name: perturbation-effect-prioritization
description: Use when the input is a gene expression profile plus per-cell perturbation
  assignment (control cells labeled 'CTRL') and the goal is to rank perturbations
  and quantify their relationships at single-cell granularity.
when_to_use_negative:
- Input is raw FASTQ data — MUSIC starts from a gene expression profile, not fastq
  files
- The analysis goal is differential expression or cluster-level phenotype discovery
  only; pooled CRISPR screening without single-cell readout cannot resolve perturbation
  effects at cellular heterogeneity level
- Control sample labels cannot be set to 'CTRL', which MUSIC requires
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MUSIC
  role: Integrated topic-model pipeline performing QC, filtering, topic modeling,
    and perturbation effect prioritization (TPDS/TPDR/CS)
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Expression imputation of the sparse filtered single-cell matrix prior to topic
    modeling
  repo: https://github.com/mohuangx/SAVER
- name: clusterProfiler
  role: Functional enrichment annotation of each topic's genes (Topic_func_anno)
- name: CRISPRseek
  role: Predicts possible sgRNA off-targets from sgRNA sequence information
- name: devtools
  role: Installs MUSIC from GitHub (install_github('bm2-lab/MUSIC')) under R>=3.4.1
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:53f7861856143e4056a7da589b39e348eb2cd1ae28592db7bbd2fb5a0a72031a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - CRISPRseek
  - Get_topics
  - MUSIC
  - R
  - SAVER
  - clusterProfiler
  - devtools
  - topicmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# perturbation-effect-prioritization

## Summary

Quantitatively estimate and prioritize individual gene perturbation effects from single-cell CRISPR screening (Perturb-seq/CRISP-seq/CROP-seq) data using MUSIC's topic-model-based scores (TPDS, TPDR, CS), which capture overall effects, topic-specific effects, and perturbation-pair correlations.

## When to use

Use when the input is a gene expression profile plus per-cell perturbation assignment (control cells labeled 'CTRL') and the goal is to rank perturbations and quantify their relationships at single-cell granularity.

## When NOT to use

- Input is raw FASTQ data — MUSIC starts from a gene expression profile, not fastq files
- The analysis goal is differential expression or cluster-level phenotype discovery only; pooled CRISPR screening without single-cell readout cannot resolve perturbation effects at cellular heterogeneity level
- Control sample labels cannot be set to 'CTRL', which MUSIC requires

## Inputs

- Gene-by-cell expression profile matrix (e.g., crop_stimulated expression_profile.txt, 36722 genes x 3259 cells)
- Per-cell perturbation assignment vector (perturb_information), with control cells labeled 'CTRL'
- Optional sgRNA sequence information table (sgRNA_information.txt) for off-target analysis via CRISPRseek
- 10X-format directory (barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv, cbc_gbc_dict_grna.tsv), e.g., perturb_GSM2396857

## Outputs

- Overall perturbation ranking (Rank_overall) with TPDS impact scores per perturbation
- Topic-specific perturbation ranking (Rank_specific) with TPDR scores
- Correlation matrix between perturbations (Correlation_perturbation)
- Topic function annotations from clusterProfiler enrichment
- CSV result table (e.g., music_topic_scores_rankings.csv, perturbation_analysis_results.csv)

## How to apply

Preprocess the expression matrix and perturbation information with Input_preprocess() (crop-style) or Input_preprocess_10X() (10X directories with barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv), ensuring control cells carry the label 'CTRL'. Run Cell_qc() with default thresholds (genes detected > 500, UMIs > 1000, mitochondrial percentage below threshold), optionally impute with Data_imputation() using SAVER (e.g., cpu_num=30), and filter invalid edits with Cell_filtering() (cell_num_threshold=20; perturbations with fewer than 30 perturbed cells are dropped; perturbations are also dropped if gene expression is all-zero or >=90% of influenced cells are filtered). Fit graded LDA topic models over topic_number=c(4:6) with Get_topics(), select the optimal number data-drivenly with Select_topic_number() (topic specificity, purity, and their combination), annotate topics with Topic_func_anno(species='Hs') via clusterProfiler, and compute per-cell topic distribution differences with Diff_topic_distri(). Rank perturbations with Rank_overall() and Rank_specific(); internally, the topic probability difference (TPD) between perturbed and control cells is z-normalized and corrected for case/control sample imbalance by 1000x bootstrap resampling, aggregated into overall (TPDS) and topic-specific (TPDR) statistics, with the efficiency score CS computed by sweeping random thresholds. Quantify perturbation relationships with Correlation_perturbation() and export rankings/TPDS to CSV.

## Related tools

- **MUSIC** (Integrated topic-model pipeline performing QC, filtering, topic modeling, and perturbation effect prioritization (TPDS/TPDR/CS)) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Expression imputation of the sparse filtered single-cell matrix prior to topic modeling) — https://github.com/mohuangx/SAVER
- **clusterProfiler** (Functional enrichment annotation of each topic's genes (Topic_func_anno))
- **CRISPRseek** (Predicts possible sgRNA off-targets from sgRNA sequence information)
- **devtools** (Installs MUSIC from GitHub (install_github('bm2-lab/MUSIC')) under R>=3.4.1)

## Examples

```
library(MUSIC); crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Perturbation rankings from Rank_overall()/Rank_specific() match paper-reported orders on crop_stimulated (e.g., Cebpb ranked 2nd/1st, TP53 ranked 1st, LCK ranked 6th)
- Independently implemented TPD/TPDS/TPDR/CS values reproduce identical perturbation ordering to Rank_overall()/Rank_specific() on the same distri_diff object
- On the Perturb-seq UPR dataset, reproduced TPDS values recapitulate original findings: PERK=94.0 > IRE1alpha=23.2 > ATF6=11.0
- Perturbation correlations match reported values: cor(Cebpb,Rela)=0.99, cor(TP53,MLH1)=0.99, cor(LCK,ZAP70)=0.93
- QC retains only cells with genes detected > 500 and UMIs > 1000, and perturbations with fewer than 30 perturbed cells are excluded from downstream ranking

## Limitations

- Recommended topic number is restricted to a narrow scope (4 to 6)
- Current filtering steps for data noise may need future refinement
- Off-target analysis is only provided for knockout (not CRISPRi) perturbations
- Only 70–80% of sgRNAs generate true loss-of-function of the targeted gene, so knockout efficiency must be evaluated
- The distribution of topics between cases and controls is affected by their sample-number ratio; imbalance is handled by 1000x bootstrapping

## Evidence

- [abstract] MUSIC accurately quantifies and prioritizes the individual gene perturbation effect on cell phenotypes: "Comprehensive tests applied to all the publicly available data revealed that MUSIC accurately quantifies and prioritizes the individual gene perturbation effect on cell phenotypes"
- [discussion] Cell QC thresholds: "MUSIC evaluates cell quality based on three factors29, i.e., number of genes detected (default 500), number of unique molecular identifiers induced (default 1000), and percentage of mitochondrial"
- [discussion] Exclusion of small perturbations: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [readme] Sample imbalance handling: "such a sample imbalance issue is addressed by the bootstrapping strategy when prioritizing the perturbation effect"
- [readme] Installation: "You can install the **MUSIC** package from Github using **devtools** packages with R>=3.4.1."
