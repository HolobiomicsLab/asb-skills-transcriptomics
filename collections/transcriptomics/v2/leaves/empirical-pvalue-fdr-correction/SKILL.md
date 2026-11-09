---
name: empirical-pvalue-fdr-correction
description: Use when you have many gene signatures scored for phylogenetic autocorrelation
  (e.g., Geary's C on a KNN graph of cophenetic distances) and need multiple-testing-corrected
  significance before reporting which signatures track the phylogeny.
when_to_use_negative:
- Input is a single pre-specified hypothesis test without a family of many signatures
  — no multiple-testing correction is needed
- The goal is unguided discovery of heritable gene modules rather than significance-testing
  user-defined signatures (use Hotspot module identification instead)
- Signatures were not evaluated against an empirical null (randomly generated signatures),
  so naive BH correction over unrelated nominal tests would not match the PhyloVision
  procedure
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PhyloVision
  role: Runs the signature-centric phylogenetic autocorrelation analysis whose empirical
    p-values are BH-corrected to FDR; reports and filters signatures at FDR < 0.05
  repo: https://github.com/YosefLab/VISION
- name: Hotspot
  role: compute_autocorrelations function produces per-gene statistics with C, Z,
    Pval, and FDR columns analogous to the signature-level correction
  repo: https://github.com/YosefLab/Hotspot
- name: R
  role: Implementation language for the PhyloVision pipeline including hypergeometric
    and multiple-testing statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:ebce251dfc10769dce74dba7700388e5b011263d5b471e0c60da6825271130bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - Hotspot
  - PhyloVision
  - R
  - VISION
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# empirical-pvalue-fdr-correction

## Summary

Assigns significance to phylogenetic autocorrelation of gene signature scores by testing each signature's Geary's C statistic against a null of randomly generated signatures, then controlling the false discovery rate with Benjamini-Hochberg correction so signatures can be reported at FDR < 0.05.

## When to use

Use when you have many gene signatures scored for phylogenetic autocorrelation (e.g., Geary's C on a KNN graph of cophenetic distances) and need multiple-testing-corrected significance before reporting which signatures track the phylogeny.

## When NOT to use

- Input is a single pre-specified hypothesis test without a family of many signatures — no multiple-testing correction is needed
- The goal is unguided discovery of heritable gene modules rather than significance-testing user-defined signatures (use Hotspot module identification instead)
- Signatures were not evaluated against an empirical null (randomly generated signatures), so naive BH correction over unrelated nominal tests would not match the PhyloVision procedure

## Inputs

- Per-signature Geary's C (C' = 1 - C) statistics from PhyloVision phylogenetic autocorrelation analysis
- Empirical p-values from comparison against randomly generated signatures
- Full set of evaluated MSigDB signatures (Hallmark, C2, C5 BP) supplied as .gmt files

## Outputs

- signature_autocorrelation_table.csv containing a C' statistic in [0,1] and an FDR for every evaluated signature
- Collapsable report table retaining only signatures with FDR < 0.05 (Benjamini-Hochberg corrected)

## How to apply

In the PhyloVision signature-centric analysis, cell-level signature Z-scores are evaluated with Geary's C (reported as C' = 1 - C) on a KNN graph built from cophenetic distances (num_neighbors = 30). Empirical p-values are derived by comparing each observed statistic against randomly generated signatures, and are then Benjamini-Hochberg corrected to FDR values. In the related unguided Hotspot workflow, per-gene autocorrelations computed via compute_autocorrelations likewise return a table with C, Z, Pval, and FDR columns. Downstream filtering retains only signatures with FDR < 0.05 in the reported collapsable table, and all C' statistics and FDR values are exported (e.g., signature_autocorrelation_table.csv) regardless of significance.

## Related tools

- **PhyloVision** (Runs the signature-centric phylogenetic autocorrelation analysis whose empirical p-values are BH-corrected to FDR; reports and filters signatures at FDR < 0.05) — https://github.com/YosefLab/VISION
- **Hotspot** (compute_autocorrelations function produces per-gene statistics with C, Z, Pval, and FDR columns analogous to the signature-level correction) — https://github.com/YosefLab/Hotspot
- **R** (Implementation language for the PhyloVision pipeline including hypergeometric and multiple-testing statistics)

## Examples

```
hs_results = hs.compute_autocorrelations()  # returns DataFrame with C, Z, Pval, FDR columns
```

## Evaluation signals

- Every signature in signature_autocorrelation_table.csv has both a C' statistic in [0,1] (1 = perfect autocorrelation) and an FDR value
- Reported collapsable table contains only signatures with FDR < 0.05
- Number of BH-corrected entries equals the number of evaluated signatures (no signatures silently dropped before correction)
- Hotspot compute_autocorrelations output DataFrame contains the expected columns C, Z, Pval, and FDR
- Benjamini-Hochberg FDR values are monotone with nominal empirical p-values and bounded in [0,1]

## Limitations

- Empirical p-values depend on the null of randomly generated signatures, so significance reflects comparison to random signatures rather than a parametric model
- The article notes its metastasis-dataset analysis 'still lacks experimental validation to support the generated hypotheses', so corrected significance alone does not establish biology

## Evidence

- [other] Autocorrelation p-values BH-corrected against random signatures: [evidence span withheld — anchor status: not_found]
- [results] Geary's C autocorrelation of signature scores: [evidence span withheld — anchor status: not_found]
- [methods] Hotspot compute_autocorrelations used for autocorrelation statistics: "using the “compute_autocorrelations” function in Hotspot"
- [readme] Hotspot output includes FDR column: [evidence span withheld — anchor status: external_doc]
