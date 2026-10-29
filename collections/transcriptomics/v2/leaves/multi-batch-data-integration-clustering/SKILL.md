---
name: multi-batch-data-integration-clustering
description: Use when input is paired multi-omics single-cell data (H5 files with
  X1 RNA, X2 ATAC, Y labels) spanning multiple batches, and the goal is integrated
  clustering with batch-effect correction rather than per-batch analysis.
when_to_use_negative:
- Data is single-modal (RNA-only or ADT-only) — scMDC is designed for paired multimodal
  inputs; use single-modal deep clustering instead
- Only a joint embedding is needed (not clustering) — the paper notes methods 'focusing
  on joint embedding are not designed and optimized for clustering', and conversely
  scMDC targets clustering
- Only CPU hardware is available and runtime is critical — all experiments were conducted
  on an Nvidia Tesla P100 (16 G) GPU
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3295
tools:
- name: scMDC
  role: Conditional-autoencoder deep clustering model that corrects batch effects
    and clusters multi-batch RNA+ATAC data end-to-end
  repo: github.com/xianglin226/scMDC
- name: Seurat V4
  role: Competing multimodal baseline using the weighted nearest-neighbor (WNN) procedure
    on the same multi-batch data
  repo: github.com/satijalab/seurat
- name: Cobolt
  role: Competing joint-latent-representation multimodal baseline (v1.0.0) for benchmark
    comparison
  repo: github.com/epurdom/cobolt
- name: scMM
  role: Competing mixture-of-experts deep generative baseline (v1.0.0) for benchmark
    comparison
  repo: github.com/kodaim1115/scMM
- name: SKlearn
  role: K-means on PCA baseline and source of clustering metrics (NMI, AMI, ARI) used
    for evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:63fed60d65f441a5c82f813328ad3eb2d29df31eb10aca3ef39a9684601d92a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cobolt
  - SKlearn
  - Seurat V4
  - scMDC
  - scMM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-batch-data-integration-clustering

## Summary

Jointly clusters paired multimodal single-cell data across batches (e.g. multi-batch SMAGE-seq PBMC13K RNA+ATAC) using scMDC's conditional autoencoder, which corrects batch effects while learning a shared latent embedding, followed by deep K-means clustering.

## When to use

Use when input is paired multi-omics single-cell data (H5 files with X1 RNA, X2 ATAC, Y labels) spanning multiple batches, and the goal is integrated clustering with batch-effect correction rather than per-batch analysis.

## When NOT to use

- Data is single-modal (RNA-only or ADT-only) — scMDC is designed for paired multimodal inputs; use single-modal deep clustering instead
- Only a joint embedding is needed (not clustering) — the paper notes methods 'focusing on joint embedding are not designed and optimized for clustering', and conversely scMDC targets clustering
- Only CPU hardware is available and runtime is critical — all experiments were conducted on an Nvidia Tesla P100 (16 G) GPU

## Inputs

- H5-format multimodal single-cell files with X1 (RNA), X2 (ATAC), Y (cell-type labels), and Batch indicators (e.g. multi-batch SMAGE-seq PBMC13K)
- Raw scATAC-seq reads mapped to gene regions via Signac GeneActivity before preprocessing

## Outputs

- Cluster label predictions per cell (prediction file) from integrated multi-batch clustering
- Estimated number of clusters K when n_clusters=-1
- Benchmark table (e.g. smage_seq_benchmark_results.csv) with NMI/AMI/ARI values, ranks, and paired t-test p-values for scMDC vs. Cobolt, scMM, Seurat V4, and K-means+PCA

## How to apply

Assemble each dataset as an H5 file containing X1 (RNA counts), X2 (ATAC counts), Y (cell-type labels), and Batch indicators, per the scMDC 'dataset' folder readme. Train scMDC with the SMAGE-seq defaults (encodeLayer [256,128,64], sigma2=2.5, phi=0.005, pretrain_epochs=400, gamma=0.1, tol=0.001) on GPU, using zero-inflated negative binomial reconstruction loss; the conditional autoencoder structure removes batch effects so cells from different PBMC13K batches are clustered jointly. Let scMDC estimate K (n_clusters=-1, resolution=0.2, n_neighbors=30); K-means initializes K centroids on the pretrained latent space, then autoencoder, KL-loss, and deep K-means are optimized simultaneously. Benchmark against Cobolt, scMM, Seurat V4 (WNN), and K-means on PCA with AMI/NMI/ARI, and assess significance with one-sided paired t-tests.

## Related tools

- **scMDC** (Conditional-autoencoder deep clustering model that corrects batch effects and clusters multi-batch RNA+ATAC data end-to-end) — github.com/xianglin226/scMDC
- **Seurat V4** (Competing multimodal baseline using the weighted nearest-neighbor (WNN) procedure on the same multi-batch data) — github.com/satijalab/seurat
- **Cobolt** (Competing joint-latent-representation multimodal baseline (v1.0.0) for benchmark comparison) — github.com/epurdom/cobolt
- **scMM** (Competing mixture-of-experts deep generative baseline (v1.0.0) for benchmark comparison) — github.com/kodaim1115/scMM
- **SKlearn** (K-means on PCA baseline and source of clustering metrics (NMI, AMI, ARI) used for evaluation)

## Examples

```
python run_scMDC.py --data multi_batch_pbmc13k.h5 --encodeLayer 256 128 64 --sigma2 2.5 --phi 0.005 --pretrain_epochs 400 --gamma 0.1 --tol 0.001 --n_clusters -1 --resolution 0.2 --n_neighbors 30
```

## Evaluation signals

- Output CSV contains NMI, AMI, and ARI for all five methods on all four SMAGE-seq datasets plus per-dataset ranks and paired t-test p-values
- scMDC attains the top mean rank across metrics, with Cobolt typically second-best for AMI and NMI, matching the paper's reported outperformance
- Multi-batch PBMC13K integrated clustering places cells from different batches in shared clusters consistent with reference cell-type labels
- One-sided paired t-tests comparing scMDC vs. each competitor across datasets reach statistical significance
- Estimated K (n_clusters=-1, resolution=0.2, n_neighbors=30) matches the number of reference cell-type labels

## Limitations

- All experiments require GPU hardware (Nvidia Tesla P100 16 G), which may limit reproducibility on CPU-only systems
- The number of clusters K must generally be specified by the user; only via the n_clusters=-1 estimation mode is K inferred, adding sensitivity to resolution and n_neighbors settings
- ATAC data must first be converted to gene activity (Signac GeneActivity mapping reads to gene regions) before H5 assembly

## Evidence

- [intro] batch correction objective: "by employing a conditional autoencoder framework, scMDC can correct batch effects when analyzing multi-batch data."
- [methods] h5 input prep: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] pretraining schedule: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [intro] joint optimization: "The whole model, including the autoencoder, the KL-loss, and the deep K-means clustering, are optimized simultaneously."
- [readme] scMDC scope from README: "Extensive simulation and real-data experiments reveal that scMDC outperforms existing single-cell single-modal and multimodal clustering methods on"
