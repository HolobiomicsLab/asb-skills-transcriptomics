---
name: large-scale-scrnaseq-clustering-benchmarking
description: Use when a clustering method for scRNA-seq count matrices must be benchmarked
  against competitors on datasets with ground-truth cell-type labels, or when robustness
  on downsampled large-scale data must be quantified with ARI/NMI and significance
  tests.
when_to_use_negative:
- The input is already a latent embedding or precomputed clustering — this skill benchmarks
  full clustering pipelines from raw count matrices, not downstream consensus steps
  in isolation.
- The dataset lacks ground-truth cell-type labels — ARI (rand_index.m) and NMI (computeNMI.m)
  computations require reference labels.
- The goal is a single-method clustering run without competitor comparison or robustness
  assessment — a benchmark setup with multiple seeds and competitor methods is unnecessary
  overhead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scBGEDA
  role: 'the clustering model being benchmarked; three modules: preprocess.py, SDDA.py
    (Python/TensorFlow), BC.m (MATLAB)'
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: 'first module: filtering, normalization, and highly-variable gene selection
    before benchmarking'
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: 'second module: single-cell Dual Denoising Autoencoder feature learning, run
    across ten seeds with --highly_genes 2000'
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: 'third module: bipartite graph ensemble clustering (m=100, k from 2 to 40)
    invoked via matlab -r'
  repo: https://github.com/wangyh082/scBGEDA
- name: rand_index.m
  role: computes ARI values after clustering for benchmark comparison
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: computes NMI values after clustering for benchmark comparison
  repo: https://github.com/wangyh082/scBGEDA
- name: SciPy
  role: Wilcoxon signed-rank test on ARI values across seed replicates to assess significance
    of improvement over competitors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:f1707c40a82cba7977b361b8883f01153b1947117c1081ee0e476f66f1584052
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - MATLAB
  - Python
  - SDDA.py
  - Scanpy
  - SciPy
  - computeNMI.m
  - jgraph
  - preprocess.py
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# large-scale-scrnaseq-clustering-benchmarking

## Summary

Benchmark a scRNA-seq deep-clustering pipeline (scBGEDA: dual denoising autoencoder + bipartite graph ensemble clustering) against competitor methods on labeled datasets, evaluating clustering quality with ARI/NMI medians across ten seeds (1111–10000) and Wilcoxon significance tests, plus downsampled robustness checks on large-scale data (e.g., PBMC_68k at 20/40/60/80%).

## When to use

Use when a clustering method for scRNA-seq count matrices must be benchmarked against competitors on datasets with ground-truth cell-type labels, or when robustness on downsampled large-scale data must be quantified with ARI/NMI and significance tests.

## When NOT to use

- The input is already a latent embedding or precomputed clustering — this skill benchmarks full clustering pipelines from raw count matrices, not downstream consensus steps in isolation.
- The dataset lacks ground-truth cell-type labels — ARI (rand_index.m) and NMI (computeNMI.m) computations require reference labels.
- The goal is a single-method clustering run without competitor comparison or robustness assessment — a benchmark setup with multiple seeds and competitor methods is unnecessary overhead.

## Inputs

- scRNA-seq raw count matrices with ground-truth cell-type labels (e.g., PBMC_68k, Tabula Muris, Adam) from the scBGEDA figshare deposit (https://figshare.com/articles/software/scBGEDA/19657911)
- Preprocessed expression matrices after filtering, normalization and highly variable gene selection (2000 highly variable genes by default)
- Random seed list (1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000)

## Outputs

- Per-run ARI and NMI values for scBGEDA and competitor methods (rand_index.m, computeNMI.m)
- Median ARI and NMI across the ten seeds per dataset (reported as the final output of the pipeline)
- Wilcoxon signed-rank test results (SciPy) for ARI improvement over each competitor
- Downsampling robustness table: median ARI at 20%, 40%, 60%, 80% cell retention levels

## How to apply

Download the count matrices with ground-truth cell-type labels from the scBGEDA figshare deposit and preprocess each dataset with the first module (preprocess.py): filtering, normalization, and highly variable gene selection (e.g., 2000 highly variable genes). Run the second module, the Single-cell Dual Denoising Autoencoder (SDDA.py, Python/TensorFlow), with explicit --dims '256, 32', --alpha 0.001, --learning_rate 0.0001, --batch_size 256, --pretrain_epoch 1000, --noise_sd 1.5, across the ten random seeds 1111–10000 to ensure fairness of random cell selection. Run the third module, Bipartite Graph Ensemble Clustering (BC.m / BGEC.m in MATLAB), with ensemble size m=100 and basic clustering k range 2–40 (bcsLowK=2, bcsUpK=40) using the same seeds, then compute ARI with rand_index.m and NMI with computeNMI.m per run. For the benchmark, run the nine competitor methods with their default settings on the identical preprocessed data, record median ARI/NMI across seeds, and apply a Wilcoxon signed-rank test (SciPy) to assess significance of ARI differences. For the robustness check, downsample the large-scale dataset (PBMC_68k) to 20%, 40%, 60%, and 80% of its cells, re-run the full pipeline at each level, and record median ARI across seeds; no significant difference across levels (P>0.05) indicates robustness.

## Related tools

- **scBGEDA** (the clustering model being benchmarked; three modules: preprocess.py, SDDA.py (Python/TensorFlow), BC.m (MATLAB)) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (first module: filtering, normalization, and highly-variable gene selection before benchmarking) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (second module: single-cell Dual Denoising Autoencoder feature learning, run across ten seeds with --highly_genes 2000) — https://github.com/wangyh082/scBGEDA
- **BC.m** (third module: bipartite graph ensemble clustering (m=100, k from 2 to 40) invoked via matlab -r) — https://github.com/wangyh082/scBGEDA
- **rand_index.m** (computes ARI values after clustering for benchmark comparison) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (computes NMI values after clustering for benchmark comparison) — https://github.com/wangyh082/scBGEDA
- **SciPy** (Wilcoxon signed-rank test on ARI values across seed replicates to assess significance of improvement over competitors)

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000"; matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- The final output reports median ARI and NMI per dataset; scBGEDA should attain the best median ARI at every downsampling level (20/40/60/80%) of PBMC_68k, matching the reported pattern in Figures 4B-D and Supplementary Tables S21-S22.
- Wilcoxon signed-rank tests on PBMC_68k should show significant ARI improvement over each competitor; the downsampling robustness check should yield no significant difference across data sizes (P>0.05).
- All ten seeds (1111–10000) must appear in both the SDDA.py --random_seed argument and the MATLAB BC.m seed vector — mismatched seeds invalidate the fairness of the benchmark.
- Each method must be run on the identical preprocessed data (same filtering, normalization, and 2000 highly variable genes); competitors run with their default settings.
- A run of the full pipeline on the example dataset 'Adam' should reproduce median ARI/NMI values consistent with the deposited results before scaling up to PBMC_68k / Tabula Muris.

## Limitations

- The pipeline requires both Python (TensorFlow 2.6.0, Keras 2.6.0, scanpy 1.7.2, jgraph 0.2.1) and MATLAB, since the ensemble clustering module is MATLAB-only.
- The number of clusters (bcsLowK=2 to bcsUpK=40) bounds the cluster-count search space; datasets with more than 40 expected cell types may need adjusted parameters.
- Missing-signal caveats from the article index: specific names of the fifteen datasets, the clustering metric set, and the names of the nine compared state-of-the-art methods are not listed in the results text and must be recovered from supplementary tables.
- Random seeds are used to select cells randomly to ensure fairness — results are medians over 10 seeds, not single deterministic runs.

## Evidence

- [supplementary] The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911).: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
- [supplementary] rand_index.m | Computing ARI values after clustering: "rand_index.m | Computing ARI values after clustering"
- [readme] The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively.: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
- [results] Multiple comparison experiments were conducted on fifteen scRNA-seq datasets from different sequencing platforms using a variety of clustering metrics: "Multiple comparison experiments were conducted on fifteen scRNA-seq datasets from different sequencing platforms using a variety of clustering metrics"
- [readme] matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]": "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [supplementary] python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000": "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
