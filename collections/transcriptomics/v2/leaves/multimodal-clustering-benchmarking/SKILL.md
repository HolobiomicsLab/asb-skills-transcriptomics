---
name: multimodal-clustering-benchmarking
description: Use when comparing multimodal single-cell clustering methods under controlled
  simulation conditions — e.g. varying clustering signal or dropout rates — where
  simulated ground-truth cell-type labels enable AMI/NMI/ARI evaluation.
when_to_use_negative:
- The input is real CITE-seq data with curated labels where the paper's real-data
  workflow (e.g. GSE128639 BMNC, GSE100866 CBMN) is used instead of the SymSim simulation
  pipeline
- The goal is marker detection, GSEA, or cell-type annotation rather than comparing
  clustering performance across methods
- The data is single-modality scRNA-seq only, since the competing methods here are
  designed for paired mRNA/ADT multimodal input
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SymSim
  role: R package used to simulate paired single-cell multi-omics count data with
    varying clustering signal, dropout, and batch effects
  repo: https://github.com/YosefLab/SymSim
- name: scMDC
  role: 'Method under test: multimodal deep clustering run with pretrain_epochs=400,
    gamma=0.1, tol=0.001 and supplied K'
  repo: https://github.com/xianglin226/scMDC
- name: Seurat V4
  role: Competing method using the weighted nearest-neighbor (WNN) procedure for multi-omics
    clustering
  repo: https://github.com/satijalab/seurat
- name: Tscan
  role: 'Competing method: PCA plus Gaussian mixture model clustering on scRNA-seq'
  repo: https://github.com/zji90/TSCAN
- name: SC3
  role: 'Competing method: spectral clustering over Euclidean, Pearson, and Spearman
    distance matrices'
  repo: https://github.com/hemberg-lab/SC3
- name: BREM-SC
  role: 'Competing method: hierarchical Bayesian mixture model jointly modeling scRNA-seq
    and ADT with multinomial models'
  repo: https://github.com/tarot0410/BREMSC
- name: CiteFuse
  role: 'Competing method: merges cell-to-cell similarity matrices of ADT and mRNA
    by similarity network fusion'
  repo: https://github.com/SydneyBioX/CiteFuse
- name: Specter
  role: 'Competing method: landmark-based spectral clustering with linear-time scalability,
    supports multimodal RNA+ADT input'
  repo: https://github.com/canzarlab/Specter
- name: scikit-learn
  role: Provides AMI, NMI, and ARI metrics for scoring cluster assignments against
    simulated ground-truth labels
- name: Scanpy
  role: 'Preprocesses simulated mRNA and ADT data separately: zero-count filtering,
    size-factor normalization, log-transform, scaling'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:6f27922ab8a68811155a5b1bf7b58d158f4a456947cdc8a20dd82007c2503ff5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - BREM-SC
  - CiteFuse
  - SC3
  - Scanpy
  - Seurat
  - Seurat V4
  - Specter
  - SymSim
  - Tscan
  - scDCC
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# multimodal-clustering-benchmarking

## Summary

Benchmark paired mRNA/ADT (CITE-seq) clustering methods on SymSim-simulated datasets spanning clustering-signal, dropout-rate, and batch-effect regimes, scoring each method against simulated ground-truth labels with AMI, NMI, and ARI.

## When to use

Use when comparing multimodal single-cell clustering methods under controlled simulation conditions — e.g. varying clustering signal or dropout rates — where simulated ground-truth cell-type labels enable AMI/NMI/ARI evaluation.

## When NOT to use

- The input is real CITE-seq data with curated labels where the paper's real-data workflow (e.g. GSE128639 BMNC, GSE100866 CBMN) is used instead of the SymSim simulation pipeline
- The goal is marker detection, GSEA, or cell-type annotation rather than comparing clustering performance across methods
- The data is single-modality scRNA-seq only, since the competing methods here are designed for paired mRNA/ADT multimodal input

## Inputs

- SymSim-simulated paired mRNA/ADT count datasets in h5 format (X1=RNA matrix, X2=ADT matrix, Y=ground-truth cell-type labels)
- 10 replicate datasets per experimental regime (clustering signal, dropout rate, batch effect)
- Known number of clusters K supplied by the user
- Raw CITE-seq count matrices if benchmarking extends to real data

## Outputs

- simulation_clustering_metrics.csv with mean AMI/NMI/ARI per method per experimental setting across 10 replicates
- Per-replicate clustering assignments from each method
- Values formatted into the Fig. 4 simulation panels and Supplementary Table 3 layout

## How to apply

Simulate 10 paired mRNA/ADT count datasets per experimental regime (clustering signal, dropout, batch effect) with SymSim, exporting each in h5 format with X1=RNA, X2=ADT, Y=ground-truth labels. Preprocess each dataset per the scMDC pipeline: filter zero-count genes and ADTs, size-factor normalize, log-transform, and scale mRNA and ADT separately with Scanpy. Run scMDC with simulation-study parameters (pretrain_epochs=400, gamma=0.1, tol=0.001, sigma noise on RNA/ADT) supplying the known number of clusters K, then run competitors (Tscan, Seurat V4 WNN, SC3, scDCC, BREM-SC, CiteFuse, Specter) with recommended settings. Compute AMI, NMI, and ARI with scikit-learn metrics against the simulated labels for every replicate, aggregate mean scores per method per setting across the 10 replicates, and write them into the Fig. 4 / Supplementary Table 3 layout as simulation_clustering_metrics.csv.

## Related tools

- **SymSim** (R package used to simulate paired single-cell multi-omics count data with varying clustering signal, dropout, and batch effects) — https://github.com/YosefLab/SymSim
- **scMDC** (Method under test: multimodal deep clustering run with pretrain_epochs=400, gamma=0.1, tol=0.001 and supplied K) — https://github.com/xianglin226/scMDC
- **Seurat V4** (Competing method using the weighted nearest-neighbor (WNN) procedure for multi-omics clustering) — https://github.com/satijalab/seurat
- **Tscan** (Competing method: PCA plus Gaussian mixture model clustering on scRNA-seq) — https://github.com/zji90/TSCAN
- **SC3** (Competing method: spectral clustering over Euclidean, Pearson, and Spearman distance matrices) — https://github.com/hemberg-lab/SC3
- **BREM-SC** (Competing method: hierarchical Bayesian mixture model jointly modeling scRNA-seq and ADT with multinomial models) — https://github.com/tarot0410/BREMSC
- **CiteFuse** (Competing method: merges cell-to-cell similarity matrices of ADT and mRNA by similarity network fusion) — https://github.com/SydneyBioX/CiteFuse
- **Specter** (Competing method: landmark-based spectral clustering with linear-time scalability, supports multimodal RNA+ADT input) — https://github.com/canzarlab/Specter
- **scikit-learn** (Provides AMI, NMI, and ARI metrics for scoring cluster assignments against simulated ground-truth labels)
- **Scanpy** (Preprocesses simulated mRNA and ADT data separately: zero-count filtering, size-factor normalization, log-transform, scaling)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score, normalized_mutual_info_score, adjusted_rand_score; ami = adjusted_mutual_info_score(Y_true, labels_pred)
```

## Evaluation signals

- scMDC attains the highest mean AMI/NMI/ARI over all competing methods across the signal, dropout, and batch-effect experiments, with TotalVI as next-best
- Each method is evaluated on exactly 10 replicate datasets per experimental regime
- All three metrics (AMI, NMI, ARI) are computed with scikit-learn against the simulated ground-truth labels Y from the h5 file
- The output table reproduces the layout of Fig. 4 simulation panels and Supplementary Table 3
- Each method's cluster assignment count matches the supplied number of clusters K

## Limitations

- The number of clusters K must be specified by the user; no automatic estimation is provided
- All scMDC experiments require GPU hardware (Nvidia Tesla P100, 16 G), which may limit reproducibility on CPU-only systems
- Benchmark conclusions depend on SymSim's simulation assumptions for count generation, dropout, and batch effects

## Evidence

- [discussion] Three experiments, 10 replicates: "We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment."
- [discussion] Separate modality normalization: "mRNA and ADT data are normalized separately but using the same method."
- [discussion] K must be user-specified: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] GPU hardware requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [readme] Specter multimodal capability: [evidence span withheld — anchor status: external_doc]
