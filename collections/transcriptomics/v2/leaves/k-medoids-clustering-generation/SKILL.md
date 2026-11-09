---
name: k-medoids-clustering-generation
description: Use when building tree-shaped trajectory inference from single-cell data
  and you need many alternative clusterings (rather than one) to explore topologies,
  e.g. linear, bifurcating, or multifurcating trajectories in a SingleCellExperiment
  object.
when_to_use_negative:
- The dataset contains extremely rare cell types together with extremely prevalent
  cell types — k-medoids may not work properly and manual cluster merging would be
  needed.
- The target trajectory is disconnected or contains cycles — Totem cannot generate
  such trajectories, so generating clusterings for this purpose is futile.
- Input embedding is a high-dimensional PCA representation (e.g. 30 principal components)
  where k-medoids produces degenerate 1–2 cell clusters.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Runs the CLARA k-medoids clustering step over a range of k values and applies
    the min.cluster.size filtering; installed via BiocManager or remotes from GitHub.
  repo: https://github.com/elolab/Totem
- name: dyndimred
  role: Provides the LMDS embedding (5 dimensions) that serves as the input space
    in which CLARA clustering is performed.
- name: SingleCellExperiment
  role: Container class for the expression data fed into PrepareTotem and the Totem
    clustering step.
- name: Slingshot
  role: Consumes the selected clusterings downstream to construct Minimum Spanning
    Trees as trajectory backbones.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:56ba8220c60ee142dd5ba7680685953ef62004c00cf91c2b7fc491a691c3b12c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - Seurat
  - SingleCellExperiment
  - Slingshot
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# k-medoids-clustering-generation

## Summary

Generate a large ensemble of k-medoids (CLARA) clusterings over a range of k values on a low-dimensional single-cell embedding, so that many candidate clusterings exist for topology estimation and cell-connectivity-based trajectory selection with Totem.

## When to use

Use when building tree-shaped trajectory inference from single-cell data and you need many alternative clusterings (rather than one) to explore topologies, e.g. linear, bifurcating, or multifurcating trajectories in a SingleCellExperiment object.

## When NOT to use

- The dataset contains extremely rare cell types together with extremely prevalent cell types — k-medoids may not work properly and manual cluster merging would be needed.
- The target trajectory is disconnected or contains cycles — Totem cannot generate such trajectories, so generating clusterings for this purpose is futile.
- Input embedding is a high-dimensional PCA representation (e.g. 30 principal components) where k-medoids produces degenerate 1–2 cell clusters.

## Inputs

- SingleCellExperiment object with single-cell expression data, prepared via PrepareTotem (e.g. Seurat LogNormalize normalization)
- 5-dimensional LMDS embedding from dyndimred
- Range of k values (number of clusters) to sweep

## Outputs

- Ensemble of ~10,000 CLARA k-medoids clustering results with associated per-clustering scores
- Filtered set of clusterings passing min.cluster.size = 5
- Ranked clustering list (by Calinski-Harabasz score, cell connectivity, or criterion combination) for downstream MST/Slingshot trajectory construction

## How to apply

First embed the cells in a low-dimensional space — the paper uses 5-dimensional LMDS from dyndimred, since k-medoids can behave poorly on higher-dimensional PCA input. Then run the k-medoids CLARA algorithm a large number of times (default 10,000 in Totem) over a range of k values, using the Totem clustering step on the prepared SingleCellExperiment object (prepared with PrepareTotem, e.g. with Seurat LogNormalize normalization). Filter out clusterings that contain clusters smaller than min.cluster.size (default 5 cells), because these are considered unreliable. Keep the resulting ranked set of clusterings for subsequent MST construction with Slingshot and ranking by Calinski-Harabasz score and/or cell connectivity; the rationale is that trajectory inference success depends on finding a good clustering, so testing many clusterings instead of tuning one is fast and requires no deep methodological expertise.

## Related tools

- **Totem** (Runs the CLARA k-medoids clustering step over a range of k values and applies the min.cluster.size filtering; installed via BiocManager or remotes from GitHub.) — https://github.com/elolab/Totem
- **dyndimred** (Provides the LMDS embedding (5 dimensions) that serves as the input space in which CLARA clustering is performed.)
- **SingleCellExperiment** (Container class for the expression data fed into PrepareTotem and the Totem clustering step.)
- **Slingshot** (Consumes the selected clusterings downstream to construct Minimum Spanning Trees as trajectory backbones.)

## Evaluation signals

- The clustering step ran the default 10,000 CLARA iterations across the range of k values without runtime errors.
- Clusterings containing any cluster smaller than min.cluster.size = 5 cells were filtered out before ranking.
- Ranking the retained clusterings by Calinski-Harabasz score and/or cell connectivity yields an ordered list usable to select clusterings for Slingshot MST construction.
- Selected clusterings produce biologically meaningful MST topologies with correct milestone connections (leaves/branch points consistent with cell connectivity).
- Re-running the step reproduces a comparable distribution of clustering scores, confirming the ensemble covers diverse k values.

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing clusters of 1–2 cells.
- On datasets with extremely rare cell types alongside extremely prevalent cell types, k-medoids might not work properly and manual cluster merging is required.
- Totem cannot generate disconnected trajectories or trajectories with cycles, so the clustering ensemble only supports tree-shaped topologies.
- Default parameters (e.g. min.cluster.size = 5) may need adjustment for datasets with different cell-type prevalence.

## Evidence

- [intro] `Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering.: "Totem generates a large number of clustering results with a k-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k).: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [discussion] the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30): "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [readme] Totem uses Slingshot [2] to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
