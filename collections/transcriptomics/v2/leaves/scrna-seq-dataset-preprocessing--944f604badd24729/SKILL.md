---
name: scrna-seq-dataset-preprocessing
description: Use when raw scRNA-seq count data must be prepared for deep clustering
  with scBGEDA, requiring gene filtering and selection of highly variable genes before
  running SDDA.py and BC.m.
when_to_use_negative:
- Input is already preprocessed, denoised, or a low-dimensional latent embedding —
  re-running preprocess.py would not add information and could distort downstream
  ZINB/denoising reconstruction.
- The analysis goal is not scBGEDA-based clustering (e.g. a non-autoencoder pipeline
  with its own preprocessing), since this module is specifically the first module
  of scBGEDA.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3577
tools:
- name: scBGEDA
  role: Parent pipeline whose first (preprocessing) module this skill covers; real
    datasets and source codes are on figshare
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: Function of the first module of scBGEDA; performs filtering and highly-variable-gene
    selection before the autoencoder
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Second module of scBGEDA; consumes the preprocessed data to learn latent embeddings
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:67518957c40b46088640f52ddd0c7a24a615a4b6183f1ba740c7a5ff4eb44acc
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
  - scBGEDA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# scrna-seq-dataset-preprocessing

## Summary

Preprocessing module of scBGEDA that filters scRNA-seq count matrices and selects highly variable genes (e.g. --highly_genes 2000) to prepare denoised inputs for the dual denoising autoencoder and bipartite graph ensemble clustering.

## When to use

Use when raw scRNA-seq count data must be prepared for deep clustering with scBGEDA, requiring gene filtering and selection of highly variable genes before running SDDA.py and BC.m.

## When NOT to use

- Input is already preprocessed, denoised, or a low-dimensional latent embedding — re-running preprocess.py would not add information and could distort downstream ZINB/denoising reconstruction.
- The analysis goal is not scBGEDA-based clustering (e.g. a non-autoencoder pipeline with its own preprocessing), since this module is specifically the first module of scBGEDA.

## Inputs

- Raw scRNA-seq count matrix for a dataset (e.g. dataset 'Adam' from the figshare deposit)
- Parameter --highly_genes (number of highly variable genes, e.g. 2000)
- Random seed vector "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000"

## Outputs

- Preprocessed scRNA-seq expression matrix restricted to the selected highly variable genes, ready for SDDA.py
- Prepared inputs for downstream NMI (computeNMI.m) and ARI (rand_index.m) evaluation after clustering

## How to apply

Run preprocess.py, the first module of scBGEDA, on each real scRNA-seq dataset to execute the data preprocessing mechanism. Select 2000 highly variable genes by passing --highly_genes 2000, and use fixed random seeds (1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000) which 'are used to select the cells randomly to ensure the fairness'. The preprocessed data feeds directly into SDDA.py (single-cell dual denoising autoencoder) and then BC.m (bipartite graph ensemble clustering). Keep preprocessing settings identical across datasets and comparison methods (SHARP, CIDR, SOUP, Seurat, SC3, Scanpy, PCA, DCA, scDeepCluster, scziDesk, DESC) so that NMI/ARI results remain comparable.

## Related tools

- **scBGEDA** (Parent pipeline whose first (preprocessing) module this skill covers; real datasets and source codes are on figshare) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (Function of the first module of scBGEDA; performs filtering and highly-variable-gene selection before the autoencoder) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Second module of scBGEDA; consumes the preprocessed data to learn latent embeddings) — https://github.com/wangyh082/scBGEDA

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5 --gpu_option "2"
```

## Evaluation signals

- The preprocessed dataset retains the specified number of highly variable genes (e.g. 2000) as set by --highly_genes.
- Downstream SDDA.py runs complete successfully on the preprocessed output with the same seed vector (1111–10000).
- After BC.m clustering, computeNMI.m and rand_index.m produce non-null NMI/ARI values for every method-dataset-seed combination.
- Using identical preprocessing across datasets and comparison methods yields benchmark results where scBGEDA rows show NMI/ARI at least as high as the 11 comparison methods, consistent with the paper's findings.
- All cells are selected consistently under the fixed random seeds, matching the fairness rationale stated in the README.

## Limitations

- Preprocessing hyperparameters (highly_genes, seeds) must match those used in the published benchmark; deviating defaults may change clustering quality relative to the reported NMI/ARI.
- Requires a conda environment with specific pinned dependencies (python 3.6+, tensorflow 2.6.0, keras 2.6.0, numpy 1.19.5, scanpy 1.7.2, etc.) — version mismatches can break the pipeline.
- Random seeds control cell selection, so results depend on the seed vector; comparisons must use the same seeds across all methods.

## Evidence

- [supplementary] preprocess.py is the first module of scBGEDA: "preprocess.py| Function of the first module of scBGEDA"
- [readme] Random seeds select cells randomly to ensure fairness: "--random_seed RANDOM_SEED The random seeds which are used to select the cells randomly to ensure the fairness"
- [readme] Example preprocessing/SDDA invocation with 2000 HVGs and seed vector: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [supplementary] Data availability for real datasets and source codes: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
- [readme] Workflow comprises three components including the data preprocessing mechanism: "the data preprocessing mechanism, the single-cell dual denoising autoencoder network, and the bipartite graph ensemble clustering method"
