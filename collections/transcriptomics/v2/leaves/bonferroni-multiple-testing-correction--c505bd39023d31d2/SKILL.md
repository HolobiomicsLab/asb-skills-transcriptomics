---
name: bonferroni-multiple-testing-correction
description: Use when many module-level or gene-set-level statistics (e.g., dozens
  of leiden-derived GRN module scores) are tested against the same phenotype and individual
  uncorrected p-values would inflate the false positive rate.
when_to_use_negative:
- Only a single pre-specified hypothesis is tested, so no multiplicity correction
  is needed
- The analysis goal is to control expected false discoveries rather than any false
  positive occurring (an FDR procedure such as Benjamini-Hochberg is the appropriate
  choice, not family-wise Bonferroni)
- Module p-values are not comparable across the same family of tests (e.g., different
  phenotypes with unrelated nulls) without redefining the correction family
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: Provides the merged, edge-pruned GRN whose leiden-partitioned modules are
    the hypothesis family being Bonferroni-corrected
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: AUCell scoring produces the per-spot module expression scores that are correlated
    with plaque burden and age
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:8e35774dca67f960c2a59b7571b9ba6013ca90240e400294c4c0022e71638fd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bonferroni-multiple-testing-correction

## Summary

Apply Bonferroni family-wise correction when testing many SCING GRN modules simultaneously for association with a phenotype (e.g., amyloid beta plaque burden and age in mouse AD Visium data), so that per-module Pearson correlation p-values are judged against the number of modules tested.

## When to use

Use when many module-level or gene-set-level statistics (e.g., dozens of leiden-derived GRN module scores) are tested against the same phenotype and individual uncorrected p-values would inflate the false positive rate.

## When NOT to use

- Only a single pre-specified hypothesis is tested, so no multiplicity correction is needed
- The analysis goal is to control expected false discoveries rather than any false positive occurring (an FDR procedure such as Benjamini-Hochberg is the appropriate choice, not family-wise Bonferroni)
- Module p-values are not comparable across the same family of tests (e.g., different phenotypes with unrelated nulls) without redefining the correction family

## Inputs

- Per-module AUCell score matrix (33 leiden-partitioned SCING GRN modules x Visium spots)
- Spot-level beta-amyloid plaque burden annotations from GSE152506 spot metadata
- Mouse age values (3-18 months) per spot
- Empirical null distribution of Pearson correlations from 1,000 random gene sets per module

## Outputs

- Bonferroni-corrected p-values for each module's plaque-burden and age correlations
- List of significantly phenotype-associated modules (e.g., 6 significant plaque-associated modules, with module 25 as the top hit)
- Figure 6-style summaries of module spatial maps and correlation significance

## How to apply

First score module-specific expression per spatial spot with AUCell (from SCENIC) after partitioning the merged SCING GRN into 33 leiden subnetworks. For each module, compute a Pearson correlation of the module score with the spot-level beta-amyloid plaque burden annotation (and with mouse age, 3-18 months), and calibrate significance against an empirical null built from 1,000 random gene sets of matched size. Collect the correlation p-values across all modules and apply the Bonferroni correction (multiply each p-value by the number of modules, or compare against alpha/number_of_tests) to control the family-wise error rate; in the source analysis, 6 of the 33 SCING modules remained significantly plaque-associated after this correction. Report corrected p-values alongside module pathway annotations (e.g., microglial activation, lysosome, cell migration for module 25) to judge whether surviving modules reflect expected AD biology.

## Related tools

- **SCING** (Provides the merged, edge-pruned GRN whose leiden-partitioned modules are the hypothesis family being Bonferroni-corrected) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (AUCell scoring produces the per-spot module expression scores that are correlated with plaque burden and age)

## Examples

```
from statsmodels.stats.multitest import multipletests; reject, p_bonf, _, _ = multipletests(module_pvals, alpha=0.05, method='bonferroni')
```

## Evaluation signals

- The number of tests in the correction equals the number of leiden modules (33 in the source analysis); the Bonferroni threshold should be alpha/33, not alpha/number_of_genes
- Significant modules must survive both the permutation null of 1,000 random gene sets AND the Bonferroni correction; uncorrected hits that disappear after correction should be excluded
- Top surviving module (module 25) should match expected AD biology (microglial activation, lysosome, cell migration enrichment), consistent with plaque pathogenesis
- The reported count of significant associations (6 modules for plaque burden) should be reproducible when re-running Pearson correlations on AUCell scores with the same seed/null sets

## Limitations

- Bonferroni is conservative; genuinely weaker module-phenotype associations may be discarded when 33 modules are tested, especially given only ~6 survive in the source analysis
- SCING GRN links are not necessarily causal or directional, so corrected module-phenotype associations indicate regional covariation, not causal regulation
- The null of 1,000 random gene sets plus Bonferroni controls family-wise error but hyperparameter and dataset-dependent network construction choices propagate into the tested module set

## Evidence

- [other] Compute Pearson correlations of each module's expression with plaque burden and age against a null of 1,000 random gene sets, applying Bonferroni correction for multiple testing.: [evidence span withheld — anchor status: not_found]
- [other] Six SCING modules were significantly associated with amyloid beta plaque burden via Pearson correlation against a permutation null.: [evidence span withheld — anchor status: not_found]
- [methods] We combine these subnetworks with the AUCell method to get module specific expression for each cell.: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [readme] These can be used for clustering or phenotypic association.: "These can be used for clustering or phenotypic association."
