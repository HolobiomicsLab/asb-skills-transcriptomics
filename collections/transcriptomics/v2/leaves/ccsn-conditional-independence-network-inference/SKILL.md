---
name: ccsn-conditional-independence-network-inference
description: Use when a normalized scRNA-seq gene expression matrix must be transformed
  into per-cell gene-gene association networks where indirect (non-direct) edges are
  filtered out via conditional-independence hypothesis tests, e.g. before computing
  NFE or CNDM-based analyses.
when_to_use_negative:
- The analysis goal requires causal (directional) gene regulatory networks — c-CSN
  is not designed for causal inference and does not orient gene associations
- Bulk RNA-seq or other data without per-cell resolution, since the method's core
  output is one direct-association network per individual cell
- Datasets where pairwise gene expression distributions are too sparse to estimate
  scatter-diagram neighborhoods reliably in 3D expression space
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: 'Core method: builds the conditional cell-specific network (CCSN) for each
    cell by conditional-independence testing of gene pairs'
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:655b1b36824d36fe2e458b739edd858cf564ef4c7195080556087a5f624a4dc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ccsn-conditional-independence-network-inference

## Summary

Constructs a conditional cell-specific network (CCSN) for every cell in a scRNA-seq dataset by testing conditional independence of each gene pair given one or more conditional genes, so that only direct gene-gene associations are retained. This per-cell network (or its degree matrix, CNDM) can then feed clustering, dimension reduction, or network flow entropy (NFE) potency estimation.

## When to use

Use when a normalized scRNA-seq gene expression matrix must be transformed into per-cell gene-gene association networks where indirect (non-direct) edges are filtered out via conditional-independence hypothesis tests, e.g. before computing NFE or CNDM-based analyses.

## When NOT to use

- The analysis goal requires causal (directional) gene regulatory networks — c-CSN is not designed for causal inference and does not orient gene associations
- Bulk RNA-seq or other data without per-cell resolution, since the method's core output is one direct-association network per individual cell
- Datasets where pairwise gene expression distributions are too sparse to estimate scatter-diagram neighborhoods reliably in 3D expression space

## Inputs

- Normalized single-cell RNA-seq gene expression matrix (genes x cells), e.g. the Wang NPC differentiation dataset (483 cells, Day 0-30) or the Yang mouse hepatoblast dataset (E10 vs E17)
- Set of conditional genes (e.g. top-degree hub genes selected from an initial CSN)

## Outputs

- Per-cell CCSN adjacency networks (one conditional association network per cell per conditional gene)
- Conditional network degree matrix (CNDM): m x n matrix of cijk gene-gene interaction degrees across cells
- Downstream NFE scores when edges are weighted (pij proportional to xi*xj) and row-normalized into stochastic matrix P

## How to apply

First estimate the joint probability distribution of gene pairs by scatter-diagram neighborhoods in 3D expression space, then run the conditional-independence hypothesis test for every gene pair (x, y) given a conditional gene z, rejecting H0 when the normalized statistic exceeds the standard-normal quantile at the chosen significance level. Select conditional genes from the top-degree hub ('importance') genes of an initial CSN (G >= 1 conditions, noting each conditional gene multiplies runtime by ~G with no parallel scheme in the published implementation). For each cell k, combine per-condition networks into CCSN Czg(k) and average them into the cell-specific gene-gene interaction degree matrix cijk; stacking these over cells yields the conditional network degree matrix (CNDM), which can be used for PCA/t-SNE, clustering, or, when edges are weighted by pij proportional to xi*xj and row-normalized into stochastic matrix P, for NFE = -sum(i,j) xi*pij*log(xi*pij). Rationale: plain CSN overestimates associations by including indirect edges; conditioning eliminates them, producing one direct-association network per cell.

## Related tools

- **c-CSN** (Core method: builds the conditional cell-specific network (CCSN) for each cell by conditional-independence testing of gene pairs) — https://github.com/LinLi-0909/c-CSN

## Evaluation signals

- Each conditional-independence test rejects H0 only when the normalized statistic exceeds the standard-normal quantile at the chosen significance level
- The per-cell output is exactly one direct-association network per cell (and per conditional gene), averaged into cijk degree values with no indirect edges retained
- For the Wang NPC dataset, downstream NFE Wilcoxon test comparing Day 0/Day 1 progenitors vs Day 30 neurons should reproduce P = 2.062E-12 (P = 3.756E-19 for Yang E10 vs E17)
- CNDM-derived clustering should outperform gene-expression-matrix (GEM) clustering, e.g. higher ARI than GEM on the Kim/Buettner/Kolodziejczyk/Chu-time benchmark datasets
- Runtime scales roughly with G (number of conditional genes); a single conditional gene should keep per-pair testing tractable for ~hundreds of cells

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel scheme is desired
- c-CSN is not designed to construct causal gene association networks, and the directions of gene associations cannot be obtained
- Plain CSN (without conditioning) overestimates gene-gene associations by including indirect associations, so results depend on careful selection of conditional genes

## Evidence

- [abstract] we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell: "we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell"
- [intro] overestimation rationale: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [methods] hypothesis test: "we first use CSN or Equation (3) to distinguish the independence of gene x and gene y and we then use the following hypothesis test"
- [methods] conditional gene selection: [evidence span withheld — anchor status: not_found]
- [supplementary] availability: "CCSN is available at https://github.com/LinLi-0909/c-CSN."
