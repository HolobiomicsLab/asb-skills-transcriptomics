---
name: topic-model-based-perturbation-effect-ranking
description: Use when analyzing Perturb-Seq/CROP-seq expression profiles plus per-cell
  perturbation labels (control labeled 'CTRL') to quantitatively prioritize individual
  gene perturbation effects at single-cell granularity.
when_to_use_negative:
- Input is raw FASTQ data — MUSIC starts from the gene expression profile, not fastq
  data
- Perturbations with perturbed cells lower than 30 (default) — MUSIC does not consider
  them in ranking
- Perturbations whose corresponding gene expression values are all zero among all
  the cells — they are filtered directly before ranking
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MUSIC
  role: Integrated pipeline providing Input_preprocess, Cell_qc, Data_imputation,
    Cell_filtering, Get_topics, Select_topic_number, Diff_topic_distri, Rank_overall
    and Rank_specific used to build topic models and rank perturbation effects
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: R package adopted by MUSIC for single-cell RNA-seq data imputation before
    topic modeling
  repo: https://github.com/mohuangx/SAVER
- name: R
  role: Runtime environment (R>=3.4.1) in which the MUSIC package is installed and
    executed
- name: devtools
  role: Used to install the MUSIC package from GitHub via install_github("bm2-lab/MUSIC")
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:b50b6828a6c114d801ec74e5c1243a48653a880939667260eca681df14b8f9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# topic-model-based-perturbation-effect-ranking

## Summary

MUSIC's topic-modeling pipeline that ranks gene perturbation effects from single-cell CRISPR screening data: after QC, imputation (SAVER) and cell filtering, it builds topic models on the entire screen dataset, automatically selects the optimal topic number, computes cell-level topic distributions, and prioritizes perturbations via overall and topic-specific rankings.

## When to use

Use when analyzing Perturb-Seq/CROP-seq expression profiles plus per-cell perturbation labels (control labeled 'CTRL') to quantitatively prioritize individual gene perturbation effects at single-cell granularity.

## When NOT to use

- Input is raw FASTQ data — MUSIC starts from the gene expression profile, not fastq data
- Perturbations with perturbed cells lower than 30 (default) — MUSIC does not consider them in ranking
- Perturbations whose corresponding gene expression values are all zero among all the cells — they are filtered directly before ranking

## Inputs

- expression_profile.txt (genes x cells count matrix, e.g. crop_stimulated: 36722 genes x 3259 cells)
- perturb_information.txt (per-cell perturbation labels; control cells must be labeled 'CTRL')
- sgRNA_information.txt (optional, for off-target effect evaluation)
- 10X-format inputs (barcodes.tsv, genes.tsv, matrix.mtx, cbc_gbc_dict.tsv, cbc_gbc_dict_grna.tsv) via Input_preprocess_10X()

## Outputs

- Overall perturbation effect ranking list (Rank_overall)
- Functional topic-specific perturbation ranking list (Rank_specific)
- Cell-level topic distribution profiles (Diff_topic_distri)
- Topic occurrence probabilities of genes used for functional annotation (e.g. via clusterProfiler)
- QC/filtered expression profiles and Invalid_rate plots from Cell_qc/Cell_filtering

## How to apply

Preprocess the input with Input_preprocess() (control sample must be labeled 'CTRL', mitochondrial ^MRP and ribosomal ^RP genes are filtered), then run Cell_qc (genes detected > 500, UMIs > 1000, mitochondrial percentage < 10%), optionally Data_imputation with SAVER (cpu_num=30), and Cell_filtering with cell_num_threshold=20 to remove perturbed cells whose DE profile resembles controls. Build topic models with Get_topics over a recommended scope of topic numbers 4 to 6, select the optimal topic number in a data-driven manner with Select_topic_number, and obtain cell-level topic distributions with Diff_topic_distri. Perturbations whose targeted gene expression is zero in all cells, or with fewer than 30 (default) perturbed cells, are excluded before ranking. Prioritize effects with Rank_overall (overall perturbation effect) and Rank_specific (topic-specific), using MUSIC's bootstrapping strategy to handle sample imbalance between cases and controls when comparing topic distributions.

## Related tools

- **MUSIC** (Integrated pipeline providing Input_preprocess, Cell_qc, Data_imputation, Cell_filtering, Get_topics, Select_topic_number, Diff_topic_distri, Rank_overall and Rank_specific used to build topic models and rank perturbation effects) — https://github.com/bm2-lab/MUSIC
- **SAVER** (R package adopted by MUSIC for single-cell RNA-seq data imputation before topic modeling) — https://github.com/mohuangx/SAVER
- **R** (Runtime environment (R>=3.4.1) in which the MUSIC package is installed and executed)
- **devtools** (Used to install the MUSIC package from GitHub via install_github("bm2-lab/MUSIC"))

## Examples

```
crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30)
```

## Evaluation signals

- The selected topic number falls within the recommended scope of 4 to 6 tried by Select_topic_number
- Ranking lists must exclude perturbations with fewer than 30 perturbed cells and perturbations whose gene is zero in all cells
- Ablation of imputation/filtering steps should yield ranking-list Pearson correlations averaging about 0.6, with the combination strategy changing the ranking most
- Zero-expression filtering should remove only CDKN2A, PTPRD and IER3IP1 in the 14 datasets, with with/without-filter rank correlations of 0.99, 0.93 and 0.98
- Input validation: expression_profile dimensions match perturbation_information length (e.g. 36722 x 3259 profile with 3259 labeled cells) and control label is 'CTRL'

## Limitations

- The recommended topic number is restricted to a narrow scope (4 to 6)
- Only 70-80% of sgRNAs generate true loss-of-function of the targeted gene
- Off-target analysis is only provided for knockout, not CRISPRi
- Current filtering steps for data noise may need future refinement
- Data imputation and cell filtering may take a long time without parallel computation

## Evidence

- [discussion] topic modeling on entire dataset: "topic modeling was performed on the entire screen dataset to compare the impact of different perturbations under the same background"
- [discussion] automatic topic number selection and scope: "MUSIC applies an automatic strategy to select the optimal topic number"
- [discussion] topic scope 4-6: "we recommended a reasonable scope (now 4 to 6) of topic model number to be tried"
- [discussion] minimum perturbed cells threshold: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [readme] bootstrapping for sample imbalance: "such a sample imbalance issue is addressed by the bootstrapping strategy when prioritizing the perturbation effect"
- [readme] control label requirement: "The label of the control sample needs to be "CTRL"."
