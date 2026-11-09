---
name: diffusion-pseudotime-estimation
description: Use when a single-cell expression dataset is expected to contain a continuous
  differentiation trajectory and per-cell developmental ordering (pseudotime) is needed,
  e.g., to test whether FFC label entropy is high at transition states and low at
  terminal cell types.
when_to_use_negative:
- Data has no developmental trajectory (e.g., static PBMC immune cell types without
  a progenitor root) — no monotonic pseudotime axis exists to correlate against
- Clustering has produced only one cluster, since Monte Carlo internal validation
  cannot estimate meaningful label uncertainty in that case
- Cell labels are already fully known and confident; the entropy-pseudotime comparison
  is only informative where transition/uncertain states exist
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Preprocesses PBMC count matrices ('zheng17' recipe), computes Leiden clusters
    and diffusion pseudotime rooted at the Procr-expressing cluster
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: Splatter
  role: Generates simulated scRNA-seq differentiation paths in R where each cell carries
    a known pseudotime coordinate
- name: forest-fire-clustering (FFC)
  role: Provides the point-wise label entropies (entropy_list after validate() and
    entropy()) that are correlated against pseudotime
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scipy
  role: Computes Spearman correlation between label entropy and pseudotime per cell
    type/path
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:23a814025816c91d7eb80732d99031026cdd63b3c70cb8f2a6f3760429bd7363
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - R
  - Splatter
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scanpy
  - scipy
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# diffusion-pseudotime-estimation

## Summary

Estimates a diffusion pseudotime ordering of cells along differentiation trajectories, rooted at a progenitor-marker-defined cluster (e.g., Leiden clusters with Procr expression), so that cell-level progression coordinates can be compared against Forest Fire Clustering's point-wise label entropies.

## When to use

Use when a single-cell expression dataset is expected to contain a continuous differentiation trajectory and per-cell developmental ordering (pseudotime) is needed, e.g., to test whether FFC label entropy is high at transition states and low at terminal cell types.

## When NOT to use

- Data has no developmental trajectory (e.g., static PBMC immune cell types without a progenitor root) — no monotonic pseudotime axis exists to correlate against
- Clustering has produced only one cluster, since Monte Carlo internal validation cannot estimate meaningful label uncertainty in that case
- Cell labels are already fully known and confident; the entropy-pseudotime comparison is only informative where transition/uncertain states exist

## Inputs

- Raw PBMC RNA gene expression count matrix (cells x genes) preprocessed with scanpy 'zheng17' recipe
- Splatter-simulated continuous differentiation path scRNA-seq data in R with per-cell pseudotime coordinates
- Leiden cluster assignments and progenitor marker expression (e.g., Procr) for root selection
- Forest Fire Clustering point-wise label entropies (cluster_obj.entropy_list) after FFC preprocess/fit/validate

## Outputs

- Per-cell diffusion pseudotime values rooted at the progenitor Leiden cluster
- Spearman correlation coefficients between entropy_list and pseudotime per dataset/cell type/path
- entropy_pseudotime_correlation.csv with columns for dataset, cell type/path, and correlation value

## How to apply

Preprocess the raw RNA gene expression count matrix with scanpy using the 'zheng17' recipe, then identify Leiden clusters and use the cluster expressing the progenitor marker (e.g., Procr) as the diffusion pseudotime root, as done for the PBMC datasets in the Forest Fire Clustering paper. For Splatter-simulated differentiation paths in R, each cell already carries a known pseudotime coordinate, so diffusion pseudotime is only needed for experimental data. Compute pseudotime for all cells, then per cell type (or per Splatter path) correlate pseudotime against cluster_obj.entropy_list obtained after FFC().preprocess(X), FFC().fit(fire_temp=100), and cluster_obj.validate() using scipy.stats Spearman correlation. Rationale: the paper's finding is that entropy highlights transition states along developmental trajectories, so a valid application should reproduce an inverse entropy-vs-pseudotime relationship within each cell type.

## Related tools

- **scanpy** (Preprocesses PBMC count matrices ('zheng17' recipe), computes Leiden clusters and diffusion pseudotime rooted at the Procr-expressing cluster) — https://github.com/gersteinlab/forest-fire-clustering
- **Splatter** (Generates simulated scRNA-seq differentiation paths in R where each cell carries a known pseudotime coordinate)
- **forest-fire-clustering (FFC)** (Provides the point-wise label entropies (entropy_list after validate() and entropy()) that are correlated against pseudotime) — https://github.com/gersteinlab/forest-fire-clustering
- **scipy** (Computes Spearman correlation between label entropy and pseudotime per cell type/path)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; c=FFC(); c.preprocess(X); c.fit(fire_temp=100); c.validate(); c.entropy(); rho=stats.spearmanr(c.entropy_list, dpt)
```

## Evaluation signals

- Every reported Spearman correlation between label entropy and pseudotime must be negative, mirroring the paper's Figs. 4d-f and 5i finding
- Cells with the highest entropy within each cell type should correspond to progenitor/transition cells identified by diffusion pseudotime
- The correlation table entropy_pseudotime_correlation.csv must contain rows for each Splatter path and each PBMC cell type with a finite correlation value
- Pseudotime root must be a Leiden cluster with progenitor marker expression (e.g., Procr), so pseudotime increases away from that root

## Limitations

- The relationship between entropy and pseudotime is evaluated within each cell type/path; pooling across cell types may obscure the inverse trend
- Diffusion pseudotime depends on root choice — the paper roots at Leiden clusters with Procr marker expression, so an incorrect root invalidates the comparison
- Forest Fire Clustering's Monte Carlo internal validation is unreliable when there is only one cluster, limiting entropy computation on poorly separated data

## Evidence

- [discussion] Procr-rooted pseudotime workflow: "Leiden clusters with Procr marker expression were used as roots for diffusion pseudotime analysis."
- [discussion] PBMC preprocessing recipe: "The RNA gene expression count data for both PBMC datasets were preprocessed using the “zheng17” recipe in the scanpy package"
- [abstract] Entropy highlights developmental transitions: "enable the computation of “label entropies", highlighting transitions along developmental trajectories"
- [readme] README description of entropy along pseudo-time: "it computes point-wise label entropies that can highlight novel transition cell types de novo along developmental pseudo-time manifolds."
