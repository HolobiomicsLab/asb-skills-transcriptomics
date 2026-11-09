---
name: sgrna-knockout-efficiency-evaluation
description: Use when preprocessing Perturb-seq/CROP-seq data before model building
  and you need to remove cells whose sgRNA-induced knockout failed (20-30% show wild-type
  phenotype) or perturbations represented by too few cells.
when_to_use_negative:
- 'You are analyzing CRISPRi data: MUSIC''s off-target analysis is only provided for
  knockout screens'
- The input is raw fastq data — MUSIC starts from the gene expression profile (e.g.,
  expression_profile.txt), not fastq
- Perturbation samples without a control labeled 'CTRL', which Cell_filtering's similarity-to-control
  comparison requires
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: MUSIC
  role: Provides Cell_filtering to remove perturbed cells with invalid sgRNA edits
    and perturbations with fewer than 30 cells or >=90% filtered cells
  repo: github.com/bm2-lab/MUSIC
- name: CRISPRseek
  role: Predicts candidate sgRNA off-targets from sgRNA fasta sequences via offTargetAnalysis
    with CFDscore, max.mismatch=3
- name: SAVER
  role: Imputes the QC-passed expression profile prior to cell filtering
  repo: github.com/mohuangx/SAVER
- name: R
  role: Runtime environment (R>=3.4.1) in which MUSIC and its dependencies execute
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:f8168b8275cec099f24ed25aefeeb306c74a5ff79a1582b3441cbfbe39aba27d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CRISPRseek
  - MUSIC
  - R
  - SAVER
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# sgrna-knockout-efficiency-evaluation

## Summary

Evaluates sgRNA knockout efficiency and off-target effects in single-cell CRISPR screening by filtering perturbed cells with invalid sgRNA edits and perturbations with too few cells (MUSIC's Cell_filtering), optionally augmented with CRISPRseek-based off-target prediction.

## When to use

Use when preprocessing Perturb-seq/CROP-seq data before model building and you need to remove cells whose sgRNA-induced knockout failed (20-30% show wild-type phenotype) or perturbations represented by too few cells.

## When NOT to use

- You are analyzing CRISPRi data: MUSIC's off-target analysis is only provided for knockout screens
- The input is raw fastq data — MUSIC starts from the gene expression profile (e.g., expression_profile.txt), not fastq
- Perturbation samples without a control labeled 'CTRL', which Cell_filtering's similarity-to-control comparison requires

## Inputs

- Imputed expression profile matrix (genes x cells) from Data_imputation
- Perturbation information vector (named character, control labeled 'CTRL')
- sgRNA_information.txt (optional; needed only for off-target analysis)
- sgRNA fasta file (e.g., crop_stimulated_sgrna.fa) for CRISPRseek

## Outputs

- Filtered expression profile and perturbation information (invalid-edit cells and low-cell-count perturbations removed)
- Per-cell keep/filter table from Cell_filtering
- Invalid_rate diagnostic plot
- Per-perturbation off-target report (offTarget_Info) when off-target analysis is run

## How to apply

After Cell_qc and SAVER-based Data_imputation (set.seed(234)), run MUSIC's Cell_filtering with cell_num_threshold=20 on the imputed expression profile and perturbation information: cells whose differentially expressed profile is more similar to control cells than to other same-perturbation cells are filtered as carrying invalid edits, perturbations with >=90% of cells filtered are dropped, and perturbations with fewer than 30 perturbed cells are excluded. To additionally assess off-target effects, run CRISPRseek's offTargetAnalysis on the sgRNA fasta (e.g., crop_stimulated_sgrna.fa) with BSgenome.Hsapiens.UCSC.hg38, CFDscore scoring.method, min.score=1, and max.mismatch=3, then correlate results via Get_offtarget() to build the per-perturbation offTarget_Info report. This is motivated by evidence that only 70-80% of sgRNAs generate true loss-of-function of the targeted gene.

## Related tools

- **MUSIC** (Provides Cell_filtering to remove perturbed cells with invalid sgRNA edits and perturbations with fewer than 30 cells or >=90% filtered cells) — github.com/bm2-lab/MUSIC
- **CRISPRseek** (Predicts candidate sgRNA off-targets from sgRNA fasta sequences via offTargetAnalysis with CFDscore, max.mismatch=3)
- **SAVER** (Imputes the QC-passed expression profile prior to cell filtering) — github.com/mohuangx/SAVER
- **R** (Runtime environment (R>=3.4.1) in which MUSIC and its dependencies execute)

## Examples

```
crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Cell_filtering returns a per-cell keep/filter table and the Invalid_rate plot is produced
- Filtering honored the >=90% filtered-proportion rule: perturbations at or above this threshold are dropped entirely
- No perturbation with fewer than 30 perturbed cells remains in the filtered output
- Across the 14 public screening datasets, sgRNA filtering removes on average ~41% of cells (QC removes ~6%)
- If off-target analysis is run, Get_offtarget() produces an offTarget_Info report per perturbation

## Limitations

- Only 70-80% of sgRNAs generate true loss-of-function of the targeted gene, so filtering cannot rescue failed knockouts
- MUSIC's authors acknowledge current filtering steps for data noise may need future refinement
- In the current version MUSIC only provides off-target information for knockout, not CRISPRi
- Cell_filtering can take a long time without parallel computation (cpu_num)

## Evidence

- [readme] filter cells with invalid edits: "Besides the conventional quality control and data normalization applied in single-cell RNA-seq analysis, **MUSIC** addresses two specific considerations"
- [discussion] cell similarity filter rule: "if the differentially expressed gene profile of a perturbed cell is more similar to the control cells than that of other same perturbed cells, this cell will be filtered"
- [discussion] perturbation thresholds: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [discussion] sgRNA efficiency rationale: "but only 70–80% of them will generate true loss-of-function of the targeted gene30,31"
- [discussion] off-target tooling: "CRISPRseek34 is performed to predict possible off-targets based on the sgRNA sequence information"
- [methods] Cell_filtering invocation: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
