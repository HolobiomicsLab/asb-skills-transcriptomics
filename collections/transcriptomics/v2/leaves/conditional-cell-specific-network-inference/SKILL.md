---
name: conditional-cell-specific-network-inference
description: Use when you need per-sample (per-cell) direct gene-gene association
  networks from an expression matrix, e.g. for sample-level pathway connectivity such
  as EGFR degree density in TCGA-LUAD tumor vs normal.
when_to_use_negative:
- You need causal gene-gene association networks or directionality of gene associations
  — c-CSN is not designed for this and is left as future work
- You only need a single population-level gene-gene association network rather than
  per-cell/per-sample networks with indirect associations removed
- Input is single-cell clustering/trajectory output (e.g. SC3, SIMLR, Monocle results)
  rather than a raw or normalized gene expression matrix
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: c-CSN
  role: Core method that constructs the conditional cell-specific network (CCSN) for
    each cell by conditional independence testing on gene pairs
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:7fa02241889adfd4c4bb8055fc552ca8cd1b7990be563353887bc346dc7a24ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# conditional-cell-specific-network-inference

## Summary

Constructs a conditional cell-specific network (CCSN) for each individual cell/sample from scRNA-seq or bulk RNA-seq data by testing conditional independence of gene pairs given top-degree 'importance' genes, eliminating the indirect associations that make plain CSN overestimate gene–gene connectivity.

## When to use

Use when you need per-sample (per-cell) direct gene-gene association networks from an expression matrix, e.g. for sample-level pathway connectivity such as EGFR degree density in TCGA-LUAD tumor vs normal.

## When NOT to use

- You need causal gene-gene association networks or directionality of gene associations — c-CSN is not designed for this and is left as future work
- You only need a single population-level gene-gene association network rather than per-cell/per-sample networks with indirect associations removed
- Input is single-cell clustering/trajectory output (e.g. SC3, SIMLR, Monocle results) rather than a raw or normalized gene expression matrix

## Inputs

- Normalized scRNA-seq or bulk RNA-seq gene expression matrix (cells/samples × genes), e.g. TCGA LUAD RNA-seq data from the GDC portal
- Significance level α for the conditional independence hypothesis test
- Number of conditional genes GG (top-degree 'importance' genes selected from the CSN)

## Outputs

- One CCSN (direct gene-gene association network) per cell/sample
- Cell-specific gene-gene interaction degree matrix C-k integrated over the G conditional genes
- Conditional network degree matrix (CNDM), an m × n matrix suitable for dimension reduction, clustering, or NFE computation

## How to apply

For each cell (or sample), estimate the conditional independence statistic ρxy|z from scatter-diagram neighborhoods in 3D expression space, then test each gene pair at significance level α against the Nα quantile of the standard normal distribution to decide whether to draw a direct edge. Select the top GG largest 'importance' (top-degree hub) genes from the CSN as the conditional genes z, and build a CCSN Czg(k) for cell k given each conditional gene zg; integrate the G per-gene CCSNs into the cell-specific gene-gene interaction degree matrix C-k = (1/G)Σ Czg(k), yielding the conditional network degree matrix (CNDM). This removes indirect associations that inflate plain CSN networks, producing robust per-cell network data; the resulting CNDM can feed standard downstream analyses (PCA/t-SNE, clustering, NFE), or individual CCSNs can be queried directly (e.g. EGF/EGFR degrees for EGFR pathway density). Code is available at https://github.com/LinLi-0909/c-CSN.

## Related tools

- **c-CSN** (Core method that constructs the conditional cell-specific network (CCSN) for each cell by conditional independence testing on gene pairs) — https://github.com/LinLi-0909/c-CSN

## Examples

```
from c_csn import ccsn; A = ccsn(expr_matrix, alpha=0.05, G=10)  # one direct-association network per cell
```

## Evaluation signals

- Each input cell yields exactly one direct association network (one CCSN per cell), and integrating the G per-gene CCSNs yields a degree matrix C-k = (1/G)Σ Czg(k)
- The c-CSN network is sparser than a plain CSN because indirect associations are eliminated (CSN overestimates gene-gene associations by including both direct and indirect links)
- Gene pairs are accepted as edges only when the conditional independence statistic ρxy|z is significant at level α relative to the Nα quantile of the standard normal distribution
- The resulting CNDM supports standard downstream analyses: PCA/t-SNE dimension reduction and clustering (K-means, hierarchical, K-medoids, SC3, SIMLR, Seurat) with higher ARI than the gene expression matrix on benchmark datasets
- For the TCGA-LUAD EGFR application, EGF/EGFR CCSN degrees should be consistently higher across tumor samples than normal samples, reproducing dense EGFR pathway connectivity in tumors but not benign tissues

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel computation scheme is desired
- c-CSN is not designed to construct causal gene association networks, and directions of gene associations cannot be obtained
- The hypothesis test treats each gene pair independently of the rest of the network

## Evidence

- [abstract] one network per cell: "we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell"
- [abstract] direct associations only: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [intro] CSN overestimation motivation: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [methods] conditional gene selection: [evidence span withheld — anchor status: not_found]
- [methods] degree matrix construction: [evidence span withheld — anchor status: not_found]
- [discussion] parallelization limitation: "a parallel computation scheme is desired to reduce the computation time"
