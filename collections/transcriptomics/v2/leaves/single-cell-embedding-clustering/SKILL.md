---
name: single-cell-embedding-clustering
description: Use when inferring a tree-shaped trajectory from single-cell data and
  a diverse set of candidate clusterings is needed to construct minimum spanning trees
  and estimate cell connectivity, as in the Totem workflow.
when_to_use_negative:
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids
  may fail and manual cluster merging is required instead
- The embedding is a high-dimensional PCA (e.g. 30 principal components), where k-medoids
  may produce degenerate 1-2 cell clusters
- The analysis goal does not require an ensemble of dissimilar clusterings (e.g. a
  single trusted prior clustering is already available to guide the search)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: Totem
  role: R package whose RunClustering/RunCLARA functions execute the repeated k-medoids
    clustering with k-range sampling, seeds, and min.cluster.size filtering
  repo: https://github.com/elolab/Totem
- name: dyndimred
  role: Produces the low-dimensional embedding (LMDS, 5 dimensions) on which CLARA
    clustering is run
- name: SingleCellExperiment
  role: Container format for the input single-cell data prepared with PrepareTotem
- name: R
  role: Language environment in which the clustering-generation stage is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:ca62ecbcacbd811ae8feb76d486b629667544a98304f30dd68e9762c83bf0999
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-embedding-clustering

## Summary

Generates a large ensemble of dissimilar clusterings by running the CLARA k-medoids algorithm many times (default 10,000 runs, k from 3 to 20) on a low-dimensional embedding of single-cell data, providing the clustering basis for downstream MST-based trajectory inference in Totem.

## When to use

Use when inferring a tree-shaped trajectory from single-cell data and a diverse set of candidate clusterings is needed to construct minimum spanning trees and estimate cell connectivity, as in the Totem workflow.

## When NOT to use

- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids may fail and manual cluster merging is required instead
- The embedding is a high-dimensional PCA (e.g. 30 principal components), where k-medoids may produce degenerate 1-2 cell clusters
- The analysis goal does not require an ensemble of dissimilar clusterings (e.g. a single trusted prior clustering is already available to guide the search)

## Inputs

- SingleCellExperiment object prepared for Totem (e.g. binary_tree_1 simulated dataset)
- Low-dimensional embedding of the cells, e.g. 5-dimensional landmark MDS (LMDS) from dyndimred
- Clustering parameters: runs (default 10000), k range (default 3-20), min.cluster.size (default 5), RNG seed

## Outputs

- R object of retained CLARA k-medoids clusterings (e.g. clustering_results.rds), each clustering containing only clusters of at least min.cluster.size cells

## How to apply

Start from a SingleCellExperiment object prepared with PrepareTotem and its low-dimensional embedding (e.g. 5-dimensional landmark MDS produced with dyndimred, which is faster than MDS for large datasets; note that a 2-dimensional embedding is needed only for visualization). Invoke Totem's RunClustering/RunCLARA functions with runs = 10000 (default) and k sampled over a range (default 3-20), enabling RNG seed handling for reproducibility. Apply the min.cluster.size = 5 filter to discard clusterings containing clusters with fewer than five cells, then return the retained clusterings as the output R object. Rationale: Totem relies on many dissimilar clusterings so that an MST can be built per clustering and cell connectivity can rank candidates (via the Calinski-Harabasz score and connectivity), so k-medoids instability across runs is exploited rather than avoided.

## Related tools

- **Totem** (R package whose RunClustering/RunCLARA functions execute the repeated k-medoids clustering with k-range sampling, seeds, and min.cluster.size filtering) — https://github.com/elolab/Totem
- **dyndimred** (Produces the low-dimensional embedding (LMDS, 5 dimensions) on which CLARA clustering is run)
- **SingleCellExperiment** (Container format for the input single-cell data prepared with PrepareTotem)
- **R** (Language environment in which the clustering-generation stage is executed)

## Examples

```
clustering_results <- RunClustering(ti, runs = 10000, k = 3:20, min.cluster.size = 5)
```

## Evaluation signals

- The number of retained clusterings equals 10,000 (or the chosen runs value) minus the clusterings filtered out by min.cluster.size
- Every retained clustering contains only clusters with at least five cells (the min.cluster.size criterion)
- The output R object serializes successfully as clustering_results.rds and contains k values within the sampled range (3-20 by default)
- Re-running with the same RNG seed reproduces the same set of clusterings

## Limitations

- k-medoids may not work well with PCA input when the number of principal components is high (e.g. 30), producing 1-2 cell clusters
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, k-medoids might not work properly, requiring manual cluster merging
- The default 10,000 runs and the min.cluster.size filter reduce the retained set; validation must account for filtered-out runs

## Evidence

- [intro] the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k).: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] The second parameter (min.cluster.size) determines the clusterings that are filtered out because they include too small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] Here, we use the landmark Multi-Dimensional Scaling (LMDS) from dyndimred package [4] to generate a low-dimensional embedding with 5 dimensions.: "Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [discussion] the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30): "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [intro] `Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering.: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
