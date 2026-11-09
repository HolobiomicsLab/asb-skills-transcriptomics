---
name: single-cell-latent-embedding-clustering
description: Use when the task is cell-type clustering on raw scRNA-seq count data
  (e.g. datasets like 'Adam') where deep-autoencoder latent representations must be
  learned and converted into consensus cell-type partitions evaluated by NMI/ARI.
when_to_use_negative:
- The input is not single-cell transcriptome data or the goal is not cell-type clustering
  — scBGEDA is designed for scRNA-seq cell-type identification, not bulk RNA-seq or
  general feature tables.
- A single fixed clustering (rather than a graph-based consensus partition from an
  ensemble of basic clusterings) is required — BGEC.m is specifically a consensus/ensemble
  function.
- The task requires downstream marker-gene or functional genomic analysis without
  clustering — those are separate capabilities requiring the trained model's gene-influence
  quantification.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scBGEDA
  role: The full deep-clustering pipeline (preprocessing + dual denoising autoencoder
    + bipartite graph ensemble clustering) that this skill applies
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: 'First module: data preprocessing of the scRNA-seq input'
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: 'Second module: trains the single-cell dual denoising autoencoder to produce
    the latent embedded space'
  repo: https://github.com/wangyh082/scBGEDA
- name: network.py
  role: Defines the single-cell dual denoising autoencoder network architecture
  repo: https://github.com/wangyh082/scBGEDA
- name: loss.py
  role: Implements the ZINB reconstruction and denoising reconstruction loss functions
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: 'Third module main function: drives bipartite graph ensemble clustering on
    the learned latent space'
  repo: https://github.com/wangyh082/scBGEDA
- name: BGEC.m
  role: Performs the bipartite graph ensemble clustering (graph-based consensus function)
  repo: https://github.com/wangyh082/scBGEDA
- name: rand_index.m
  role: Computes ARI values after clustering
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: Computes NMI values after clustering
  repo: https://github.com/wangyh082/scBGEDA
- name: MATLAB
  role: Runtime for the third-module ensemble clustering and metric scripts (BC.m,
    BGEC.m, rand_index.m, computeNMI.m)
- name: Python
  role: Runtime (python 3.6+, tensorflow 2.6.0, keras 2.6.0, scanpy 1.7.2) for preprocessing
    and the autoencoder modules
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:8e774fe44b22541c8706b43217b3eda9d81cd1c4411c76bf22e8bc992852dbb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - BGEC.m
  - MATLAB
  - Python
  - SDDA.py
  - Scanpy
  - computeNMI.m
  - loss.py
  - network.py
  - preprocess.py
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 27
---

# single-cell-latent-embedding-clustering

## Summary

Clusters scRNA-seq cell types by first learning a compressed latent embedding with a single-cell dual denoising autoencoder (ZINB + denoising reconstruction loss), then deriving a consensus partition from that latent space via bipartite graph ensemble clustering (scBGEDA).

## When to use

Use when the task is cell-type clustering on raw scRNA-seq count data (e.g. datasets like 'Adam') where deep-autoencoder latent representations must be learned and converted into consensus cell-type partitions evaluated by NMI/ARI.

## When NOT to use

- The input is not single-cell transcriptome data or the goal is not cell-type clustering — scBGEDA is designed for scRNA-seq cell-type identification, not bulk RNA-seq or general feature tables.
- A single fixed clustering (rather than a graph-based consensus partition from an ensemble of basic clusterings) is required — BGEC.m is specifically a consensus/ensemble function.
- The task requires downstream marker-gene or functional genomic analysis without clustering — those are separate capabilities requiring the trained model's gene-influence quantification.

## Inputs

- Raw scRNA-seq count data for a named dataset (e.g. 'Adam'), obtained from the figshare deposit https://figshare.com/articles/software/scBGEDA/19657911
- Number of highly variable genes (e.g. 2000)
- Random seeds used to select cells randomly to ensure fairness (e.g. '1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000')
- Ground-truth cell labels for NMI/ARI computation

## Outputs

- Latent low-dimensional feature representation of cells from the dual denoising autoencoder
- Consensus cell-type cluster assignments from bipartite graph ensemble clustering
- Clustering performance report with median ARI (rand_index.m) and NMI (computeNMI.m) values

## How to apply

Run scBGEDA's three modules in sequence: (1) preprocess.py applies the data preprocessing mechanism, selecting highly variable genes (e.g. --highly_genes 2000 for the 'Adam' dataset); (2) SDDA.py trains the single-cell dual denoising autoencoder network (network.py, loss.py, utils.py) that projects data into a compressed low-dimensional space (e.g. --dims '256, 32') by synergistic optimization of the ZINB reconstruction loss and denoising reconstruction loss, controlled by the alpha hyperparameter (e.g. 0.001) and noise_sd (e.g. 1.5), across random seeds '1111, 2222, ..., 10000' for fairness; (3) in MATLAB, BC.m invokes BGEC.m, the bipartite graph ensemble clustering that exploits relationships between cells and the latent embedded space via a graph-based consensus function, generating basic clusterings with k between bcsLowK=2 and bcsUpK=40 over an ensemble size m=100. Evaluate the resulting partitions with rand_index.m (ARI) and computeNMI.m (NMI), reporting median values across the seed ensemble.

## Related tools

- **scBGEDA** (The full deep-clustering pipeline (preprocessing + dual denoising autoencoder + bipartite graph ensemble clustering) that this skill applies) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (First module: data preprocessing of the scRNA-seq input) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Second module: trains the single-cell dual denoising autoencoder to produce the latent embedded space) — https://github.com/wangyh082/scBGEDA
- **network.py** (Defines the single-cell dual denoising autoencoder network architecture) — https://github.com/wangyh082/scBGEDA
- **loss.py** (Implements the ZINB reconstruction and denoising reconstruction loss functions) — https://github.com/wangyh082/scBGEDA
- **BC.m** (Third module main function: drives bipartite graph ensemble clustering on the learned latent space) — https://github.com/wangyh082/scBGEDA
- **BGEC.m** (Performs the bipartite graph ensemble clustering (graph-based consensus function)) — https://github.com/wangyh082/scBGEDA
- **rand_index.m** (Computes ARI values after clustering) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (Computes NMI values after clustering) — https://github.com/wangyh082/scBGEDA
- **MATLAB** (Runtime for the third-module ensemble clustering and metric scripts (BC.m, BGEC.m, rand_index.m, computeNMI.m))
- **Python** (Runtime (python 3.6+, tensorflow 2.6.0, keras 2.6.0, scanpy 1.7.2) for preprocessing and the autoencoder modules)

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000"; matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- The final MATLAB output reports clustering performance with median ARI (rand_index.m) and median NMI (computeNMI.m) values across the seed ensemble — both metrics must be produced.
- Reproduced NMI/ARI should match the values reported in the paper's comparison tables for the fifteen scRNA-seq datasets, with scBGEDA expected to outperform other state-of-the-art methods.
- The autoencoder training must use both loss components (ZINB reconstruction loss and denoising reconstruction loss) with alpha controlling their relative impact; omitting either decoder breaks the dual-denoising design.
- Basic clusterings for the ensemble must span k from bcsLowK (e.g. 2) to bcsUpK (e.g. 40) over the ensemble size m (e.g. 100), with the same random_seed list used in SDDA.py reused in BC.m.
- Run with multiple random seeds (e.g. 1111–10000) to ensure fairness, since seeds are used to select cells randomly.

## Limitations

- Requires two runtimes: a Python environment (python 3.6+, tensorflow 2.6.0, keras 2.6.0, numpy 1.19.5, jgraph 0.2.1, scipy 1.5.4, scanpy 1.7.2, pathos 0.2.8, tqdm 4.64.0) and MATLAB for the clustering module.
- The autoencoder may be insufficient for latent representation of some scRNA-seq data and obtaining consensus partitions from feature representations remains under-explored — the motivation for the ensemble consensus step.
- Performance varies by dataset; hyperparameters (dataname, highly_genes, random_seed, dims, alpha, learning_rate, batch_size, pretrain_epoch, noise_sd, gpu_option, m, bcsLowK, bcsUpK) may need tuning per dataset rather than relying on defaults.

## Evidence

- [abstract] a single-cell dual denoising autoencoder network is proposed to project the data into a compressed low-dimensional space: "a single-cell dual denoising  autoencoder network is proposed to project the data into a compressed low-dimensional space"
- [abstract] bipartite graph ensemble clustering exploits cell-latent relationships via graph-based consensus: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [supplementary] Python invocation for SDDA.py on dataset Adam: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [supplementary] MATLAB invocation for BC.m on dataset Adam: "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [supplementary] rand_index.m and computeNMI.m compute post-clustering metrics: "rand_index.m | Computing ARI values after clustering"
- [readme] README lists key functions including BGEC.m: "BGEC.m       | Bipartite Graph Ensemble Clustering"
- [readme] The random seed ensures fairness of cell selection: "--random_seed RANDOM_SEED The random seeds which are used to select the cells randomly to ensure the fairness"
- [readme] Output reports median ARI and NMI: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
