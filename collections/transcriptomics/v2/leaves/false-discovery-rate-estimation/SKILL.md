---
name: false-discovery-rate-estimation
description: Use when a SingleCellGGM GGM network has been computed (~20,000 iterations)
  and edge cutoffs (pcor >= 0.03, co-expression in >= 10 cells) must be validated
  by a permutation-based FDR estimate before network clustering.
when_to_use_negative:
- The expression matrix has fewer than ~5,000 cells or has not been filtered for low-expression
  genes, since SingleCellGGM pcor sampling and permutation statistics will not be
  reliable
- Only raw counts are available — counts must first be log-normalized (log2(counts/total
  * 10000 + 1)) before FDR estimation
- The goal is cell-type annotation of a novel dataset with existing GEPs (use CellGEP
  instead; FDR estimation is part of reference network construction)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SingleCellGGM
  role: Computes pairwise partial correlation coefficients (pcors) over ~20,000 iterations;
    fdr_control permutes its input genes to estimate the FDR of its significant edges
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: fdr_control
  role: MATLAB function that re-runs SingleCellGGM on permuted genes and tabulates
    FDR per pcor cutoff
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: adjust_cutoff
  role: MATLAB function to apply the FDR-supported pcor cutoff (default 0.03) and
    co-expressed cell cutoff (default 10) to the network
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:75f822a8468f54fe38513a0b8db018b46b8a6742a5237ade3ea9c55e78266b32
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/SingleCellGGM
  tools:
  - MCL
  - SingleCellGGM
  - adjust_cutoff
  - fdr_control
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# false-discovery-rate-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the gene-pair false discovery rate of a SingleCellGGM co-expression network by re-running SingleCellGGM on gene-permuted expression data and comparing significant edges against the empirical network. This yields the per-cutoff FDR that justifies retaining edges (e.g., pcor >= 0.03 in >= 10 cells) before MCL GEP clustering.

## When to use

Use when a SingleCellGGM GGM network has been computed (~20,000 iterations) and edge cutoffs (pcor >= 0.03, co-expression in >= 10 cells) must be validated by a permutation-based FDR estimate before network clustering.

## When NOT to use

- The expression matrix has fewer than ~5,000 cells or has not been filtered for low-expression genes, since SingleCellGGM pcor sampling and permutation statistics will not be reliable
- Only raw counts are available — counts must first be log-normalized (log2(counts/total * 10000 + 1)) before FDR estimation
- The goal is cell-type annotation of a novel dataset with existing GEPs (use CellGEP instead; FDR estimation is part of reference network construction)

## Inputs

- log-normalized single-cell gene expression matrix (double or sparse, cells in rows, genes in columns)
- SingleCellGGM result object (ggm) containing SigEdges, pcor_all, coexpressed_cell_num, and RoundNumber
- permutation_fraction parameter (default 1) controlling the fraction of genes permutated

## Outputs

- fdr_control object with a fdr.fdr table (Pcor, SigEdgeNum, FDR, SigPermutatedEdgeNum, SigPermutatedEdgeProportion) across 91 pcor cutoffs
- adjusted SingleCellGGM network object with FDR-supported edges (e.g., 127,229 gene pairs for MCA at FDR 4.17E-05)
- tab-delimited edge list (e.g., mca.ggm.network.txt) for GGM network construction

## How to apply

After running SingleCellGGM on the log-normalized, gene-filtered expression matrix (cells in rows, genes in columns, double/sparse format, low-expression genes removed if expressed in < 10 cells), call fdr_control(expression_matrix, ggm, permutation_fraction) with permutation_fraction defaulting to 1, which permutes genes and re-scores edges at each pcor cutoff. Inspect fdr.fdr, a table with columns Pcor, SigEdgeNum, FDR, SigPermutatedEdgeNum, and SigPermutatedEdgeProportion, and select the pcor cutoff giving an acceptable FDR; for the MCA dataset (61,637 cells, 24,802 Ensembl-ID genes), pcor >= 0.03 and co-expression in >= 10 cells gave ~127,229 gene pairs at FDR 4.17E-05. Optionally use adjust_cutoff(ggm, 0.03, 10) to apply the chosen cutoffs to the network object. The rationale is that permutation destroys true co-expression structure, so edges surviving permutation quantify how many reported edges are expected by chance. Write surviving edges to a tab-delimited network file (e.g., mca.ggm.network.txt) for downstream MCL clustering into GEPs.

## Related tools

- **SingleCellGGM** (Computes pairwise partial correlation coefficients (pcors) over ~20,000 iterations; fdr_control permutes its input genes to estimate the FDR of its significant edges) — https://github.com/MaShisongLab/SingleCellGGM
- **fdr_control** (MATLAB function that re-runs SingleCellGGM on permuted genes and tabulates FDR per pcor cutoff) — https://github.com/MaShisongLab/SingleCellGGM
- **adjust_cutoff** (MATLAB function to apply the FDR-supported pcor cutoff (default 0.03) and co-expressed cell cutoff (default 10) to the network) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
fdr = fdr_control(expression_matrix, ggm); fdr.fdr
```

## Evaluation signals

- The fdr.fdr table is produced and contains the expected columns: Pcor, SigEdgeNum, FDR, SigPermutatedEdgeNum, SigPermutatedEdgeProportion
- For the MCA dataset, applying pcor >= 0.03 and co-expression in >= 10 cells yields ~127,229 gene pairs at FDR 4.17E-05
- The SigEdges table dimension matches the expected edge count (e.g., 127229x9 table with Ensembl gene IDs such as ENSMUSG*)
- The written edge-list file (e.g., mca.ggm.network.txt) exists and contains the surviving gene pairs for downstream MCL clustering
- The permutation-based SigPermutatedEdgeProportion at the chosen cutoff is consistent with the reported FDR (near-zero for MCA at pcor 0.03)

## Limitations

- FDR estimates depend on the number of iterations (default ~20,000) and permutation_fraction (default 1); reduced permutations change precision
- Edges are only supported at a specific pcor cutoff — adjusting the cutoff (e.g., to 0.02 via adjust_cutoff) changes the FDR and must be re-evaluated
- SingleCellGGM relies exclusively on transcriptome data and neglects other single-cell modalities (ATAC-seq, single-cell proteomics)

## Evidence

- [readme] fdr_control call signature: [evidence span withheld — anchor status: external_doc]
- [readme] cutoff semantics of edge selection: [evidence span withheld — anchor status: external_doc]
