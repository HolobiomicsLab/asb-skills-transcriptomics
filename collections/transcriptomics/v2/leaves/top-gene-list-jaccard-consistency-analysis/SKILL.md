---
name: top-gene-list-jaccard-consistency-analysis
description: Use when comparing DE gene lists from multiple scRNA-seq DE methods (e.g.,
  MAST, edgeR, zingeR, iDEA) to measure cross-method consistency of ranked DE calls
  at a fixed list size.
when_to_use_negative:
- When only one DE method was run — the Jaccard index requires at least two gene lists
  to compare
- When assessing per-gene rank correlation or concordance across the full ranked list
  rather than overlap of a fixed-size top set (use rank-based measures instead)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Augments per-method DE summary statistics via iDEA.fit and iDEA.BMA to produce
    gene lists whose cross-method Jaccard consistency is then compared against the
    standalone results
  repo: https://github.com/xzhoulab/iDEA
- name: MAST
  role: Produces gene-level DE summary statistics (beta coefficient and variance)
    whose top-1500 gene list is one input to the Jaccard comparison
- name: edgeR
  role: Produces ZINB-WaVE-weighted gene-level DE summary statistics whose top-1500
    gene list is one input to the Jaccard comparison
- name: zingeR
  role: Produces gene-level DE summary statistics (via the zinbwaveZinger workflow)
    whose top-1500 gene list is one input to the Jaccard comparison
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:f4bd16c73f2a588ecb191185762cee4c748cc9f27ee43847f920e2038fc871f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CreateiDEAObject
  - MAST
  - edgeR
  - iDEA
  - iDEA.BMA
  - iDEA.fit
  - zingeR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# top-gene-list-jaccard-consistency-analysis

## Summary

Quantify the reproducibility/consistency of differential expression results across methods by computing the Jaccard index of their top-ranked DE gene lists (e.g., top-1500 genes), as used in the iDEA paper to show iDEA-augmented DE calls are more consistent than standalone MAST, edgeR, and zingeR calls.

## When to use

Use when comparing DE gene lists from multiple scRNA-seq DE methods (e.g., MAST, edgeR, zingeR, iDEA) to measure cross-method consistency of ranked DE calls at a fixed list size.

## When NOT to use

- When only one DE method was run — the Jaccard index requires at least two gene lists to compare
- When assessing per-gene rank correlation or concordance across the full ranked list rather than overlap of a fixed-size top set (use rank-based measures instead)

## Inputs

- Per-method ranked DE gene lists or per-gene summary statistics (beta coefficient and its variance) from MAST, edgeR (ZINB-WaVE weighted), and/or zingeR
- iDEA-augmented DE results from iDEA.fit + iDEA.BMA on each method's summary statistics
- A fixed list size cutoff (e.g., top-1500 genes)

## Outputs

- Pairwise Jaccard index matrix of top-1500 DE gene lists between DE methods
- Comparison of Jaccard indices before vs. after iDEA augmentation (expected 0.59 -> 0.77 in the tau1=5, CR=10% simulation)

## How to apply

For each DE method, rank genes by their DE significance (e.g., FDR-adjusted p-values from the method's summary statistics) and take the top-ranked set (the iDEA simulations used the top-1500 genes). Compute the pairwise Jaccard index — |intersection| / |union| — between the top-gene lists of each method pair. To assess whether an integrative method improves consistency, recompute the Jaccard indices after augmenting each method's summary statistics with iDEA (via CreateiDEAObject with max_var_beta=100 and min_precent_annot=0.0025, then iDEA.fit with init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, and iDEA.BMA to aggregate DE evidence). In the iDEA paper's simulation setting (tau1=5, CR=10%), the Jaccard index of top-1500 DE gene lists increased from 0.59 (standalone methods) to 0.77 (iDEA-augmented), which serves as the expected validation benchmark.

## Related tools

- **iDEA** (Augments per-method DE summary statistics via iDEA.fit and iDEA.BMA to produce gene lists whose cross-method Jaccard consistency is then compared against the standalone results) — https://github.com/xzhoulab/iDEA
- **MAST** (Produces gene-level DE summary statistics (beta coefficient and variance) whose top-1500 gene list is one input to the Jaccard comparison)
- **edgeR** (Produces ZINB-WaVE-weighted gene-level DE summary statistics whose top-1500 gene list is one input to the Jaccard comparison)
- **zingeR** (Produces gene-level DE summary statistics (via the zinbwaveZinger workflow) whose top-1500 gene list is one input to the Jaccard comparison) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
idea_obj <- iDEA.BMA(iDEA.fit(CreateiDEAObject(beta=beta, var_beta=var_beta, annotation=genesets, max_var_beta=100, min_precent_annot=0.0025), fit_noGS=FALSE, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000)); jac <- length(intersect(top1500_a, top1500_b)) / length(union(top1500_a, top1500_b))
```

## Evaluation signals

- In the tau1=5, CR=10% simulation, Jaccard of top-1500 DE gene lists between standalone methods should be around 0.59
- After iDEA augmentation, the top-1500 list Jaccard index should increase to around 0.77
- All gene lists must be truncated to the same fixed size (top-1500) before computing Jaccard, and the same ranking statistic must be used per method
- Jaccard values must lie in [0,1]; a decrease after iDEA augmentation indicates a pipeline error relative to the reported result

## Limitations

- The Jaccard index depends on the arbitrary choice of top-list size (here top-1500); results may differ at other cutoffs
- iDEA does not explicitly account for gene set overlap that may cause non-independence, and z-score modeling effectively assumes the prior distribution of true effect sizes depends on standard errors and sample size — both could affect DE rankings in edge cases
- DE analysis on inferred cell types may yield inflated DE test statistics with artificially smaller standard errors, distorting the top-gene rankings being compared

## Evidence

- [other] the top-1500 DE gene list Jaccard index should increase from 0.59 to 0.77 when iDEA is applied: [evidence span withheld — anchor status: not_found]
- [abstract] iDEA can improve the power and consistency of DE analysis and the accuracy of GSE analysis: "By integrating DE and GSE analyses, iDEA can improve the power and consistency of DE analysis and the accuracy of GSE analysis."
- [intro] iDEA models all genes together by borrowing information across genes: "iDEA models all genes together by borrowing information across genes in terms of DE effect size distributional properties."
- [methods] The iDEA object is created by the function CreateiDEAObject.: "The iDEA object is created by the function CreateiDEAObject."
