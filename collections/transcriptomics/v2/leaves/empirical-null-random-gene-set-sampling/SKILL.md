---
name: empirical-null-random-gene-set-sampling
description: Use when testing whether an unsupervised or bulk-derived gene set (e.g.,
  cyclers with q < 0.05) is enriched for a reference set such as circadian clock genes,
  without a parametric enrichment model.
when_to_use_negative:
- When a standard parametric enrichment test (e.g., Fisher's exact test p-value alone)
  is sufficient and gene-set independence assumptions hold
- When the gene set of interest comes from a supervised differential-expression contrast
  rather than an unsupervised de novo call whose overlap with reference sets is the
  question
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: numpy
  role: Sample 1000 random gene sets of the observed set size from the assayed gene
    universe to build the null distribution
  repo: github.com/bauerbach95/tempo
- name: scipy
  role: Compute the hypergeometric/Fisher-style overlap enrichment statistic for observed
    and null gene sets
- name: pandas
  role: Load cycler sets and bulk JTKCycle results; write fig6_enrichment_results.csv
- name: anndata
  role: Load annotated cell-by-gene scRNA-seq data containing the Tempo-called de
    novo cycler sets
  repo: github.com/bauerbach95/tempo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:d2bfafd38ab6918f857ecab45b3774dccc49645772e96429913fb05c1907e632
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Tempo
  - anndata
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# empirical-null-random-gene-set-sampling

## Summary

Assess whether a gene set (e.g., Tempo's de novo cyclers called unsupervised on scRNA-seq) is enriched for circadian clock genes by comparing the observed overlap statistic to an empirical null built from 1000 randomly sampled gene sets of the same size drawn from the assayed gene universe.

## When to use

Use when testing whether an unsupervised or bulk-derived gene set (e.g., cyclers with q < 0.05) is enriched for a reference set such as circadian clock genes, without a parametric enrichment model.

## When NOT to use

- When a standard parametric enrichment test (e.g., Fisher's exact test p-value alone) is sufficient and gene-set independence assumptions hold
- When the gene set of interest comes from a supervised differential-expression contrast rather than an unsupervised de novo call whose overlap with reference sets is the question

## Inputs

- Cell-by-gene scRNA-seq data in anndata format with annotated Tempo-called cycler gene sets per cell type (SMC, fibroblast, endothelial, macrophage, hepatocyte)
- Bulk JTKCycle result tables from Zhang et al. aorta and liver datasets (cyclers defined as q < 0.05)
- Curated circadian clock gene reference set
- Assayed gene universe (all genes measured in the dataset)

## Outputs

- Observed enrichment statistics for de novo and bulk cycler sets per cell type
- Empirical p-values computed as rank/quantile within the 1000-draw null distribution
- fig6_enrichment_results.csv written with pandas

## How to apply

Load the observed gene set — for Fig. 6 this means the Tempo-called de novo cyclers per real cell type (189 aorta SMC, 109 fibroblast, 87 endothelial, 28 macrophage, 117 hepatocyte, called with phases fixed to sample-collection times) from cell-by-gene anndata objects, plus bulk JTKCycle cyclers (q < 0.05) from the Zhang et al. aorta and liver datasets. Compute the observed overlap between each cycler set and the circadian clock gene set using a hypergeometric/Fisher-style statistic in scipy. Generate the empirical null by drawing 1000 random gene sets of identical size from the assayed gene universe with numpy, recomputing the statistic for each, and convert the observed statistic to an empirical p-value as its rank/quantile within the 1000-draw null distribution. This calibration is essential because parametric p-values may poorly approximate the true overlap distribution when gene sets are correlated or the universe is misspecified.

## Related tools

- **numpy** (Sample 1000 random gene sets of the observed set size from the assayed gene universe to build the null distribution) — github.com/bauerbach95/tempo
- **scipy** (Compute the hypergeometric/Fisher-style overlap enrichment statistic for observed and null gene sets)
- **pandas** (Load cycler sets and bulk JTKCycle results; write fig6_enrichment_results.csv)
- **anndata** (Load annotated cell-by-gene scRNA-seq data containing the Tempo-called de novo cycler sets) — github.com/bauerbach95/tempo

## Examples

```
import numpy as np; from scipy.stats import hypergeom; null = [hypergeom.sf(len(genes & clock)-1, N_universe, len(g), len(clock)) for g in (set(np.random.choice(N_universe, n_obs, replace=False)) for _ in range(1000))]
```

## Evaluation signals

- Observed enrichment statistic for each cycler set falls in the upper tail of its own 1000-draw null distribution
- Empirical p-values are computed as rank/quantile of the observed statistic within exactly 1000 null draws (not fewer)
- Null gene sets are sampled with the same size as the observed cycler set (e.g., 117 for hepatocytes) from the same assayed gene universe
- fig6_enrichment_results.csv contains one row per cell type and bulk dataset with observed statistic and empirical p-value for both tests
- Directionally consistent with the finding: de novo cyclers enriched for collection-time-fixed cyclers across SMC, fibroblast, endothelial, macrophage, and hepatocyte

## Limitations

- Empirical p-values are limited by the number of null draws (rank within 1000 samples caps resolution near ~1/1000)
- Results depend on correctly defining the assayed gene universe; misspecification biases the null overlap distribution
- Simulated analyses in the paper suggest de novo cycler calls may be affected by assuming 24 h sinusoidal CCG expression means, which could propagate into enrichment results

## Evidence

- [other] null sampling workflow: [evidence span withheld — anchor status: not_found]
- [other] bulk cycler definition: [evidence span withheld — anchor status: not_found]
- [readme] Tempo dependencies: "Tempo requires python >= 3.8 and depends on the following python packages: - anndata - numpy - pandas - scanpy>=1.6"
- [abstract] Tempo purpose: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
