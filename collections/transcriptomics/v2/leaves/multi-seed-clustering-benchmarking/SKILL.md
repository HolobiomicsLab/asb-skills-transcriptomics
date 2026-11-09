---
name: multi-seed-clustering-benchmarking
description: Use when a new scRNA-seq clustering method must be validated against
  baselines across multiple real datasets, and clustering quality must be measured
  as NMI/ARI averaged over repeated random-seed runs.
when_to_use_negative:
- The goal is to tune a single clustering run on one dataset rather than to compare
  methods fairly across datasets — seeding and benchmarking overhead is unnecessary
  there
- The data are not scRNA-seq count matrices from single-cell transcriptome profiles
  (the method assumes zero-inflated scRNA-seq data with highly variable gene selection)
- Only one clustering metric is required per single run — this skill is specifically
  about multi-seed, multi-method, multi-dataset NMI/ARI benchmarking
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scBGEDA
  role: 'Proposed clustering method under benchmark: dual denoising autoencoder (ZINB
    + denoising reconstruction loss) plus bipartite graph ensemble clustering'
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: First module of scBGEDA; selects highly variable genes (e.g. --highly_genes
    2000) before benchmark runs
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Second module of scBGEDA; trains the single-cell dual denoising autoencoder
    on each dataset across the 10 random seeds
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: Main function of the third module of scBGEDA; runs bipartite graph ensemble
    clustering in MATLAB with the same seed vector
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: Computes NMI values after clustering for every method–dataset–seed combination
- name: rand_index.m
  role: Computes ARI values after clustering for every method–dataset–seed combination
- name: MATLAB
  role: Runtime for the third-module consensus clustering and metric scripts (BC.m,
    BGEC.m, computeNMI.m, rand_index.m)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:06897c84a72e6c001d193b404c21a8c9f55488034709dd98c0732b68bd16e94c
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# multi-seed-clustering-benchmarking

## Summary

Benchmark a single-cell clustering method against state-of-the-art competitors by running every method on the same scRNA-seq datasets under 10 fixed random seeds (1111–10000) and comparing averaged NMI and ARI, as done for scBGEDA versus SHARP, CIDR, SOUP, Seurat, SC3, Scanpy, PCA, DCA, scDeepCluster, scziDesk and DESC.

## When to use

Use when a new scRNA-seq clustering method must be validated against state-of-the-art baselines across multiple real datasets, and clustering quality must be measured as NMI/ARI averaged over repeated random-seed runs.

## When NOT to use

- The goal is to tune a single clustering run on one dataset rather than to compare methods fairly across datasets — seeding and benchmarking overhead is unnecessary there
- The data are not scRNA-seq count matrices from single-cell transcriptome profiles (the method assumes zero-inflated scRNA-seq data with highly variable gene selection)
- Only one clustering metric is required per single run — this skill is specifically about multi-seed, multi-method, multi-dataset NMI/ARI benchmarking

## Inputs

- Real scRNA-seq datasets from the figshare deposit https://figshare.com/articles/software/scBGEDA/19657911
- Preprocessed expression matrices with 2000 highly variable genes selected by preprocess.py
- Fixed random-seed vector: 1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000
- Eleven comparison method outputs (SHARP, CIDR, SOUP, Seurat, SC3, Scanpy, PCA, DCA, scDeepCluster, scziDesk, DESC) on the same datasets and seeds

## Outputs

- NMI values per method–dataset–seed combination (computed by computeNMI.m)
- ARI values per method–dataset–seed combination (computed by rand_index.m)
- Averaged NMI/ARI table (one row per method per dataset) suitable for clustering_benchmark_results.csv
- Final clustering performance report; the BC.m output reports the median values of ARI and NMI

## How to apply

Download the real scRNA-seq datasets and source codes from the figshare deposit (https://figshare.com/articles/software/scBGEDA/19657911) and clone https://github.com/wangyh082/scBGEDA. For each dataset, run preprocess.py (first module, e.g. --highly_genes 2000 to select 2000 highly variable genes), then SDDA.py (second module, the single-cell dual denoising autoencoder) with the fixed seed string "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" to obtain latent embeddings. Run BC.m in MATLAB with the same seed vector (e.g. matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]", where 100 is the ensemble size m and 2–40 bound the number of clusters in the basic clusterings) to produce bipartite graph ensemble consensus clusterings. Run each comparison method (SHARP, CIDR, SOUP, Seurat, SC3, Scanpy, PCA, DCA, scDeepCluster, scziDesk, DESC) on the same datasets under the same 10 seeds, then compute NMI with computeNMI.m and ARI with rand_index.m for every method–dataset–seed combination and average over seeds per method per dataset.

## Related tools

- **scBGEDA** (Proposed clustering method under benchmark: dual denoising autoencoder (ZINB + denoising reconstruction loss) plus bipartite graph ensemble clustering) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (First module of scBGEDA; selects highly variable genes (e.g. --highly_genes 2000) before benchmark runs) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Second module of scBGEDA; trains the single-cell dual denoising autoencoder on each dataset across the 10 random seeds) — https://github.com/wangyh082/scBGEDA
- **BC.m** (Main function of the third module of scBGEDA; runs bipartite graph ensemble clustering in MATLAB with the same seed vector) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (Computes NMI values after clustering for every method–dataset–seed combination)
- **rand_index.m** (Computes ARI values after clustering for every method–dataset–seed combination)
- **MATLAB** (Runtime for the third-module consensus clustering and metric scripts (BC.m, BGEC.m, computeNMI.m, rand_index.m))

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000"; matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- The results table contains one row per method per dataset with non-missing NMI and ARI fields, averaged over the 10 seeds
- All runs used the identical seed vector (1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000) for scBGEDA and every comparison method, ensuring fairness
- scBGEDA rows exhibit NMI/ARI values at least as high as the 11 comparison methods, consistent with the paper's reported superiority
- The BC.m output reports the median values of ARI and NMI, matching the expected output explanation in the README
- Preprocessing applied 2000 highly variable genes (--highly_genes 2000) per dataset before the autoencoder was trained

## Limitations

- The pipeline requires both Python (tensorflow 2.6.0, keras 2.6.0, scanpy 1.7.2, etc.) and MATLAB, since the third module (BC.m/BGEC.m) and metric scripts are MATLAB functions
- Random seeds are used 'to select the cells randomly to ensure the fairness'; omitting the same seeds across methods breaks comparability
- The exact names of the benchmarked scRNA-seq datasets and compared methods are not enumerated in the article/README body; they must be obtained from the figshare deposit
- Runs are computationally heavy: 10 seeds × 12 methods × 20 datasets, with GPU option and pretrain_epoch (e.g. 1000) settings per autoencoder run

## Evidence

- [abstract] seeds: "The experimental results indicated that scBGEDA outperforms other state-of-the-art methods on these datasets, and also demonstrated its scalability to large-scale scRNA-seq datasets."
- [supplementary] figshare availability: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
- [supplementary] SDDA command with seeds: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [supplementary] MATLAB BC command: "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [methods] fifteen datasets benchmarks: "Multiple comparison experiments were conducted on fifteen scRNA-seq datasets from different sequencing platforms using a variety of clustering metrics"
- [readme] seed rationale (readme): "The random seeds which are used to select the cells randomly to ensure the fairness"
- [readme] metric scripts (readme): "rand_index.m | Computing ARI values after clustering"
- [readme] output explanation (readme): "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
