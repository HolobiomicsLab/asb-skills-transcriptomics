---
name: clock-controlled-gene-overlap-analysis
description: Use when you have a called cycler gene set (e.g., Tempo de novo cyclers
  per cell type) and want to test its overlap with a reference gene set (clock genes
  or bulk JTKCycle q<0.05 cyclers) with empirical p-values.
when_to_use_negative:
- Your cycler gene sets were not derived from Tempo de novo calls or phases fixed
  to sample-collection times, so the enrichment comparison is not meaningful
- You lack a defined gene universe (the assayed gene space) from which to draw the
  1000 null gene sets
- The question is phase estimation accuracy itself, not gene-set overlap enrichment
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Tempo
  role: source of the de novo cycler gene sets under enrichment testing
  repo: github.com/bauerbach95/tempo
- name: anndata
  role: loads annotated cell-by-gene single-cell data
  repo: github.com/bauerbach95/tempo
- name: pandas
  role: loads gene lists and writes fig6_enrichment_results.csv
  repo: github.com/bauerbach95/tempo
- name: numpy
  role: samples the 1000 random gene sets for the empirical null
  repo: github.com/bauerbach95/tempo
- name: scipy
  role: computes hypergeometric/Fisher-style enrichment statistics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:6a04fa580fec24623137c2c610e6157abecc59f94e634f281b0b482f7b1b8db3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Tempo
  - anndata
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# clock-controlled-gene-overlap-analysis

## Summary

Tests whether Tempo-called de novo cycler gene sets (from unsupervised single-cell phase inference) are enriched for circadian clock genes or for cyclers called with phases fixed to sample-collection times, using hypergeometric-style enrichment statistics with empirical null calibration from 1000 random gene sets.

## When to use

Use when you have a called cycler gene set (e.g., Tempo de novo cyclers per cell type) and want to test its overlap with a reference gene set (clock genes or bulk JTKCycle q<0.05 cyclers) with empirical p-values.

## When NOT to use

- Your cycler gene sets were not derived from Tempo de novo calls or phases fixed to sample-collection times, so the enrichment comparison is not meaningful
- You lack a defined gene universe (the assayed gene space) from which to draw the 1000 null gene sets
- The question is phase estimation accuracy itself, not gene-set overlap enrichment

## Inputs

- Annotated cell-by-gene scRNA-seq matrix (anndata) for aorta SMCs, fibroblasts, endothelial cells, macrophages, and liver hepatocytes
- Tempo-called de novo cycler gene lists per cell type (from phases fixed to sample-collection times)
- Bulk JTKCycle results from Zhang et al. aorta and liver datasets
- Curated circadian clock gene set

## Outputs

- Observed enrichment statistics and empirical p-values per cell type (de novo) and per bulk dataset
- fig6_enrichment_results.csv (written with pandas)

## How to apply

Load Tempo-called de novo cyclers per real cell type (189 aorta SMC, 109 fibroblast, 87 endothelial, 28 macrophage, 117 hepatocyte genes) from annotated cell-by-gene data using pandas/anndata. Define bulk cycler sets as genes with q < 0.05 in JTKCycle results from Zhang et al. aorta and liver datasets. For each observed cycler set, compute the overlap with the circadian clock gene set using scipy's hypergeometric/Fisher enrichment statistic. Build an empirical null by sampling 1000 random gene sets of the same size from the assayed gene universe with numpy, recomputing the statistic per draw. Convert each observed statistic to an empirical p-value as its rank/quantile within the null distribution, and write results per cell type and bulk to fig6_enrichment_results.csv with pandas.

## Related tools

- **Tempo** (source of the de novo cycler gene sets under enrichment testing) — github.com/bauerbach95/tempo
- **anndata** (loads annotated cell-by-gene single-cell data) — github.com/bauerbach95/tempo
- **pandas** (loads gene lists and writes fig6_enrichment_results.csv) — github.com/bauerbach95/tempo
- **numpy** (samples the 1000 random gene sets for the empirical null) — github.com/bauerbach95/tempo
- **scipy** (computes hypergeometric/Fisher-style enrichment statistics)

## Examples

```
import pandas as pd, numpy as np; from scipy.stats import hypergeom; p = hypergeom.sf(k-1, M, n, len(cyclers)); null = [hypergeom.sf(np.random.choice(M, len(cyclers), replace=False), M, n, k).min() for _ in range(1000)]
```

## Evaluation signals

- Observed cycler set sizes match the paper values: 189 (SMC), 109 (fibroblast), 87 (endothelial), 28 (macrophage), 117 (hepatocyte)
- Empirical p-values fall within [0, 1] and are computed as the observed statistic's rank/quantile among exactly 1000 null draws
- Null gene sets are sampled only from the assayed gene universe and have the same size as each observed cycler set
- Bulk cycler sets contain only genes with JTKCycle q < 0.05
- fig6_enrichment_results.csv contains one row per test with enrichment statistic and empirical p-value for both de novo and bulk sets per cell type

## Limitations

- Simulated analyses suggest the de novo cycler issue may stem from assuming 24 h sinusoidal CCG expression means; more flexible waveform modeling is needed
- No explicit software version for Tempo is stated, so reproducibility of the cycler calls depends on the repository state
- Empirical p-values are quantized by the 1000-draw null resolution

## Evidence

- [results] Tempo-called de novo cyclers enriched for cyclers called with phases fixed to sample collection times: [evidence span withheld — anchor status: not_found]
- [results] Cycler set sizes per cell type: [evidence span withheld — anchor status: not_found]
- [methods] Tempo package dependencies include anndata, numpy, pandas, scipy: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Tempo installation and environment activation from README: "git clone https://github.com/bauerbach95/tempo"
- [abstract] De novo cyclers identified fully unsupervised: "Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
