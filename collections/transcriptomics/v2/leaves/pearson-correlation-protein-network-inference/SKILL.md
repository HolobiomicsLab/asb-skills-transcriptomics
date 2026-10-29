---
name: pearson-correlation-protein-network-inference
description: Use when you need a plain co-expression/PCC baseline network on a protein-by-condition
  matrix (tsv/csv, e.g. HPA 566,109-cell scRNA-seq or PRIDE bulk proteomics) to benchmark
  against a learned method like FAVA.
when_to_use_negative:
- The goal is the highest-quality network itself, not a baseline — the article shows
  FAVA's latent-space correlations outperform plain co-expression 'by a wide margin'
  (>90% precision, >5000 true positives with <500 false positives on HPA vs KEGG).
- The input is already a dense, low-dimensional latent representation — at that point
  simple correlation is exactly FAVA's contribution; applying PCC on the original
  high-dimensional data is what fails.
- The matrix is small and dense with no sparseness/redundancy concerns, where the
  runtime and accuracy disadvantages of brute-force PCC are less relevant.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implementation language for computing the plain PCC baseline and sweeping
    the correlation cut-off on the same matrices used for FAVA
- name: favapy
  role: Produces the FAVA networks that PCC is benchmarked against on HPA scRNA-seq
    and PRIDE proteomics data
  repo: https://github.com/mikelkou/fava
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:b37dcc2e29dfd24eeb71092f1519903bf3b54cbd6d934160b005d3eaf5fcfd0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - Python
  - favapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pearson-correlation-protein-network-inference

## Summary

Infers protein functional association networks via plain Pearson correlation (PCC) on gene/protein-by-condition matrices, serving as the classical co-expression baseline that FAVA is benchmarked against on Human Protein Atlas scRNA-seq and PRIDE proteomics data.

## When to use

Use when you need a plain co-expression/PCC baseline network on a protein-by-condition matrix (tsv/csv, e.g. HPA 566,109-cell scRNA-seq or PRIDE bulk proteomics) to benchmark against a learned method like FAVA.

## When NOT to use

- The goal is the highest-quality network itself, not a baseline — the article shows FAVA's latent-space correlations outperform plain co-expression 'by a wide margin' (>90% precision, >5000 true positives with <500 false positives on HPA vs KEGG).
- The input is already a dense, low-dimensional latent representation — at that point simple correlation is exactly FAVA's contribution; applying PCC on the original high-dimensional data is what fails.
- The matrix is small and dense with no sparseness/redundancy concerns, where the runtime and accuracy disadvantages of brute-force PCC are less relevant.

## Inputs

- Gene/protein-by-condition expression matrix in tsv/csv (Human Protein Atlas scRNA-seq, 566,109 cells)
- Protein-by-condition bulk proteomics matrix (PRIDE, 633 experiments, 18,846 proteins)
- KEGG functional association reference set for benchmarking

## Outputs

- Ranked list of protein-protein interaction pairs per correlation cut-off (swept from +1 to -1)
- Precision-vs-cut-off curve benchmarked against KEGG functional associations
- Wall-clock runtime profile of PCC on each dataset (e.g. ~9 days on HPA scRNA-seq)

## How to apply

Compute Pearson correlation coefficients between all protein pairs directly on the original high-dimensional expression matrix — the same tsv/csv matrix fed to FAVA (Human Protein Atlas scRNA-seq with 566,109 cells; PRIDE bulk proteomics with 633 experiments and 18,846 proteins). Rank the resulting protein-protein pairs by correlation and sweep the Pearson cut-off across its full range from 1 (high correlation) to -1 (high anti-correlation), generating a ranked list of interactions for each cut-off. Benchmark the ranked pairs against KEGG functional associations and plot precision curves so PCC can be compared head-to-head with FAVA on both datasets. Also record the wall-clock runtime: on the HPA scRNA-seq matrix PCC takes ~9 days versus 1 day for FAVA (a ≥9x difference), because the VAE latent-space compression of FAVA avoids the brute-force correlation cost. The rationale for keeping this as a baseline is that the paper shows raw co-expression suffers from sparseness and redundancy of omics data, so PCC is expected to be outperformed by latent-space correlation.

## Related tools

- **Python** (Implementation language for computing the plain PCC baseline and sweeping the correlation cut-off on the same matrices used for FAVA)
- **favapy** (Produces the FAVA networks that PCC is benchmarked against on HPA scRNA-seq and PRIDE proteomics data) — https://github.com/mikelkou/fava

## Examples

```
import numpy as np; pcc = np.corrcoef(X, rowvar=False)  # plain PCC baseline on the same tsv/csv matrix passed to favapy
```

## Evaluation signals

- Precision curve against KEGG is produced for the full cut-off sweep from +1 to -1 on both HPA and PRIDE matrices.
- PCC precision on the HPA scRNA-seq curve should fall well below FAVA's benchmark of >5000 true positives with <500 false positives (>90% precision).
- The FAVA curve outperforms PCC 'by a wide margin' — any reversal indicates a benchmarking error (e.g. wrong matrix, wrong KEGG mapping).
- Runtime comparison must show PCC taking ~9 days vs FAVA's 1 day on the HPA matrix (≥9x faster for FAVA).
- The same tsv/csv input matrix is used for both PCC and FAVA so the comparison is controlled.

## Limitations

- Plain co-expression analysis 'does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy (many similar cells/samples are analyzed)' of omics data.
- Computationally expensive at scale: ~9 days wall-clock on the 566,109-cell HPA scRNA-seq matrix, versus 1 day for FAVA's VAE-based compression.
- Pearson correlation on raw data can yield anti-correlation links (cutoff sweep extends to -1), whose functional validity is weaker than positive co-expression.

## Evidence

- [intro] Inferring networks from omics data via simple co-expression analysis does not work well due to high sparseness and redundancy: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy"
- [intro] Latent-space correlations outperform correlations on the original data representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [abstract] FAVA outperforms existing network-inference methods including co-expression baselines: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods"
