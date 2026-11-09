---
name: simulated-cite-seq-data-generation
description: Use when you need synthetic CITE-seq ADT data with known cluster-level
  variation structure to evaluate how a purity metric or clustering parameter (e.g.,
  LinQ-View's rank parameter) behaves as the number of surface-protein features changes.
when_to_use_negative:
- Your goal is to evaluate real biological CITE-seq data (e.g., the 8,617-cell GSE100866
  CBMC dataset) rather than controlled simulation of purity-score behavior
- The dataset has a large number of ADT features — the paper states LinQ-View is best
  suited to limited ADT feature counts (e.g., <50), so simulations testing its rank
  parameter should stay in that regime
- You need a clustering-accuracy benchmark against other methods (Seurat 4, MOFA2)
  rather than a controlled test of a single parameter
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LinQ-View
  role: ADT purity scoring procedure whose rank parameter (swept 1-5) is evaluated
    on the simulated datasets
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: R v3.6.3
  role: Runtime environment in which the simulations and purity scoring are executed
  repo: https://www.r-project.org/
- name: dplyr v1.0.2
  role: Tabulates per-cluster purity scores for every simulated dataset and rank combination
  repo: https://dplyr.tidyverse.org/
- name: ggplot2 v3.3.2
  role: Produces the Figure 3-style comparison plot of purity scores across datasets
    and ranks
  repo: https://ggplot2.tidyverse.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:e03f688922f04e4f429d463bb4a7a7f474f0245e7e0a66ce4fde04f83136c55a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R v3.6.3
  - dplyr
  - dplyr v1.0.2
  - ggplot2
  - ggplot2 v3.3.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# simulated-cite-seq-data-generation

## Summary

Generates simulated CITE-seq datasets with a controlled number of ADT variation features (9, 18, 27) across 6 clusters, using per-cluster per-feature means and SDs from Table S7, so that the rank-dependent behavior of the LinQ-View ADT purity score can be benchmarked independently of real-data confounders.

## When to use

Use when you need synthetic CITE-seq ADT data with known cluster-level variation structure to evaluate how a purity metric or clustering parameter (e.g., LinQ-View's rank parameter) behaves as the number of surface-protein features changes.

## When NOT to use

- Your goal is to evaluate real biological CITE-seq data (e.g., the 8,617-cell GSE100866 CBMC dataset) rather than controlled simulation of purity-score behavior
- The dataset has a large number of ADT features — the paper states LinQ-View is best suited to limited ADT feature counts (e.g., <50), so simulations testing its rank parameter should stay in that regime
- You need a clustering-accuracy benchmark against other methods (Seurat 4, MOFA2) rather than a controlled test of a single parameter

## Inputs

- Per-cluster per-feature means and SDs for simulated ADT features (Table S7 of the LinQ-View paper)
- Target number of variation features per simulation (9, 18, or 27)
- Fixed cluster count (6 clusters)
- Rank sweep range for purity scoring (1-5)

## Outputs

- Simulated CITE-seq datasets with 6 clusters and 9, 18, or 27 ADT variation features
- adt_purity_scores.csv — per-cluster ADT purity scores tabulated for every dataset and rank
- figure3_adt_scores.png — ggplot2 comparison figure reproducing the Figure 3 layout

## How to apply

Build simulated CITE-seq datasets containing 6 clusters, drawing ADT feature values from the per-cluster per-feature means and SDs listed in Table S7 of the LinQ-View paper, scaling the number of variation features to 9, 18, and 27 per dataset. Run the LinQ-View ADT purity scoring procedure in R v3.6.3 on each simulation, sweeping the adjustable rank parameter over 1-5, since the paper shows optimal rank shifts upward with feature count. Tabulate per-cluster purity scores for every dataset-rank combination using dplyr v1.0.2, then plot score comparisons with ggplot2 v3.3.2 following the Figure 3 layout. Judge correctness by whether the best-performing rank range shifts from 2-3 (9 features) to 3-4 (18 features) to 4-5 (27 features), and whether the single-feature high-variance cluster (cluster 3) scores below cluster 2 while clusters 1, 2, 4, 5, 6 decrease with variation-feature count.

## Related tools

- **LinQ-View** (ADT purity scoring procedure whose rank parameter (swept 1-5) is evaluated on the simulated datasets) — https://github.com/WilsonImmunologyLab/LinQView
- **R v3.6.3** (Runtime environment in which the simulations and purity scoring are executed) — https://www.r-project.org/
- **dplyr v1.0.2** (Tabulates per-cluster purity scores for every simulated dataset and rank combination) — https://dplyr.tidyverse.org/
- **ggplot2 v3.3.2** (Produces the Figure 3-style comparison plot of purity scores across datasets and ranks) — https://ggplot2.tidyverse.org/

## Evaluation signals

- Best-performing rank range is 2-3 on the 9-feature simulation, 3-4 on the 18-feature simulation, and 4-5 on the 27-feature simulation, matching Figure 3
- Simulated cluster score ordering: clusters 1, 2, 4, 5, 6 decrease with variation-feature count across datasets
- Cluster 3 (high variance on a single feature) scores below cluster 2 in every simulation
- adt_purity_scores.csv contains a per-cluster score row for all 3 simulated datasets x 5 rank values (6 clusters each)
- figure3_adt_scores.png renders with ggplot2 v3.3.2 and mirrors the Figure 3 layout

## Limitations

- Per-cluster per-feature means and SDs needed for the simulations are only provided in supplemental Table S7, not in the main text
- LinQ-View's pairwise distance computation scales quadratically with cell count, limiting time efficiency for large simulated datasets
- The rank parameter is dataset-size dependent: the optimal range shifts with the number of ADT features, so scores from one feature count cannot be directly reused to pick ranks for another

## Evidence

- [methods] Generate simulated CITE-seq datasets containing 6 clusters with 9, 18, and 27 variation features using the per-cluster per-feature means and SDs listed in Table S7: [evidence span withheld — anchor status: not_found]
- [results] ADT scoring performance depends on rank relative to feature count: [evidence span withheld — anchor status: not_found]
- [discussion] Means and SDs for each cluster and each feature has been listed in Table S7: "Means and SDs for each cluster and each feature has been listed in Table S7"
- [discussion] LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50): "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
