---
name: omics-alteration-frequency-computation
description: Use when raw per-cancer-type multi-omics alteration matrices (SNV, CNA,
  methylation, expression) must be reduced to per-gene feature summaries, e.g., to
  test whether predicted driver genes differ from other genes.
when_to_use_negative:
- Input is already a per-gene, per-cancer feature matrix with aggregated alteration
  scores — no further frequency averaging is needed
- Patient-level, sample-resolved analysis is required — averaging omics across all
  samples ignores the specific characteristics of individual cancer patients, as the
  paper itself notes
- The goal is to identify driver genes from raw data rather than to characterize or
  validate a pre-existing ranked prediction list
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DGMP
  role: Supplies the ranked gene driver probabilities ('logits' with driver probability
    in the second column) that define the top-100 predictions and the CG/NPCG groups
    whose omics alteration frequencies are computed and compared
  repo: https://github.com/NWPU-903PR/DGMP
- name: Python
  role: Used to load the multi-omics alteration matrices, compute per-gene SNV/CNA/methylation/expression
    metrics, run two-sided Mann-Whitney U tests, and render the comparison figure
  repo: https://github.com/NWPU-903PR/DGMP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:2d15446f0ed3e9b155eb525f8926a8cd6d88eb2389ab3a229965383db61f8a87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - DGMP
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# omics-alteration-frequency-computation

## Summary

Computes per-gene multi-omics alteration features — SNV frequency, CNA rate, DNA methylation change, and differential expression — across 16 cancer types from multi-omics alteration matrices, as used by DGMP to characterize and validate predicted cancer driver genes.

## When to use

Use when raw per-cancer-type multi-omics alteration matrices (SNV, CNA, methylation, expression) must be reduced to per-gene feature summaries, e.g., to test whether predicted driver genes differ from other genes.

## When NOT to use

- Input is already a per-gene, per-cancer feature matrix with aggregated alteration scores — no further frequency averaging is needed
- Patient-level, sample-resolved analysis is required — averaging omics across all samples ignores the specific characteristics of individual cancer patients, as the paper itself notes
- The goal is to identify driver genes from raw data rather than to characterize or validate a pre-existing ranked prediction list

## Inputs

- Multi-omics gene alteration matrices for 16 cancer types (SNV frequency, CNA rate, DNA methylation change, differential expression per gene)
- DGMP 'logits' output file from './code/result/' ranking genes by cancer-driver probability (second column)
- NCG cancer gene census annotation for partitioning predictions into CGs and NPCGs

## Outputs

- Per-gene averaged omics metrics (SNV frequency, CNA rate, DNA methylation change, differential expression) per group (CG vs NPCG)
- Group means table for the four omics metrics
- Two-sided Mann-Whitney U p-values for each of the four CG-vs-NPCG comparisons
- Figure panels reproducing the group-comparison plots (Figure 4 style)

## How to apply

For each of the 16 cancer types, compute the per-gene SNV frequency, CNA rate, DNA methylation change, and differential expression from the multi-omics alteration matrices, then average each metric across cancer types within the gene groups being compared. In the DGMP context, genes are first ranked by the driver-gene probability column (second column) of the 'logits' output stored in './code/result/', and the top-100 predictions are partitioned into candidate cancer genes (CGs) and non-probably cancer genes (NPCGs) via NCG cancer gene census membership. The rationale is that DGMP itself averages SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features representing each gene, so group-level comparisons (e.g., CG vs NPCG, two-sided Mann-Whitney U tests) should use the same averaged per-gene metrics. Report group means and Mann-Whitney U p-values for all four omics metrics; significance of SNVs, CNAs, and differential expression with non-significant methylation matches the reported pattern (P = 2.1E-04, P = 0.04, P = 4.3E-03, P = 0.2).

## Related tools

- **DGMP** (Supplies the ranked gene driver probabilities ('logits' with driver probability in the second column) that define the top-100 predictions and the CG/NPCG groups whose omics alteration frequencies are computed and compared) — https://github.com/NWPU-903PR/DGMP
- **Python** (Used to load the multi-omics alteration matrices, compute per-gene SNV/CNA/methylation/expression metrics, run two-sided Mann-Whitney U tests, and render the comparison figure) — https://github.com/NWPU-903PR/DGMP

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet  # then rank genes by column 2 of ./code/result/ 'logits' and test CG vs NPCG omics metrics
```

## Evaluation signals

- The four Mann-Whitney U comparisons reproduce the reported significance pattern: SNVs P = 2.1E-04, CNAs P = 0.04, differential expression P = 4.3E-03, DNA methylation P = 0.2 (non-significant)
- The top-100 driver set is correctly derived from the second column of the 'logits' output and filtered against the NCG census into CG and NPCG subsets
- Each of the four metrics is averaged across all 16 cancer types within each group before testing, mirroring DGMP's 4-omics feature construction
- The output figure contains four group-comparison panels (SNV, CNA, methylation, differential expression) plus a table of group means and p-values
- Mean 5-CV accuracy of the upstream DGMP run matches the 'acc' output stored alongside 'logits'

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores the specific characteristics of individual cancer patients
- DNA methylation change did not significantly differ between CGs and NPCGs (P = 0.2), so this metric alone is not discriminative for driver-gene validation
- Driver-gene probabilities depend on the GRN choice (DawnNet, KEGG, or RegNetwork), so recomputation on another network may yield different gene groups

## Evidence

- [discussion] averaging omics across samples: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
- [readme] logits output format: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [readme] driver probability column: "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits'"
- [intro] multi-omics integration rationale: "DGMP aims to not only identify the highly mutated cancer driver genes, but also predict the driver genes that harbor other kinds of alterations"
- [discussion] patient-level limitation: "which will ignore the specific characteristics of individual cancer patients."
