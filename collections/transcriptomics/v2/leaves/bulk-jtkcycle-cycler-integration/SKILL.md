---
name: bulk-jtkcycle-cycler-integration
description: Use when comparing a cell-type-specific cycler gene set (e.g., 189 SMC,
  109 fibroblast, 87 endothelial, 28 macrophage, 117 hepatocyte de novo cyclers) to
  an independent bulk cycler reference to quantify overlap enrichment with empirical
  p-values.
when_to_use_negative:
- The cycler sets being compared are derived from the same phase-fixing procedure
  (circular overlap; no independent reference is being tested)
- Input lacks a defined gene universe or q-values for the bulk JTKCycle results, making
  the q < 0.05 cycler call and hypergeometric denominator uncomputable
- The goal is phase inference itself (use Tempo) rather than post-hoc cycler-set enrichment
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pandas
  role: Load annotated cell-by-gene data and cycler gene lists; write observed enrichment
    statistics and empirical p-values to fig6_enrichment_results.csv
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Read the annotated single-cell cell-by-gene matrix from which de novo cycler
    sets are taken
  repo: https://github.com/bauerbach95/tempo
- name: scipy
  role: Compute the hypergeometric/Fisher-style enrichment statistic for observed
    and null gene-set overlaps
  repo: https://github.com/bauerbach95/tempo
- name: numpy
  role: Sample the 1000 random gene sets of matched size from the assayed gene universe
    to build the empirical null
  repo: https://github.com/bauerbach95/tempo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:24796b1d7ee2ca0b7112006dfb9a872707ea15d3ef41b20805d8288e13422252
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Tempo
  - anndata
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# bulk-jtkcycle-cycler-integration

## Summary

Tests whether Tempo's fully unsupervised de novo cycler sets from scRNA-seq cell types (aorta SMC, fibroblast, endothelial, macrophage; liver hepatocyte) are enriched for bulk JTKCycle cyclers (q < 0.05 from Zhang et al. aorta/liver data), using a hypergeometric/Fisher-style enrichment statistic calibrated against an empirical null of 1000 random gene sets.

## When to use

Use when comparing a cell-type-specific cycler gene set (e.g., 189 SMC, 109 fibroblast, 87 endothelial, 28 macrophage, 117 hepatocyte de novo cyclers) to an independent bulk cycler reference to quantify overlap enrichment with empirical p-values.

## When NOT to use

- The cycler sets being compared are derived from the same phase-fixing procedure (circular overlap; no independent reference is being tested)
- Input lacks a defined gene universe or q-values for the bulk JTKCycle results, making the q < 0.05 cycler call and hypergeometric denominator uncomputable
- The goal is phase inference itself (use Tempo) rather than post-hoc cycler-set enrichment

## Inputs

- Tempo-called de novo cycler gene lists per real cell type (SMC n=189, fibroblast n=109, endothelial n=87, macrophage n=28, hepatocyte n=117), from annotated cell-by-gene data (anndata/pandas)
- Bulk JTKCycle results from Zhang et al. aorta and liver datasets (gene-level q-values)
- Circadian clock gene set
- Assayed gene universe (all genes assayed in the single-cell data)

## Outputs

- fig6_enrichment_results.csv containing observed enrichment statistics and empirical p-values for both de novo (per cell type) and bulk cycler sets
- Empirical null distributions of the enrichment statistic (1000 random gene-set draws per test)

## How to apply

Load the Tempo-called de novo cyclers per cell type (defined with phases fixed to sample-collection times) from the annotated cell-by-gene data via pandas/anndata, and load bulk JTKCycle results from the Zhang et al. aorta and liver datasets, defining bulk cyclers as genes with q < 0.05. For each cycler set (per cell type and bulk), compute the observed overlap with the circadian clock gene set using a scipy hypergeometric/Fisher-style statistic. Build an empirical null by drawing 1000 random gene sets of the same size from the assayed gene universe with numpy, recomputing the statistic for each, and convert the observed statistic to an empirical p-value as its rank/quantile within that null distribution. Rationale: enrichment against a sampled null avoids parametric assumptions and is the approach used in Fig. 6 to confirm that unsupervised de novo cyclers agree with collection-time-fixed cyclers across all evaluated real cell types.

## Related tools

- **pandas** (Load annotated cell-by-gene data and cycler gene lists; write observed enrichment statistics and empirical p-values to fig6_enrichment_results.csv) — https://github.com/bauerbach95/tempo
- **anndata** (Read the annotated single-cell cell-by-gene matrix from which de novo cycler sets are taken) — https://github.com/bauerbach95/tempo
- **scipy** (Compute the hypergeometric/Fisher-style enrichment statistic for observed and null gene-set overlaps) — https://github.com/bauerbach95/tempo
- **numpy** (Sample the 1000 random gene sets of matched size from the assayed gene universe to build the empirical null) — https://github.com/bauerbach95/tempo

## Examples

```
from scipy.stats import hypergeom; import pandas as pd; p = hypergeom.sf(k-1, M, n, N); pd.DataFrame(results).to_csv('fig6_enrichment_results.csv')
```

## Evaluation signals

- Bulk cycler set exactly equals genes with JTKCycle q < 0.05 in the Zhang et al. aorta/liver result tables
- Each observed cycler set size matches the reported counts (189 SMC, 109 fibroblast, 87 endothelial, 28 macrophage, 117 hepatocyte)
- Each null distribution contains exactly 1000 statistics, each computed on a random gene set of the same size as the observed cycler set, drawn only from the assayed gene universe
- Empirical p-values fall in [1/1000, 1] (rank/quantile within a 1000-draw null) and were computed against the correct matched null
- fig6_enrichment_results.csv contains one row per test (per cell type and bulk) with both the observed statistic and empirical p-value; positive result is de novo cyclers enriched for fixed-phase cyclers in all evaluated cell types

## Limitations

- Simulated analyses suggest de novo cycler calls may be affected by the assumption that CCG expression means follow 24 h sinusoidal patterns; more flexible waveform modeling would be needed
- Tempo as described does not explicitly model technical effects, which becomes more necessary for data collected as multiple samples over time
- Subpopulation-specific gene amplitude variation is not explicitly modeled

## Evidence

- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] To test if Tempo works properly, run the run_test.py file using using the activated environment: "To test if Tempo works properly, run the run_test.py file using using the activated environment:
python run_test.py"
- [abstract] we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
