---
name: clustering-selection-criteria-ranking
description: Use when many candidate clusterings of a trajectory-containing single-cell
  dataset must be ranked and compared by selection criteria (connectivity+VRC, ASW,
  VRC, random) to pick clusterings for MST-based trajectory inference.
when_to_use_negative:
- The dataset contains extremely rare and extremely prevalent cell types simultaneously,
  where k-medoids may not work properly and manual cluster merging is needed
- The trajectory of interest is disconnected, contains cycles, or requires correct
  pseudotime for a multi-root (converging) trajectory, which Totem/dynplot cannot
  handle
- Only a single pre-existing clustering is available and no ranking over many clusterings
  is intended — use direct Slingshot MST construction instead
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Totem
  role: Generates the large pool of CLARA k-medoids clusterings, computes cell connectivity
    and Calinski-Harabasz scores, and provides the ranking used for clustering selection
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree backbones and linear curves from each
    selected clustering for trajectory evaluation
- name: SingleCellExperiment
  role: Container format for the prepared input dataset passed to PrepareTotem
- name: Seurat
  role: Provides LogNormalize normalization of the dataset before clustering
- name: dyndimred
  role: Generates the 5-dimensional landmark MDS (LMDS) low-dimensional embedding
    used for clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:30cf6fe12dfc40438a36bfaef4e6deb18c4d0b6e35a717ef00c04a765357e94e
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# clustering-selection-criteria-ranking

## Summary

Ranks many k-medoids (CLARA) clusterings of single-cell data by alternative criteria — Totem cell connectivity combined with the Calinski-Harabasz (VRC) score, average silhouette width (ASW), VRC alone, and a random baseline — to select clusterings for trajectory construction and compare which criterion yields the best-performing inferred trajectories.

## When to use

Use when many candidate clusterings of a trajectory-containing single-cell dataset must be ranked and compared by selection criteria (connectivity+VRC, ASW, VRC, random) to pick clusterings for MST-based trajectory inference.

## When NOT to use

- The dataset contains extremely rare and extremely prevalent cell types simultaneously, where k-medoids may not work properly and manual cluster merging is needed
- The trajectory of interest is disconnected, contains cycles, or requires correct pseudotime for a multi-root (converging) trajectory, which Totem/dynplot cannot handle
- Only a single pre-existing clustering is available and no ranking over many clusterings is intended — use direct Slingshot MST construction instead

## Inputs

- SingleCellExperiment object of a trajectory-containing single-cell dataset (e.g. simulated binary_tree_1 or Zenodo 10.5281/zenodo.1443566 datasets)
- LogNormalize-normalized expression data and a 5-dimensional LMDS embedding
- Totem clustering results from 10,000 CLARA k-medoids runs over a range of k values

## Outputs

- Ranked clustering lists (top 100 per criterion: connectivity+VRC, ASW, VRC, random)
- MST-based trajectories inferred with Slingshot from top-n ranked clusterings for n = 1..100
- dynverse max and mean overall scores as a function of number of evaluated trajectories per criterion
- Results table and comparison plot of score-versus-n curves for the four criteria

## How to apply

Prepare the dataset (e.g. binary_tree_1 or a Zenodo trajectory dataset) as a SingleCellExperiment object with Seurat LogNormalize normalization and a 5-dimensional LMDS embedding from dyndimred, then run Totem's clustering step with the k-medoids CLARA algorithm (default 10,000 runs) over a range of k values, filtering out clusterings containing clusters smaller than min.cluster.size (default 5 cells). Rank the retained clusterings by each of four criteria — Totem's cell connectivity combined with the Calinski-Harabasz score, average silhouette width (ASW), VRC alone, and a random baseline — keeping the top 100 clusterings per criterion. For each criterion and each n from 1 to 100, construct minimum spanning trees with Slingshot from the top-n ranked clusterings and evaluate the resulting trajectories with dynverse metrics, recording the maximum and mean overall score. Rationale: Totem measures cell connectivity from the MST topologies of all generated clusterings, and the paper's benchmark shows connectivity-guided selection finds better best-case trajectories than ASW or VRC while sacrificing less average performance than random ranking. Summarize max/mean overall scores as a function of the number of evaluated trajectories in a results table and comparison plot across all four criteria and datasets.

## Related tools

- **Totem** (Generates the large pool of CLARA k-medoids clusterings, computes cell connectivity and Calinski-Harabasz scores, and provides the ranking used for clustering selection) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree backbones and linear curves from each selected clustering for trajectory evaluation)
- **SingleCellExperiment** (Container format for the prepared input dataset passed to PrepareTotem)
- **Seurat** (Provides LogNormalize normalization of the dataset before clustering)
- **dyndimred** (Generates the 5-dimensional landmark MDS (LMDS) low-dimensional embedding used for clustering)

## Examples

```
remotes::install_github("elolab/Totem"); sce <- PrepareTotem(SingleCellExperiment(assays=list(counts=counts))); RunTotem(sce, k.range, n.clusters = 10000, min.cluster.size = 5)
```

## Evaluation signals

- Score-versus-n curves (max and mean dynverse overall score for n = 1..100) are complete for all four criteria on every benchmark dataset with no runtime errors
- The connectivity+VRC criterion reproduces the paper's benchmark behavior, attaining the reported maximum and mean dynverse overall scores
- Clusterings with clusters smaller than min.cluster.size (default 5 cells) are excluded from all ranking lists
- Each criterion's ranked list contains exactly the top 100 clusterings, and the random baseline is reproducible as a ranking control

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root trajectories, requiring a manual work-around
- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1-2 cell clusters
- With extremely rare plus extremely prevalent cell types, k-medoids might not work properly and manual cluster merging is required

## Evidence

- [intro] Totem uses the Calinski-Harabasz score and the cell connectivity to rank the clustering results.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] Totem generates a large number of clustering results with CLARA and constructs an MST for each clustering.: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] The min.cluster.size parameter determines the clusterings that are filtered out because they include too small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] In trajectory inference methods like Slingshot that require a clustering to construct the Minimum Spanning Tree (MST) as the backbone of the trajectory.: "In trajectory inference methods like `Slingshot`[2] that require a clustering to construct the Minimum Spanning Tree (`MST`) as the backbone of the trajectory"
- [intro] The dataset was normalized using the LogNormalize method from the Seurat R package.: "The dataset was normalized using the LogNormalize method from the Seurat R package."
- [intro] We use the landmark Multi-Dimensional Scaling (LMDS) from dyndimred package to generate a low-dimensional embedding with 5 dimensions.: "Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [discussion] The k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30).: "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [discussion] Totem cannot generate disconnected trajectories or trajectories that have cycles.: "`Totem` cannot generate disconnected trajectories or trajectories that have cycles."
