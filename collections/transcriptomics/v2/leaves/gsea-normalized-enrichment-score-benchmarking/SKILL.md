---
name: gsea-normalized-enrichment-score-benchmarking
description: 'Use when comparing pseudo-time/trajectory inference methods on scRNA-seq
  data and you need a quantitative, gene-set-based metric: run GSEA with a cell-cycle
  gene set on each method''s cell ordering and rank methods by NES.'
when_to_use_negative:
- The inferred cell orderings are not continuous pseudo-time values suitable for GSEA
  continuous phenotypes (e.g., only discrete phase labels) — use Cyclum's label refinement
  instead
- The gene set of interest is unrelated to periodic/circular processes; NES benchmarking
  against the cell-cycle set will not meaningfully distinguish trajectory methods
- Expression matrices have not been preprocessed with the recommended log2(x+1), centering
  (mean=1) and scaling (variance=1), which the article states works well for Cyclum-style
  inference
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Source of the primary pseudo-time ordering being benchmarked; sinusoidal autoencoder
    projecting cells onto a circular manifold
  repo: github.com/KChen-lab/cyclum
- name: reCAT
  role: Baseline pseudo-time method benchmarked against Cyclum; reconstructs cell-cycle
    pseudo-time with a Gaussian mixture model
- name: GSEA
  role: 'Scoring engine: computes normalized enrichment scores for the cell-cycle
    gene set against each method''s pseudo-time ordering'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:29b6d882cba856bc6a356311ae1160a774e67d63532e28e7bba464bd11740dde
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - GSEA
  - reCAT
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# GSEA normalized enrichment score benchmarking

## Summary

Benchmark cell-cycle pseudo-time inference methods (Cyclum, reCAT, PC1, PC2) by running GSEA against the GO cell-cycle gene set on the inferred orderings and comparing normalized enrichment scores (NES), mirroring Fig. 4a / Supplementary Table 1 of the Cyclum paper.

## When to use

Use when comparing pseudo-time/trajectory inference methods on scRNA-seq data and you need a quantitative, gene-set-based metric: run GSEA with a cell-cycle gene set on each method's cell ordering and rank methods by NES.

## When NOT to use

- The inferred cell orderings are not continuous pseudo-time values suitable for GSEA continuous phenotypes (e.g., only discrete phase labels) — use Cyclum's label refinement instead
- The gene set of interest is unrelated to periodic/circular processes; NES benchmarking against the cell-cycle set will not meaningfully distinguish trajectory methods
- Expression matrices have not been preprocessed with the recommended log2(x+1), centering (mean=1) and scaling (variance=1), which the article states works well for Cyclum-style inference

## Inputs

- Inferred cell pseudo-time orderings (one continuous ordering per method: Cyclum, reCAT, PC1, PC2)
- Preprocessed expression matrix (log2(x+1), centered mean=1, scaled variance=1) per sample
- GSEA cell-cycle gene set (e.g., GO cell-cycle gene set)

## Outputs

- Normalized enrichment scores (NES) per method per sample
- Ranking table of methods by NES per sample (e.g., gsea_nes_ranking.csv), mirroring Fig. 4a / Supplementary Table 1

## How to apply

For each sample and each candidate method (Cyclum sinusoidal autoencoder, reCAT GMM, PCA-derived PC1/PC2), take the inferred circular pseudo-time ordering of cells as the phenotype for GSEA. Run GSEA on the cell-cycle gene set using these continuous pseudo-time orderings and record the normalized enrichment score (NES) for each method. The README provides R helpers for this handoff: mat2txt writes the expression matrix to a GSEA-compatible .txt file, and vec2cls/mat2cls write single or multiple continuous phenotype orderings (e.g., pseudotime or multiple PCs) to a GSEA-compatible .cls file. Rank the methods by NES per sample and save the comparison table (e.g., gsea_nes_ranking.csv), mirroring Fig. 4a / Supplementary Table 1. Correct application is judged by whether the method claimed superior (here, Cyclum) attains the highest cell-cycle GSEA NES across samples, including hard cases like the low-cycling sample Mel79.

## Related tools

- **Cyclum** (Source of the primary pseudo-time ordering being benchmarked; sinusoidal autoencoder projecting cells onto a circular manifold) — github.com/KChen-lab/cyclum
- **reCAT** (Baseline pseudo-time method benchmarked against Cyclum; reconstructs cell-cycle pseudo-time with a Gaussian mixture model)
- **GSEA** (Scoring engine: computes normalized enrichment scores for the cell-cycle gene set against each method's pseudo-time ordering)

## Examples

```
from cyclum.helper import mat2txt, vec2cls; mat2txt(expr, 'expr.txt'); vec2cls(pseudotime, 'pseudotime.cls')
```

## Evaluation signals

- Cyclum's inferred pseudo-time achieves the highest GSEA cell-cycle NES across all five melanoma samples (Mel78, Mel79, Mel80, Mel81, Mel88)
- NES ranking holds on the difficult low-cycling sample Mel79
- Each method produces a continuous pseudo-time .cls phenotype accepted by GSEA without format errors (via mat2txt/vec2cls/mat2cls outputs)
- Ranking table matches the reported ordering in Fig. 4a / Supplementary Table 1 (Cyclum > reCAT, PC1, PC2)

## Limitations

- Evaluation depends on the quality and relevance of the chosen gene set (GO cell-cycle set); results may not generalize to other periodic processes
- Linear representations (PC1/PC2) are expected to underperform on circular trajectories, so the benchmark is skewed toward nonlinear methods like Cyclum by design
- GSEA phenotypes must be continuous pseudo-time orderings; discrete phase labels require a different downstream treatment

## Evidence

- [results] On the five melanoma samples, Cyclum's inferred pseudo-time received the highest GSEA cell-cycle enrichment scores, outperforming reCAT, PC1, and PC2, including on the low-cycling sample Mel79.: [evidence span withheld — anchor status: not_found]
- [readme] vec2cls writes phenotypes to a GSEA compatible .cls file: [evidence span withheld — anchor status: external_doc]
- [methods] scaled, centered, log2(x+1) transformation: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum outperformed the model-free approach reCAT: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
