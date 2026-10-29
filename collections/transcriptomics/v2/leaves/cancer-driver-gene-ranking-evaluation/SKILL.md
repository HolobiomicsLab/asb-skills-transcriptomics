---
name: cancer-driver-gene-ranking-evaluation
description: Use when you have DGMP (or similar) gene-level driver-probability predictions
  (e.g., 'logits' from './code/result/') and need to assess whether top-ranked, unannotated
  predictions show cancer-like multi-omics alteration patterns versus known cancer
  genes.
when_to_use_negative:
- The goal is training or evaluating DGMP prediction accuracy itself (use 5-fold cross-validation
  accuracy, 'acc', rather than post-hoc omics-feature comparison)
- Input is raw sequencing or expression data rather than precomputed per-gene multi-omics
  alteration matrices
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0204
tools:
- name: DGMP
  role: Produces the gene-level driver probability 'logits' output that is ranked
    to obtain the top-100 predicted driver genes
  repo: https://github.com/NWPU-903PR/DGMP
- name: Python
  role: Used (Python 3.7, PyTorch 1.7.1) to load logits, rank genes, split CGs/NPCGs
    against NCG, compute omics metrics, and run Mann-Whitney U tests
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:331737eed0ef3f94a957eaceeaafc85bdc75a61b9cfc1f4335f50bae9ee4a5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - DGMP
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cancer-driver-gene-ranking-evaluation

## Summary

Rank predicted cancer driver genes from DGMP 'logits' output, partition them into known cancer genes (CGs) and non-probable cancer genes (NPCGs) via NCG annotation, and evaluate whether NPCGs differ from CGs across four multi-omics features using Mann-Whitney U tests.

## When to use

Use when you have DGMP (or similar) gene-level driver-probability predictions (e.g., 'logits' from './code/result/') and need to assess whether top-ranked, unannotated predictions show cancer-like multi-omics alteration patterns versus known cancer genes.

## When NOT to use

- The goal is training or evaluating DGMP prediction accuracy itself (use 5-fold cross-validation accuracy, 'acc', rather than post-hoc omics-feature comparison)
- Input is raw sequencing or expression data rather than precomputed per-gene multi-omics alteration matrices

## Inputs

- DGMP 'logits' output file (two columns: probability of non-cancer driver gene in column 1, probability of cancer driver gene in column 2) from './code/result/'
- NCG cancer gene census annotation for gene membership checks
- Multi-omics alteration matrices per cancer type: SNV frequency, CNA rate, DNA methylation change, differential expression, for 16 cancer types

## Outputs

- Ranked top-100 predicted driver gene list (CGs and NPCGs)
- Figure with four panels comparing CG vs NPCG distributions of SNV frequency, CNA rate, DNA methylation change, and differential expression (reproducing Figure 4)
- Table of group means and two-sided Mann-Whitney U p-values for the four omics metrics

## How to apply

Load the DGMP 'logits' file from './code/result/' for the chosen network run (e.g., DawnNet) and rank genes by the cancer-driver probability column (second column), selecting the top-100 predicted driver genes. Annotate membership against the NCG cancer gene census to split the top-100 into candidate cancer genes (CGs) and non-probable cancer genes (NPCGs). For each of 16 cancer types, compute per-gene SNV frequency, CNA rate, DNA methylation change, and differential expression from the multi-omics alteration matrices, then average each metric within the CG and NPCG groups. Compare CG versus NPCG distributions per metric with two-sided Mann-Whitney U tests and render four comparison panels (as in Figure 4). The rationale is that true drivers should show elevated SNV, CNA, methylation change, and differential expression relative to genes lacking NCG annotation.

## Related tools

- **DGMP** (Produces the gene-level driver probability 'logits' output that is ranked to obtain the top-100 predicted driver genes) — https://github.com/NWPU-903PR/DGMP
- **Python** (Used (Python 3.7, PyTorch 1.7.1) to load logits, rank genes, split CGs/NPCGs against NCG, compute omics metrics, and run Mann-Whitney U tests)

## Examples

```
python DGMP.py --gpu-no 0 --dataset DawnNet  # then load './code/result/' logits, rank genes by column 2, select top-100, split CGs/NPCGs by NCG, run two-sided Mann-Whitney U tests
```

## Evaluation signals

- The top-100 genes ranked by column 2 of 'logits' reproduce the paper's prediction set on the DawnNet run
- The CG/NPCG split is reproducible against the NCG cancer gene census annotations
- Mann-Whitney U p-values match the reported significance pattern: SNVs P = 2.1E-04, CNAs P = 0.04, differential expression P = 4.3E-03 (significant), DNA methylation P = 0.2 (not significant)
- All four comparisons show predicted driver genes with higher SNV frequency, CNA rate, DNA methylation change, and differential expression than NPCGs
- The rendered figure reproduces the four panels of Figure 4

## Limitations

- DNA methylation difference between CGs and NPCGs is not statistically significant (P = 0.2), so methylation change cannot serve alone as a validation signal
- DGMP averages SNVs, CNAs, DNA methylation, and gene expression across all samples, which ignores specific characteristics of individual cancer patients
- DGMP simply concatenates the three DGCN embedding vectors and one MLP embedding vector without accounting for their differing contributions
- Comparisons are averaged across cancer types within groups; per-cancer-type heterogeneity may be masked

## Evidence

- [results] NPCGs differ significantly from other genes in SNVs (P = 2.1E-04), CNAs (P = 0.04), and gene differential expression (P = 4.3E-03), but not in DNA methylation (P = 0.2).: [evidence span withheld — anchor status: not_found]
- [readme] The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits': "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits'"
- [discussion] DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
- [readme] The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'.: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
