---
name: adjusted-rand-index-evaluation
description: Use when a scRNA-seq dataset has ground-truth cell-type annotations and
  you need a scalar score comparing predicted cluster labels against them, or when
  benchmarking clustering methods (e.g., scAce vs DESC vs scDeepCluster) via mean
  ARI.
when_to_use_negative:
- The dataset lacks ground-truth cell-type annotations, so no external agreement score
  can be computed
- You need to decide the number of clusters itself (ARI only evaluates labels after
  clustering; scAce's adaptive cluster merging handles cluster-number selection)
- Evaluating cluster trajectories without final labels — ARI requires a complete final
  labeling per method
edam_operation: http://edamontology.org/operation_3643
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scAce
  role: Produces predicted cluster labels (adata.obs['scace_cluster']) and cell embeddings
    (adata.obsm['scace_emb']) that are scored against ground truth with ARI
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Loads datasets into AnnData and preprocesses (filter_genes min_cells=3, filter_cells
    min_genes=200, normalize_per_cell, log1p, scale) before clustering and ARI evaluation
- name: Leiden
  role: Provides de novo initial cluster labels inside scAce that feed into the adaptive
    cluster merging whose final labels are ARI-evaluated
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:2d25228ab1c9995030aa03deb0d466716ba0bc8182d6e4323d8f1779980b62e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3643
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# adjusted-rand-index-evaluation

## Summary

Compute the Adjusted Rand Index (ARI) between predicted single-cell cluster labels and provided cell-type annotations to quantify clustering accuracy, used in the scAce paper to rank methods (e.g., mean ARI over seven real scRNA-seq datasets, scAce 0.82 vs scDeepCluster 0.72 vs ADClust 0.61 in the ablation).

## When to use

Use when a scRNA-seq dataset has ground-truth cell-type annotations and you need a scalar score comparing predicted cluster labels against them, or when benchmarking clustering methods (e.g., scAce vs DESC vs scDeepCluster) via mean ARI.

## When NOT to use

- The dataset lacks ground-truth cell-type annotations, so no external agreement score can be computed
- You need to decide the number of clusters itself (ARI only evaluates labels after clustering; scAce's adaptive cluster merging handles cluster-number selection)
- Evaluating cluster trajectories without final labels — ARI requires a complete final labeling per method

## Inputs

- Predicted cluster labels from a clustering method (e.g., adata.obs['scace_cluster'] from scAce output)
- Ground-truth cell-type annotations for each cell (e.g., annotated scRNA-seq datasets from Supplementary Table S4)
- Per-dataset ARI values for averaging across benchmark datasets

## Outputs

- Per-dataset ARI score (float) for each method-label pairing
- Mean ARI over the seven datasets per method (mean-ARI ranking table)
- Comparison CSV (e.g., fig_s9a_ari_comparison.csv or cluster_trajectory_ari.csv with per-iteration cluster counts and final ARI values)

## How to apply

After running each clustering method (e.g., scAce's run_scace, DESC, graph-sc, scDeepCluster, ADClust, SCCAF), extract the predicted labels — for scAce these are in adata.obs['scace_cluster'] — and compare them against the dataset's provided cell-type annotations using the adjusted Rand index, computed per dataset. Average the per-dataset ARI values across the benchmark datasets (e.g., the seven annotated datasets in Supplementary Table S4: human pancreas GSE84133, mouse ES GSE65525, human PBMC, human kidney, mouse hypothalamus, mouse kidney, turtle brain) to obtain a mean-ARI ranking. ARI is chance-corrected, so identical-but-random labelings score near 0 while exact agreement with a different labeling scheme scores 1; this makes it suitable for comparing methods that report different cluster counts (e.g., scAce merging 25→8 clusters on Mouse kidney with final ARI 0.93).

## Related tools

- **scAce** (Produces predicted cluster labels (adata.obs['scace_cluster']) and cell embeddings (adata.obsm['scace_emb']) that are scored against ground truth with ARI) — https://github.com/sldyns/scAce
- **scanpy** (Loads datasets into AnnData and preprocesses (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) before clustering and ARI evaluation)
- **Leiden** (Provides de novo initial cluster labels inside scAce that feed into the adaptive cluster merging whose final labels are ARI-evaluated)

## Examples

```
import scanpy as sc; from scace import run_scace; from sklearn.metrics import adjusted_rand_score; adata = run_scace(sc.pp.scale(sc.pp.log1p(sc.pp.normalize_per_cell(sc.pp.filter_cells(sc.pp.filter_genes(sc.AnnData(data), min_cells=3), min_genes=200))))); ari = adjusted_rand_score(adata.obs['scace_cluster'], adata.obs['cell_type'])
```

## Evaluation signals

- The ranking table contains a mean-ARI row for each of the ten benchmarked methods across all seven datasets, with scAce achieving the highest mean ARI
- Reproduced ablation averages match Fig. S9A: scAce average ARI = 0.82, scDeepCluster = 0.72, ADClust = 0.61
- On Mouse kidney (Adam et al. 2017), scAce's final labels reach ARI = 0.93 with the trajectory 25 → 17 → 10 → 9 → 8 clusters
- ARI is computed against the provided cell-type annotations for every dataset, not against a method's own clusters (sanity check: comparing labels to themselves yields ARI = 1.0)
- Output CSVs (e.g., cluster_trajectory_ari.csv) contain one ARI row per method per dataset with no missing values

## Limitations

- ARI depends on the quality and granularity of the ground-truth annotations; methods that over-merge (e.g., ADClust merging to 5 mixed clusters on Mouse kidney) or stop early (e.g., SCCAF stopping at 17 clusters) are penalized through agreement with reference labels
- Mean ARI over datasets can mask per-dataset variability; per-dataset ARI values should be retained alongside averages
- Preprocessing choices (min_cells=3 gene filter, min_genes=200 cell filter, normalize_per_cell, log1p, scale) affect downstream ARI, so all compared methods must receive the same preprocessed input

## Evidence

- [results] scAce achieved the best mean ARI clustering accuracy, followed in ranking order by DESC, graph-sc, scDeepCluster, and ADClust: [evidence span withheld — anchor status: not_found]
- [supplementary] Compute the Adjusted Rand Index of each method's predicted labels against the provided cell-type annotations for every dataset.: [evidence span withheld — anchor status: not_found]
- [readme] The output adata contains cluster labels in adata.obs['scace_cluster'] and the cell embeddings in adata.obsm['scace_emb'].: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
- [results] scDeepCluster (average ARI = 0.72) and ADClust (average ARI = 0.61) trail scAce (average ARI = 0.82): [evidence span withheld — anchor status: not_found]
- [results] scAce initially obtained 25 clusters and gradually updated them to 17, 10, 9, and 8 clusters in the second to fifth iteration: [evidence span withheld — anchor status: not_found]
