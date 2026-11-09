---
name: rank-parameter-sweep-evaluation
description: Use when tuning the LinQ-View ADT purity score's rank hyperparameter,
  or when evaluating how a cluster-scoring metric's parameter behaves across datasets
  with different surface-protein (ADT) feature counts.
when_to_use_negative:
- Massive datasets such as 170K cells with 228 ADT features, where LinQ-View (and
  its purity scoring) failed while Seurat 4 succeeded
- CITE-seq datasets with a large number of ADT features (e.g., >=50), since LinQ-View
  is documented as best suited for large datasets with a limited number of ADT features
  (<50)
- Analyses that need per-cluster transcriptional entropy rather than surface-protein
  purity — the ROGUE score targets cluster uniformity in scRNA-seq, not the ADT modality
  rank sweep
tools:
- name: LinQ-View
  role: Provides the ADT purity score whose rank parameter is swept; runs the L-infinity-norm
    multimodal distance fusion and sigmoid-filtered ADT distance calculation
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: R v3.6.3
  role: Runtime environment in which the simulations, purity scoring, and plots are
    executed
  repo: https://www.r-project.org/
- name: dplyr
  role: Tabulates per-cluster purity scores for every dataset and rank combination
    into the score table
  repo: https://dplyr.tidyverse.org/
- name: ggplot2
  role: Plots the dataset-rank score comparisons reproducing the Figure 3 layout
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:d384057b0303d620493ff0e3c8692e0b426e6492de6cb32a6f96c16455cf4f22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R v3.6.3
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rank-parameter-sweep-evaluation

## Summary

Sweeps the adjustable 'rank' parameter (1-5) of the LinQ-View ADT purity score across CITE-seq datasets with different numbers of ADT features to identify the rank range that best quantifies cluster variation.

## When to use

Use when tuning the LinQ-View ADT purity score's rank hyperparameter, or when evaluating how a cluster-scoring metric's parameter behaves across datasets with different surface-protein (ADT) feature counts.

## When NOT to use

- Massive datasets such as 170K cells with 228 ADT features, where LinQ-View (and its purity scoring) failed while Seurat 4 succeeded
- CITE-seq datasets with a large number of ADT features (e.g., >=50), since LinQ-View is documented as best suited for large datasets with a limited number of ADT features (<50)
- Analyses that need per-cluster transcriptional entropy rather than surface-protein purity — the ROGUE score targets cluster uniformity in scRNA-seq, not the ADT modality rank sweep

## Inputs

- GSE100866 CBMC CITE-seq dataset (8,617 cord blood mononuclear cells) downloaded from NCBI GEO
- Simulated CITE-seq datasets with 6 clusters and 9, 18, or 27 variation features, built from per-cluster per-feature means and SDs in Table S7
- Rank parameter values to sweep (1-5)

## Outputs

- adt_purity_scores.csv — per-cluster ADT purity score table for every dataset and rank
- figure3_adt_scores.png — ggplot2 figure reproducing Figure 3 showing optimal rank shifting with ADT feature count and cluster score ordering

## How to apply

Download the GSE100866 CBMC CITE-seq dataset (8,617 cord blood mononuclear cells) from NCBI GEO and load it into R v3.6.3. Generate simulated CITE-seq datasets containing 6 clusters with 9, 18, and 27 variation features using the per-cluster per-feature means and SDs from Table S7 (these are only in the supplement, not the main text). Run the LinQ-View ADT purity scoring procedure — which fuses pairwise distance matrices with an L-infinity norm and applies a sigmoid filter to the ADT modality distances — on each dataset while sweeping rank over 1-5. Tabulate per-cluster purity scores for every dataset-rank combination with dplyr v1.0.2 and plot comparisons with ggplot2 v3.3.2, reproducing the Figure 3 layout. The decision point is the best-performing rank range relative to ADT feature count: ranks 2-3 for 9 features, 3-4 for 18 features, and 4-5 for 27 features, so the optimal rank shifts upward as the number of ADT features grows.

## Related tools

- **LinQ-View** (Provides the ADT purity score whose rank parameter is swept; runs the L-infinity-norm multimodal distance fusion and sigmoid-filtered ADT distance calculation) — https://github.com/WilsonImmunologyLab/LinQView
- **R v3.6.3** (Runtime environment in which the simulations, purity scoring, and plots are executed) — https://www.r-project.org/
- **dplyr** (Tabulates per-cluster purity scores for every dataset and rank combination into the score table) — https://dplyr.tidyverse.org/
- **ggplot2** (Plots the dataset-rank score comparisons reproducing the Figure 3 layout) — https://ggplot2.tidyverse.org/

## Evaluation signals

- Best-performing rank range is 2-3 on the 9-feature simulation
- Best-performing rank range is 3-4 on the 18-feature simulation
- Best-performing rank range is 4-5 on the 27-feature simulation
- The simulated cluster with high variance on a single feature (cluster 3) scores below cluster 2
- Cluster score ordering shows clusters 1,2,4,5,6 decreasing with variation-feature count, matching the Figure 3 layout

## Limitations

- Per-cluster per-feature means and SDs needed to regenerate the simulations are only in Table S7, not in the main text
- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time efficiency on large datasets
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features where Seurat 4 succeeded

## Evidence

- [intro] L-infinity fusion of modalities underlies the purity score being parameterized: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [intro] The purity score quantifies cluster purity in both modalities of CITE-seq data: "we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data"
- [discussion] The ADT modality distances passed to the purity score are sigmoid filtered: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] Simulation parameters required for the sweep live only in the supplement: "Means and SDs for each cluster and each feature has been listed in Table S7"
- [discussion] Upper bound on applicable ADT feature count: "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
- [readme] LinQ-View's stated purpose for CITE-seq multimodal integration: "Integrate multiple modalities of CITE-seq data for single cell populations identification"
