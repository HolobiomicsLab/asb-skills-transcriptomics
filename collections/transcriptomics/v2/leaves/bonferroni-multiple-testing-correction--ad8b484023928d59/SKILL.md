---
name: bonferroni-multiple-testing-correction
description: Use when a large family of statistical tests (e.g., 1641 motif enrichment
  Wilcoxon tests over a gene set) must be jointly thresholded, and raw p-values would
  otherwise yield inflated false positives.
when_to_use_negative:
- Only one or very few hypotheses are tested, where the m-fold penalty is unnecessary
  and only costs power
- Discovery-oriented analyses where controlling family-wise error rate is too conservative
  and FDR (Benjamini-Hochberg) control is the appropriate criterion
- P-values that are not independent or are the result of dependent post-hoc comparisons
  the correction was not designed to cover
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MIRA
  role: Provides the regulatory potential models and probabilistic in silico deletion
    scores whose per-motif Wilcoxon p-values are Bonferroni-corrected across 1641
    JASPAR PWMs
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Technical-effect correction whose augmentation changes the corrected enrichment
    p-values being compared
  repo: https://github.com/cistrome/MIRA
- name: JASPAR 2020 vertebrate PWM collection
  role: Defines the family of 1641 PWM hypotheses over which the Bonferroni correction
    is applied
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:e8ef96a4eaae80ab52f661f04ac208ae0683b113bc974e2559f5f595aa682e46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - JASPAR 2020 vertebrate PWM collection
  - MIRA
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bonferroni-multiple-testing-correction

## Summary

Applies Bonferroni correction to control family-wise error rate when testing many hypotheses simultaneously, as in this workflow where Wilcoxon motif-enrichment p-values are computed across 1641 JASPAR 2020 vertebrate PWMs and corrected before comparing unaugmented versus CODAL-augmented MIRA regulatory potential models.

## When to use

Use when a large family of statistical tests (e.g., 1641 motif enrichment Wilcoxon tests over a gene set) must be jointly thresholded, and raw p-values would otherwise yield inflated false positives.

## When NOT to use

- Only one or very few hypotheses are tested, where the m-fold penalty is unnecessary and only costs power
- Discovery-oriented analyses where controlling family-wise error rate is too conservative and FDR (Benjamini-Hochberg) control is the appropriate criterion
- P-values that are not independent or are the result of dependent post-hoc comparisons the correction was not designed to cover

## Inputs

- Vector of raw p-values, one per hypothesis (1641 JASPAR 2020 vertebrate PWM enrichment Wilcoxon p-values)
- Number of tests m (1641 PWMs) defining the Bonferroni multiplier

## Outputs

- Bonferroni-adjusted p-values (min(p * m, 1)) per motif
- Comparison table of corrected p-values between unaugmented and CODAL-augmented models (fig4d_motif_enrichment_table.csv)
- Set of significant motifs passing the corrected threshold (e.g., TAL1-GATA1, GATA family, TAL1-TCF3 complex motifs)

## How to apply

After per-motif p-values are computed (here, Wilcoxon tests of each motif's probabilistic in silico deletion association scores between the 200 topic-4 proerythroblast genes and the 2641-gene background), multiply each raw p-value by the number of tested hypotheses (m = 1641 PWMs) and cap at 1.0. Compare the corrected p-values between conditions — in this study, regulatory potential models trained with and without CODAL technical effects fixed as an additive, non-trainable negative-binomial component — to identify motifs whose enrichment significance changes with correction. Finally, write the full corrected comparison to a results table (fig4d_motif_enrichment_table.csv). Bonferroni is conservative; the 3-orders-of-magnitude significance gain for TAL1-GATA1 with CODAL correction survived this strict family-wise control, strengthening the finding.

## Related tools

- **MIRA** (Provides the regulatory potential models and probabilistic in silico deletion scores whose per-motif Wilcoxon p-values are Bonferroni-corrected across 1641 JASPAR PWMs) — https://github.com/cistrome/MIRA
- **CODAL** (Technical-effect correction whose augmentation changes the corrected enrichment p-values being compared) — https://github.com/cistrome/MIRA
- **JASPAR 2020 vertebrate PWM collection** (Defines the family of 1641 PWM hypotheses over which the Bonferroni correction is applied)

## Examples

```
from statsmodels.stats.multitest import multipletests; p_bonf = multipletests(raw_pvals, alpha=0.05, method='bonferroni')[1]
```

## Evaluation signals

- Adjusted p-value must equal min(raw p-value * 1641, 1.0) for every PWM — recompute a few rows by hand to verify
- No adjusted p-value can be smaller than its raw p-value, and all adjusted values lie in [0, 1]
- The correction must be applied over the same family size reported in the workflow (1641 JASPAR 2020 vertebrate PWMs)
- Codas-augmented TAL1-GATA1 enrichment should remain significant after correction, with reported ~3-orders-of-magnitude significance increase versus the uncorrected model
- Output table fig4d_motif_enrichment_table.csv should contain both raw and Bonferroni-corrected p-values for both model conditions

## Limitations

- Bonferroni is conservative when the 1641 motif tests are correlated (motif PWMs overlap in sequence specificity), reducing power relative to FDR-based methods
- The authors caution that perturbation/batch findings, even when significant, should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- Significance alone does not establish biological relevance — corrected motifs should still be checked against known lineage regulators (e.g., TAL1/GATA1 in proerythroblasts)

## Evidence

- [other] Bonferroni-correct the p-values across the 1641 PWMs, compare corrected enrichment p-values between the unaugmented and CODAL-augmented regulatory potential models: [evidence span withheld — anchor status: not_found]
- [other] Assess enrichment of each motif's association scores among the 200 topic-4 genes versus all other genes using a Wilcoxon test.: [evidence span withheld — anchor status: not_found]
- [methods] using the JASPAR79 2020 vertebrate position weight matrix (PWM) collection: "using the JASPAR79 2020 vertebrate position weight matrix (PWM) collection"
- [readme] All MIRA topic models use the new CODAL algorithm for inference.: [evidence span withheld — anchor status: external_doc]
