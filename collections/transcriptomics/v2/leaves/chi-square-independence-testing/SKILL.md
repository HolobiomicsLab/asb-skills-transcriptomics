---
name: chi-square-independence-testing
description: Use when comparing overlap of two gene-level hit sets (e.g., DGE calls
  from Welch t-tests and DUL calls from scUTRboot WUI tests) and you need to test
  whether their intersection exceeds chance expectation.
when_to_use_negative:
- Gene sets have very small expected cell counts under independence (common for tiny
  hit sets) where Fisher's exact test is more appropriate than chi-square.
- The question is effect size or direction concordance of the same genes rather than
  set membership overlap against a shared background.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: scUTRquant
  role: Generates the SingleCellExperiment gene-level and 3'UTR isoform count matrices
    from which DGE and DUL hit sets are derived before contingency testing.
  repo: https://github.com/Mayrlab/scUTRquant
- name: scUTRboot
  role: Performs the weighted UTR isoform (WUI) tests that define the DUL gene sets
    tested for independence against DGE sets.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:8d7905b8c9c8c17bdc41bb81f5ea6b5829e0f8cfaaf46520c9602a92f17f9b05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chi-square-independence-testing

## Summary

Statistical test used to determine whether two gene sets (e.g., differentially expressed genes and differentially 3'UTR-lengthened genes) overlap more than expected by chance across cell-type comparisons, establishing whether DGE and DUL are independent regulatory axes.

## When to use

Use when comparing overlap of two gene-level hit sets (e.g., DGE calls from Welch t-tests and DUL calls from scUTRboot WUI tests) and you need to test whether their intersection exceeds chance expectation.

## When NOT to use

- Gene sets have very small expected cell counts under independence (common for tiny hit sets) where Fisher's exact test is more appropriate than chi-square.
- The question is effect size or direction concordance of the same genes rather than set membership overlap against a shared background.

## Inputs

- DGE gene set per cell-type pair (|fold change| > 1.5, q < 0.05 from Welch t-tests on gene-level counts)
- DUL gene set per cell-type pair (|dWUI| > 0.10, q < 0.05 from scUTRboot WUI tests on multi-UTR genes)
- Background set of multi-UTR genes co-expressed in both cell types of the pair

## Outputs

- Per-pair DGE-DUL overlap fractions (tabulated, e.g., dge_dul_overlap_fractions.tsv)
- Chi-square test-of-independence results (chi2 statistic and p-value) per cell-type pair

## How to apply

After calling DGE hits (Welch t-tests on aggregated gene-level counts, |fold change| > 1.5 and q < 0.05) and DUL hits (scUTRboot weighted UTR isoform tests, |dWUI| > 0.10 and q < 0.05) on the same background of multi-UTR genes co-expressed in both members of each cell-type pair, build a 2x2 contingency table (DGE +/- vs DUL +/-) per pair and run a chi-square test of independence. In the scUTRquant analysis of 17 cell-type pairs, observed overlap fractions ranged 0-7.4% and were statistically consistent with independence in 14/17 pairs. Report both overlap fractions and chi-square p-values so the user can judge whether DGE and DUL behave as coupled or independent regulatory events for each comparison.

## Related tools

- **scUTRquant** (Generates the SingleCellExperiment gene-level and 3'UTR isoform count matrices from which DGE and DUL hit sets are derived before contingency testing.) — https://github.com/Mayrlab/scUTRquant
- **scUTRboot** (Performs the weighted UTR isoform (WUI) tests that define the DUL gene sets tested for independence against DGE sets.)

## Examples

```
from scipy.stats import chi2_contingency; chi2, p, dof, exp = chi2_contingency([[n_dge_dul, n_dge_only],[n_dul_only, n_neither]])
```

## Evaluation signals

- Each reported DGE-DUL overlap fraction falls within the expected 0-7.4% range observed across the 17 cell-type pairs.
- Chi-square tests of independence are non-significant (independence retained) in the majority of pairs, matching the 14/17 pairs consistent with independence.
- Contingency tables are built on a common background of multi-UTR genes co-expressed in both cell types of each pair, not all detected genes.
- DGE hits satisfy |fold change| > 1.5, q < 0.05; DUL hits satisfy |dWUI| > 0.10, q < 0.05 before overlap computation.

## Limitations

- Chi-square approximation can be unreliable when expected overlap counts are very small, as overlap fractions can be as low as 0%.
- Independence of DGE and DUL sets was not universal: 3 of 17 cell-type pairs showed overlap inconsistent with statistical independence.
- Analysis is restricted to multi-UTR genes, so conclusions do not extend to genes with a single annotated 3'UTR isoform.

## Evidence

- [results] Across pair-wise cell type comparisons, genes with changes in mRNA abundance and genes with changes in 3'UTR length had little overlap (0-7.4%), consistent with statistical independence in 14/17 sample pairs.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow description of the chi-square testing step: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract-level framing of independence finding: "mRNA abundance and mRNA length are two largely independent axes of gene regulation that together determine the amount and spatial organization of protein synthesis."
