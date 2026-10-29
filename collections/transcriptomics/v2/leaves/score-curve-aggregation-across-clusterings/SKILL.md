---
name: score-curve-aggregation-across-clusterings
description: Use when many candidate clusterings exist for a trajectory-inference
  benchmark and you must compare selection criteria (e.g. connectivity+VRC vs ASW
  vs VRC vs random) by how trajectory quality degrades over the top-n ranked candidates.
when_to_use_negative:
- The dataset has extremely rare and extremely prevalent cell types simultaneously,
  where k-medoids does not work properly and clusterings must be merged manually before
  ranking is meaningful
- The expected trajectory is disconnected or contains cycles, which Totem cannot generate
- The trajectory is multi-root (converging), since dynplot will not generate the correct
  pseudotime and scores based on pseudotime are unreliable
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: Totem
  role: Generates the CLARA k-medoids clusterings, constructs MSTs, computes cell
    connectivity, and ranks clusterings by Calinski-Harabasz score and connectivity
    for selection-curve evaluation
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree and linear curves through lineages from
    each selected top-n clustering to infer trajectories
- name: dyndimred
  role: Provides the landmark MDS (LMDS) 5-dimensional embedding used as Totem clustering
    input
- name: Seurat
  role: LogNormalize normalization of the datasets prior to PrepareTotem processing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:edb00dd496e1863ddfaaa0b28278524861756ce77aba5bb7e4c4ddf671bd5061
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# score-curve-aggregation-across-clusterings

## Summary

Benchmarks clustering-selection criteria for Totem trajectory inference by ranking 10,000 k-medoids (CLARA) clusterings per criterion, inferring MST trajectories from the top-n clusterings with Slingshot, and aggregating maximum/mean dynverse overall scores as score-versus-n curves to compare Totem cell connectivity+VRC against ASW, VRC-alone, and random ranking.

## When to use

Use when many candidate clusterings exist for a trajectory-inference benchmark and you must compare selection criteria (e.g. connectivity+VRC vs ASW vs VRC vs random) by how trajectory quality degrades over the top-n ranked candidates.

## When NOT to use

- The dataset has extremely rare and extremely prevalent cell types simultaneously, where k-medoids does not work properly and clusterings must be merged manually before ranking is meaningful
- The expected trajectory is disconnected or contains cycles, which Totem cannot generate
- The trajectory is multi-root (converging), since dynplot will not generate the correct pseudotime and scores based on pseudotime are unreliable

## Inputs

- Benchmark single-cell datasets as SingleCellExperiment objects (simulated binary_tree_1 and Zenodo doi=10.5281/zenodo.1443566 trajectory datasets)
- Seurat LogNormalize-normalized expression matrices with 5-dimensional LMDS embeddings
- Totem clustering results (10,000 CLARA runs over a range of k values, min.cluster.size=5)

## Outputs

- Top-100 ranked clustering lists per selection criterion (connectivity+VRC, ASW, VRC, random)
- Per-criterion score-versus-n curves of maximum and mean dynverse overall scores for n = 1..100
- Results table and comparison plot aggregating max/mean overall scores across all four criteria and datasets

## How to apply

Prepare each benchmark dataset (e.g. binary_tree_1 and the Zenodo 10.5281/zenodo.1443566 trajectory datasets) as a SingleCellExperiment object, normalize with Seurat LogNormalize, and embed with 5-dimensional LMDS from dyndimred via PrepareTotem. Run the k-medoids CLARA algorithm 10,000 times over a range of k values, filtering out clusterings containing clusters smaller than min.cluster.size (default 5 cells). Rank clusterings by four criteria — Totem cell connectivity combined with the Calinski-Harabasz (VRC) score, average silhouette width (ASW), VRC alone, and a random baseline — retaining the top 100 per criterion. For each criterion and each n from 1 to 100, construct minimum spanning trees with Slingshot from the top-n clusterings to infer trajectories, then evaluate each with dynverse metrics. Record the maximum and mean overall score across the n evaluated trajectories, aggregate these as functions of n for all four criteria, and summarize in a results table and comparison plot. Rationale: this reveals whether connectivity-guided selection not only finds the best trajectory but also degrades gracefully when multiple clusterings are evaluated, relative to ASW, VRC, and random.

## Related tools

- **Totem** (Generates the CLARA k-medoids clusterings, constructs MSTs, computes cell connectivity, and ranks clusterings by Calinski-Harabasz score and connectivity for selection-curve evaluation) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree and linear curves through lineages from each selected top-n clustering to infer trajectories)
- **dyndimred** (Provides the landmark MDS (LMDS) 5-dimensional embedding used as Totem clustering input)
- **Seurat** (LogNormalize normalization of the datasets prior to PrepareTotem processing)

## Examples

```
remotes::install_github("dynverse/dynfeature"); remotes::install_github("dynverse/dynplot"); remotes::install_github("elolab/Totem")
```

## Evaluation signals

- Complete max/mean overall-score-versus-n curves are produced for all four criteria (connectivity+VRC, ASW, VRC, random) on every benchmark dataset with no runtime errors
- The Totem cell-connectivity+VRC criterion attains the reported maximum and mean dynverse overall scores for the best trajectory, outperforming ASW and VRC on the maximum
- The random baseline shows the largest drop in average performance across n, while connectivity+VRC sacrifices less average performance — reproducing the paper's benchmark behavior
- All retained clusterings respect min.cluster.size (default 5 cells); clusterings with too-small clusters are excluded before ranking

## Limitations

- No correct pseudotime support for multi-root (converging) trajectories; dynplot will not generate the correct pseudotime and a manual work-around is required
- Totem cannot generate disconnected trajectories or trajectories with cycles
- k-medoids may not work well with high-dimensional PCA input (e.g. 30 principal components), producing 1-2 cell clusters
- k-medoids fails on extremely imbalanced datasets (rare plus prevalent cell types), requiring manual cluster merging

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] Ranking criteria combine connectivity and Calinski-Harabasz score.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] min.cluster.size filters out clusterings with too-small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] Clustering runs the k-medoids (CLARA) algorithm a large number of times (default 10k).: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [readme] README: Totem is a user-friendly tree-shaped trajectory inference tool.: "A user-friendly tool for inferring tree-shaped trajectories from single cell data."
