---
name: cross-dataset-clustering-benchmark-aggregation
description: Use when benchmarking a clustering model across multiple scRNA-seq datasets
  and random seeds (e.g. 20 datasets, seeds 1111–10000) and you must assemble per-dataset
  mean/median ARI comparisons into one table.
when_to_use_negative:
- You are evaluating a single dataset with a single run — per-seed aggregation and
  cross-dataset tables are unnecessary overhead
- No ground-truth cell-type labels exist for the datasets, so ARI (rand_index.m) and
  NMI (computeNMI.m) cannot be computed
- You need a novel clustering method itself rather than a benchmark/ablation comparison
  of existing configurations
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scBGEDA
  role: The clustering pipeline whose outputs (latent embeddings and consensus partitions)
    are benchmarked and aggregated across datasets
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Trains the Single-cell Dual Denoising Autoencoder per dataset, seed, and configuration
    with --highly_genes 2000 and the shared seed list
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: MATLAB main function running bipartite graph ensemble clustering on the latent
    embeddings for each seed, producing ARI/NMI performance reports
  repo: https://github.com/wangyh082/scBGEDA
- name: rand_index.m
  role: Computes ARI values after clustering, the metric aggregated into the benchmark
    table
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: Computes NMI values after clustering as a secondary benchmark metric
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: First module of scBGEDA; selects the top 2000 highly variable genes before
    training
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:4a0a4150e6edf562d5a7cd36c226f00a9d768b9a01564c39c98f1da1741e74b6
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
  - preprocess.py
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# cross-dataset-clustering-benchmark-aggregation

## Summary

Aggregates clustering performance (ARI/NMI) across many scRNA-seq datasets, seeds, and model configurations into a single benchmark table, following scBGEDA's protocol of running the dual denoising autoencoder plus bipartite graph ensemble clustering over multiple random seeds per dataset.

## When to use

Use when benchmarking a clustering model across multiple scRNA-seq datasets and random seeds (e.g. 20 datasets, seeds 1111–10000) and you must assemble per-dataset mean/median ARI comparisons into one table.

## When NOT to use

- You are evaluating a single dataset with a single run — per-seed aggregation and cross-dataset tables are unnecessary overhead
- No ground-truth cell-type labels exist for the datasets, so ARI (rand_index.m) and NMI (computeNMI.m) cannot be computed
- You need a novel clustering method itself rather than a benchmark/ablation comparison of existing configurations

## Inputs

- scRNA-seq count matrices from the figshare deposit (https://figshare.com/articles/software/scBGEDA/19657911)
- Ground-truth cell-type labels per dataset
- Latent embeddings produced by SDDA.py for each dataset, seed, and model configuration
- Predefined random seed list: 1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000

## Outputs

- Per-dataset, per-configuration, per-seed ARI and NMI values
- Aggregated benchmark table, e.g. ablation_ari_comparison.csv with mean ARI per dataset and configuration
- Median ARI and NMI values reported by the BC.m MATLAB output

## How to apply

Run the clustering pipeline end-to-end on every dataset: preprocess each scRNA-seq matrix with preprocess.py using the top 2000 highly variable genes, train the Single-cell Dual Denoising Autoencoder (SDDA.py with network.py and loss.py) under each configuration being compared, then run bipartite graph ensemble clustering (BC.m in MATLAB) on the latent embeddings. Sweep ten seeds (1111, 2222, ..., 10000) per dataset so that cell subsampling fairness is preserved, and score each partition against ground-truth cell labels with rand_index.m (ARI) and computeNMI.m (NMI). For ablation-style benchmarks (e.g. full Eq. 8 dual loss with the MSE follower-decoder term vs. ZINB-only), keep all hyperparameters --highly_genes 2000, --dims "256, 32", --alpha, --learning_rate, --batch_size, --pretrain_epoch, --noise_sd identical across configurations and change only the loss term. Aggregate per-dataset ARI/NMI by seed and report the mean (or the median across seeds, as the scBGEDA BC.m output does) so trends like 'the MSE term improves ARI on most of the 20 datasets' (Fig. 3B, Supplementary Table S20) can be verified.

## Related tools

- **scBGEDA** (The clustering pipeline whose outputs (latent embeddings and consensus partitions) are benchmarked and aggregated across datasets) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Trains the Single-cell Dual Denoising Autoencoder per dataset, seed, and configuration with --highly_genes 2000 and the shared seed list) — https://github.com/wangyh082/scBGEDA
- **BC.m** (MATLAB main function running bipartite graph ensemble clustering on the latent embeddings for each seed, producing ARI/NMI performance reports) — https://github.com/wangyh082/scBGEDA
- **rand_index.m** (Computes ARI values after clustering, the metric aggregated into the benchmark table) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (Computes NMI values after clustering as a secondary benchmark metric) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (First module of scBGEDA; selects the top 2000 highly variable genes before training) — https://github.com/wangyh082/scBGEDA

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- The aggregated table shows the full dual loss (ZINB + MSE) attaining higher ARI on most of the 20 datasets, matching the trend of Fig. 3B and Supplementary Table S20
- Every dataset × configuration × seed cell (10 seeds from 1111 to 10000) has a recorded ARI/NMI value with no missing entries
- BC.m's final output reports median ARI and NMI values consistent with the mean values computed in the aggregated table
- Identical hyperparameters (--highly_genes 2000, --dims "256, 32", --alpha, --batch_size, --pretrain_epoch, --noise_sd) are used across the compared configurations, differing only in the loss term

## Limitations

- The article reports results on fifteen to twenty scRNA-seq datasets from different sequencing platforms; datasets must come from the figshare deposit for reproducibility
- Random seeds are used to select cells randomly to ensure fairness — omitting the shared seed list breaks the comparability across configurations
- The README does not list the specific dataset names or the compared state-of-the-art methods; these must be obtained from the paper's supplementary tables

## Evidence

- [results] Multiple comparison experiments were conducted on fifteen scRNA-seq datasets from different sequencing platforms using a variety of clustering metrics: "Multiple comparison experiments were conducted on fifteen scRNA-seq datasets from different sequencing platforms using a variety of clustering metrics"
- [supplementary] seeds used across SDDA and BC: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [supplementary] rand_index.m computes ARI: "rand_index.m | Computing ARI values after clustering"
- [readme] The final output reports median ARI and NMI: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
- [readme] random seed selects cells randomly for fairness: "--random_seed RANDOM_SEED The random seeds which are used to select the cells randomly to ensure the fairness"
