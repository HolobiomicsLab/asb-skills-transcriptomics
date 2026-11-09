---
name: pancancer-multiomics-comparison-analysis
description: Use when validating gene-level predictions (e.g., DGMP 'logits' top-100
  drivers) by testing whether candidates differ from non-candidates in pan-cancer
  SNV, CNA, methylation, and expression alteration metrics.
when_to_use_negative:
- The goal is individual patient-specific characterization — the DGMP approach averages
  SNVs, CNAs, DNA methylation, and gene expression across all samples, which the authors
  note 'will ignore the specific characteristics of individual cancer patients.'
- You need methylation-specific validation only — in this workflow DNA methylation
  did not differ significantly between CGs and NPCGs (P = 0.2), so it alone cannot
  support the comparison.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: DGMP
  role: Source of the 'logits' predictions (per-gene cancer-driver probability) whose
    top-100 on DawnNet define the CG/NPCG groups being compared
  repo: https://github.com/NWPU-903PR/DGMP
- name: Python
  role: Implementation environment for the statistical comparison workflow (ranking
    genes, computing omics metrics, Mann-Whitney U tests, figure rendering)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:1e26f14c730c391dd736e0020471b3186ed394ddb2fc6edbfe563f8ab17b4042
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - DGMP
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pancancer-multiomics-comparison-analysis

## Summary

Compares predicted cancer driver genes (e.g., DGMP top-100 predictions on DawnNet) against other genes across 16 cancer types using four multi-omics features — SNV frequency, CNA rate, DNA methylation change, and differential expression — with two-sided Mann-Whitney U tests to validate that predictions capture pan-cancer alteration signals.

## When to use

Use when validating gene-level predictions (e.g., DGMP 'logits' top-100 drivers) by testing whether candidates differ from non-candidates in pan-cancer SNV, CNA, methylation, and expression alteration metrics.

## When NOT to use

- The goal is individual patient-specific characterization — the DGMP approach averages SNVs, CNAs, DNA methylation, and gene expression across all samples, which the authors note 'will ignore the specific characteristics of individual cancer patients.'
- You need methylation-specific validation only — in this workflow DNA methylation did not differ significantly between CGs and NPCGs (P = 0.2), so it alone cannot support the comparison.

## Inputs

- DGMP 'logits' output file from './code/result/' (two-column gene probabilities; second column = cancer-driver probability) for the DawnNet run
- NCG cancer gene census annotation (for partitioning top-100 into CGs vs NPCGs)
- Multi-omics gene alteration matrices across 16 cancer types (SNV frequency, CNA rate, DNA methylation change, differential expression per gene)

## Outputs

- Four-panel Figure 4 reproduction comparing CG vs NPCG distributions for SNVs, CNAs, DNA methylation change, and differential expression
- Table of CG/NPCG group means per omics metric with Mann-Whitney U p-values

## How to apply

Load the DGMP 'logits' output from './code/result/' for the DawnNet run and rank genes by the cancer-driver probability (second column) to select the top-100 predicted driver genes. Partition the top-100 into candidate cancer genes (CGs) and non-probably cancer genes (NPCGs) by membership against the NCG cancer gene census annotation. For each of the 16 cancer types, compute per-gene SNV frequency, CNA rate, DNA methylation change, and differential expression from the multi-omics alteration matrices, then average each metric across cancer types within the CG and NPCG groups. Compare CG versus NPCG distributions per metric with two-sided Mann-Whitney U tests and render the four comparisons as panels reproducing Figure 4, saving the figure with a table of group means and p-values. The rationale is that true driver predictions should show significantly higher alterations than NPCGs (e.g., SNVs P = 2.1E–04, CNAs P = 0.04, differential expression P = 4.3E–03).

## Related tools

- **DGMP** (Source of the 'logits' predictions (per-gene cancer-driver probability) whose top-100 on DawnNet define the CG/NPCG groups being compared) — https://github.com/NWPU-903PR/DGMP
- **Python** (Implementation environment for the statistical comparison workflow (ranking genes, computing omics metrics, Mann-Whitney U tests, figure rendering))

## Examples

```
python DGMP.py --gpu-no 0 --dataset DawnNet  # then rank './code/result/' logits col-2, split top-100 vs NCG, Mann-Whitney U per omics metric
```

## Evaluation signals

- All four Mann-Whitney U comparisons yield p-values consistent with Figure 4: SNVs P = 2.1E–04, CNAs P = 0.04, gene differential expression P = 4.3E–03 (all significant), and DNA methylation P = 0.2 (not significant).
- Top-100 gene selection matches ranking genes by the second column of the DGMP 'logits' output (cancer driver gene probability).
- CG/NPCG partition correctly reflects NCG cancer gene census membership of the top-100 predictions.
- Per-omics metrics are computed per gene within each of the 16 cancer types and then averaged across cancer types within each group.
- Output figure contains exactly four panels, one per omics metric, plus a table of group means and Mann-Whitney U p-values.

## Limitations

- DGMP's averaging of SNVs, CNAs, DNA methylation, and gene expression across all samples ignores specific characteristics of individual cancer patients.
- DNA methylation change shows no significant difference between CGs and NPCGs (P = 0.2), so this metric alone cannot discriminate predicted drivers in this validation.
- The comparison is restricted to genes on the DawnNet GRN and top-100 predictions; results depend on the NCG census used for the CG/NPCG split.

## Evidence

- [results] NPCGs differ significantly from other genes in SNVs (P = 2.1E–04), CNAs (P = 0.04), and gene differential expression (P = 4.3E–03), but not in DNA methylation (P = 0.2).: [evidence span withheld — anchor status: not_found]
- [discussion] DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features"
- [readme] The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits': "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column)"
- [readme] The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'.: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
