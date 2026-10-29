---
name: adjusted-rand-index-clustering-evaluation
description: Use when evaluating the quality of cell-type clustering results on scRNA-seq
  data against known cell labels, or when comparing clustering configurations (e.g.,
  ablations, seeds, methods) via ARI.
when_to_use_negative:
- Ground-truth cell labels are unavailable — ARI requires a reference partition and
  cannot be computed from clustering output alone
- The cell order of predicted and reference label vectors differs — labels must be
  aligned per cell before calling rand_index.m or the ARI will be meaningless
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: rand_index.m
  role: MATLAB function that computes ARI values after clustering
- name: computeNMI.m
  role: MATLAB function that computes NMI values as the complementary clustering metric
- name: BC.m
  role: Produces the bipartite graph ensemble clustering assignments whose ARI is
    then evaluated
  repo: https://github.com/wangyh082/scBGEDA
- name: scBGEDA
  role: The full clustering pipeline (preprocess.py, SDDA.py, BC.m) whose outputs
    are scored with ARI
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:dc2ba4dc36dce489986667134f179c2446350d0b23d667eefa4ce2390e958fe0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - MATLAB
  - Python
  - SDDA.py
  - computeNMI.m
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# adjusted-rand-index-clustering-evaluation

## Summary

Computes the Adjusted Rand Index (ARI) to quantify agreement between predicted cell clusters and ground-truth cell labels, as done in scBGEDA's benchmarking of single-cell clustering on scRNA-seq datasets.

## When to use

Use when evaluating the quality of cell-type clustering results on scRNA-seq data against known cell labels, or when comparing clustering configurations (e.g., ablations, seeds, methods) via ARI.

## When NOT to use

- Ground-truth cell labels are unavailable — ARI requires a reference partition and cannot be computed from clustering output alone
- The cell order of predicted and reference label vectors differs — labels must be aligned per cell before calling rand_index.m or the ARI will be meaningless

## Inputs

- Predicted cluster assignment vector (one label per cell) from the ensemble clustering stage (output of BC.m/BGEC.m)
- Ground-truth cell-type label vector for the same cells, in the same cell order
- Optional: multiple label vectors across random seeds (e.g., seeds 1111–10000) for median/mean ARI aggregation

## Outputs

- ARI value (scalar per clustering result / per seed)
- Aggregated ARI per dataset across seeds (e.g., ablation_ari_comparison.csv for ablation studies)
- Optionally NMI values from computeNMI.m as a paired metric

## How to apply

After the clustering stage — in scBGEDA this is the bipartite graph ensemble clustering (BC.m / BGEC.m) run on latent embeddings from the Single-cell Dual Denoising Autoencoder — collect the predicted cluster assignments and the dataset's ground-truth cell-type labels. Run rand_index.m under MATLAB to compute the ARI between the two partitions; scBGEDA repeats this across ten random seeds (1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000) so results should be summarized as median/mean ARI per dataset. The same procedure computes NMI via computeNMI.m as a complementary metric. ARI is chance-adjusted, so it is the appropriate score when comparing ablation variants such as the full dual loss (ZINB + MSE) versus ZINB-only training on the same datasets.

## Related tools

- **rand_index.m** (MATLAB function that computes ARI values after clustering)
- **computeNMI.m** (MATLAB function that computes NMI values as the complementary clustering metric)
- **BC.m** (Produces the bipartite graph ensemble clustering assignments whose ARI is then evaluated) — https://github.com/wangyh082/scBGEDA
- **scBGEDA** (The full clustering pipeline (preprocess.py, SDDA.py, BC.m) whose outputs are scored with ARI) — https://github.com/wangyh082/scBGEDA

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- ARI is 1.0 when the predicted clustering matches the reference partition exactly and near 0 when agreement is at chance level; values outside [0, 1] indicate an implementation or alignment error
- The predicted-label vector and ground-truth label vector must have identical length and cell ordering
- Across the ten scBGEDA seeds (1111–10000), per-dataset ARI should be reported as a median/mean; large per-seed variance (> several points of ARI) suggests unstable clustering
- In the MSE-loss ablation, the full dual loss (ZINB + MSE) should attain higher ARI than the ZINB-only variant on most of the 20 real scRNA-seq datasets, matching Fig. 3B and Supplementary Table S20
- ARI computed via computeNMI.m's companion pipeline (NMI) should show a broadly consistent method ranking, since scBGEDA reports both median ARI and median NMI

## Limitations

- The article's README reports that final output provides 'the median values of ARI and NMI', so single-seed ARI values are sensitive to initialization and should be aggregated over the ten documented seeds
- ARI requires curated ground-truth cell-type annotations, which may not exist or may be contested for newly generated scRNA-seq data
- Dataset and method names for the full benchmark are not enumerated in the main text (only 'fifteen scRNA-seq datasets from different sequencing platforms'); the 20 real datasets are distributed via the figshare deposit

## Evidence

- [supplementary] rand_index.m computes ARI after clustering: "rand_index.m | Computing ARI values after clustering"
- [supplementary] computeNMI.m computes NMI as a complementary metric: "computeNMI.m | Computing NMI values after clustering"
- [readme] Output reports median ARI and NMI: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
- [supplementary] Ten fixed seeds are used for fairness in clustering and evaluation: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [abstract] Bipartite graph ensemble clustering is the clustering stage evaluated by ARI: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
