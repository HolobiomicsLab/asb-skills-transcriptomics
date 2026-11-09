---
name: rna-velocity-dynamical-model-gene-ranking
description: Use when comparing unsupervised feature-selection methods on a differentiation
  trajectory and you have spliced/unspliced counts so ScVelo's dynamical model can
  produce a likelihood-ranked gene set.
when_to_use_negative:
- The dataset lacks spliced/unspliced count layers, which the ScVelo dynamical model
  requires
- The goal is not a method comparison but a single self-contained ranking, since this
  skill's value in the source workflow is as a velocity-based baseline against DELVE
  and HVG (top 500) feature sets
- Features are proteins from iterative immunofluorescence imaging, where spliced/unspliced
  RNA layers and velocity dynamics are unavailable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ScVelo
  role: Fits the RNA velocity dynamical model and provides likelihood-based gene ranking
    used as the velocity feature-selection baseline
- name: DELVE
  role: Method under comparison; provides the dynamic-seed Laplacian-score ranking
    (top p=500) benchmarked against the velocity ranking
  repo: https://github.com/jranek/delve
- name: Scanpy
  role: Infers the trajectory via diffusion pseudotime (dpt, 20 diffusion components,
    k=10) rooted in day-0 cells for each ranked feature set
- name: Python
  role: Implementation language for the full comparison pipeline (velocity ranking,
    pseudotime regression, TF tabulation)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:2e16f0ff943220a7e706982b0340c97d71c538e943f04139c6ba384bf185db71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - EnrichR
  - Python
  - ScVelo
  - Scanpy
  - anndata
  - gseapy
  - scikit-learn
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rna-velocity-dynamical-model-gene-ranking

## Summary

Ranks genes for a trajectory analysis using the RNA velocity dynamical model from ScVelo, producing a likelihood-based gene ranking that serves as a velocity-based feature-selection baseline. In the DELVE Nature Communications 2024 study it is benchmarked against DELVE and HVG selection on the hESC-to-definitive-endoderm differentiation dataset by counting lineage-specific transcription factors recovered.

## When to use

Use when comparing unsupervised feature-selection methods on a differentiation trajectory and you have spliced/unspliced counts so ScVelo's dynamical model can produce a likelihood-ranked gene set.

## When NOT to use

- The dataset lacks spliced/unspliced count layers, which the ScVelo dynamical model requires
- The goal is not a method comparison but a single self-contained ranking, since this skill's value in the source workflow is as a velocity-based baseline against DELVE and HVG (top 500) feature sets
- Features are proteins from iterative immunofluorescence imaging, where spliced/unspliced RNA layers and velocity dynamics are unavailable

## Inputs

- Preprocessed single-cell RNA-seq .h5ad object with spliced/unspliced count layers (hESC-to-definitive-endoderm dataset: 5397 cells x 2000 genes, CPM normalized, log+1, HVG-filtered)
- AnimalTFDB transcription factor annotations

## Outputs

- Gene ranking by RNA velocity dynamical model likelihood (top 500 selected)
- Lineage-specific significant gene calls (negative binomial GAM, BH q < 0.05) per feature set
- Comparison table of lineage-specific transcription factor counts recovered by DELVE, HVG, and RNA velocity
- Pseudotime traces of key regulators (e.g., SOX2, POU5F1, EOMES, GSC, SOX17, FOXA2) as in Fig. 6h

## How to apply

On the preprocessed hESC-to-definitive-endoderm h5ad object (5397 cells x 2000 genes, CPM normalized, log+1, HVG-filtered, from Zenodo record 10534873), fit the RNA velocity dynamical model with ScVelo and rank all genes by model likelihood. This velocity ranking is generated in parallel with two other 500-gene feature sets: DELVE's dynamic-seed Laplacian-score ranking and the top 500 HVGs. For each method, infer the day-0-rooted trajectory via diffusion pseudotime (Scanpy dpt, 20 diffusion components, k=10), regress each of the 500 genes along pseudotime with a negative binomial GAM, and call genes significant at Benjamini-Hochberg q < 0.05. Annotate significant lineage-specific genes as transcription factors using AnimalTFDB, then compare the TF counts recovered by the velocity ranking against DELVE and HVG. The rationale is that variance-based selection is insensitive to lineage-specific transcription factors with small variance and gradual expression progression, so a velocity-based ranking probes whether a dynamics-aware alternative recovers more regulators.

## Related tools

- **ScVelo** (Fits the RNA velocity dynamical model and provides likelihood-based gene ranking used as the velocity feature-selection baseline)
- **DELVE** (Method under comparison; provides the dynamic-seed Laplacian-score ranking (top p=500) benchmarked against the velocity ranking) — https://github.com/jranek/delve
- **Scanpy** (Infers the trajectory via diffusion pseudotime (dpt, 20 diffusion components, k=10) rooted in day-0 cells for each ranked feature set)
- **Python** (Implementation language for the full comparison pipeline (velocity ranking, pseudotime regression, TF tabulation))

## Examples

```
import scvelo as scv; scv.tl.recover_dynamics(adata); scv.tl.velocity(adata, mode='dynamical'); df = scv.tl.rank_dynamical_genes(adata, groupby='day')
```

## Evaluation signals

- The number of lineage-specific transcription factors recovered from the velocity-ranked genes must be lower than DELVE's by approximately 30%, matching the reported result in Fig. 6g
- Each method's top 500 genes are reduced to a set of pseudotime-significant genes at Benjamini-Hochberg q < 0.05; significance calls should be reproducible across the negative binomial GAM regressions
- Velocity ranking should recover expected definitive-endoderm regulators (e.g., EOMES, GSC, SOX17, FOXA2) on pseudotime traces as in Fig. 6h; absence of these indicates a failed dynamical-model fit
- The ranking uses exactly the same 2000-gene, 5397-cell CPM/log+1 preprocessed h5ad background as the DELVE and HVG baselines, so comparisons are on equal footing

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory, so careful consideration is needed when interpreting the velocity-based ranking
- Variance-based selection (like the HVG baseline this velocity ranking is compared against) is insensitive to lineage-specific features such as transcription factors with small variance and gradual expression progression; the velocity ranking partly addresses but does not eliminate sensitivity limits
- All dynamic modules are leveraged equally in the comparison workflow; the framework does not implement excluding unwanted dynamic modules

## Evidence

- [methods] rank genes by likelihood from the RNA velocity dynamical model in ScVelo: [evidence span withheld — anchor status: not_found]
- [results] the DELVE count of lineage-specific transcription factors must exceed those of HVG selection and RNA velocity by approximately 30%: [evidence span withheld — anchor status: not_found]
- [intro] variance-based selection is insensitive to lineage-specific transcription factors: "they can be overwhelmed by noisy or irrelevant features that dominate data variance, and (2) are insensitive to lineage-specific features (e.g., transcription factors)"
- [readme] twelve feature selection methods were evaluated on trajectory preservation: [evidence span withheld — anchor status: external_doc]
