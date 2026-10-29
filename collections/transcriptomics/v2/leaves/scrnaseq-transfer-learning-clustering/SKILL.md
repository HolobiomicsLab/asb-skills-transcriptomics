---
name: scrnaseq-transfer-learning-clustering
description: Use when clustering a small scRNA-Seq target dataset (e.g. rare cell
  types, 300–600 cells) for which a large, well-annotated source dataset from a related
  tissue or context exists.
when_to_use_negative:
- Source and target share only a small set of transcripts — the article notes 'a loss
  of (probably) vital information is inevitable since only the set of genes present
  in both datasets can be used'
- No large, well-annotated source dataset exists for the tissue or disease context
  of interest
- Downstream analysis other than clustering is the goal — the article explicitly states
  other downstream methods besides SC3 are an open research direction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scRNA
  role: Python framework implementing data simulation, NMF source modeling, and transfer
    learning into the target dataset (scRNA-source.sh, scRNA-target.sh)
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Downstream clustering method applied to the modified target dataset Xtrg_new
    and to baselines (TargetCluster, ConcatenateCluster)
- name: Seurat
  role: Optional batch effect removal pre-processing applied before the three clustering
    methods (R script seurat_cca_hockley_usoskin.R)
  repo: https://github.com/nicococo/scRNA/blob/master/R/seurat_cca_hockley_usoskin.R
- name: MAGIC
  role: Optional imputation pre-processing to overcome zero-inflation before clustering
    (Matlab script magic_hockley_usoskin.m)
  repo: https://github.com/nicococo/scRNA/blob/master/matlab/magic_hockley_usoskin.m
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:98be68eaab7f79034c711ae2f8f6c8d3c7d359ddf8e526bb4b1916cdfa79ddcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - MAGIC
  - NMF
  - SC3
  - Seurat
  - scRNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scrnaseq-transfer-learning-clustering

## Summary

NMF-based transfer learning method that modifies a small scRNA-Seq target dataset with information from a large well-annotated source dataset before downstream clustering, to improve clustering of small disease- or tissue-specific datasets.

## When to use

Use when clustering a small scRNA-Seq target dataset (e.g. rare cell types, 300–600 cells) for which a large, well-annotated source dataset from a related tissue or context exists.

## When NOT to use

- Source and target share only a small set of transcripts — the article notes 'a loss of (probably) vital information is inevitable since only the set of genes present in both datasets can be used'
- No large, well-annotated source dataset exists for the tissue or disease context of interest
- Downstream analysis other than clustering is the goal — the article explicitly states other downstream methods besides SC3 are an open research direction

## Inputs

- Pre-processed source scRNA-Seq expression matrix with cell-type labels (e.g. Usoskin et al. 2014 CPM dataset, GSE59739, 622 cells) or labels generated via NMF clustering
- Pre-processed target scRNA-Seq expression matrix without reliable clustering (e.g. Hockley et al. 2018 TPM dataset, GSE102962, 314 cells)
- Optional: simulated count data from scRNA cmd_generate_data.py (10,000 genes, 1800 cells)
- Optional: Seurat batch-effect-removed or MAGIC-imputed versions of the data as extra pre-processing

## Outputs

- Modified target dataset Xtrg_new used as clustering input
- Predicted target cluster labels per cluster-range value (.tsv with cell ids)
- t-SNE plots and summarizing accuracy figures (.png)
- Source model files (.npz) per cluster number
- Robustness counts, e.g. how often clusters of interest are separated over 1000 replications (robustness_table.csv)

## How to apply

Pre-process both datasets with three steps: cell filter, gene filter (removal of ubiquitous/rare genes), and log-transformation after adding a pseudo-count of 1. Learn the source dictionary Hsrc and data matrix Wsrc via NMF with elastic net regularization, initialized with one-hot-encoded source labels (or labels generated via NMF clustering if none exist). Transfer knowledge by learning Wtrg given Hsrc, simplify it to the binary cluster-membership matrix W'trg using column-wise maxima (the article reports this simplified version performed best and most consistently), then construct Xtrg_new = theta*Hsrc*W'trg + (1-theta)*Xtrg with the mixture parameter theta selected automatically via Kernel Target Alignment (KTA) scores. Feed the modified target into SC3 and benchmark against clustering the untouched target (TargetCluster) and simple source+target concatenation (ConcatenateCluster); repeat SC3 many times (e.g. 1000) because it is non-convex. The scRNA Python framework implements all three stages as command-line scripts (scRNA-generate-data.sh, scRNA-source.sh, scRNA-target.sh).

## Related tools

- **scRNA** (Python framework implementing data simulation, NMF source modeling, and transfer learning into the target dataset (scRNA-source.sh, scRNA-target.sh)) — https://github.com/nicococo/scRNA
- **SC3** (Downstream clustering method applied to the modified target dataset Xtrg_new and to baselines (TargetCluster, ConcatenateCluster))
- **Seurat** (Optional batch effect removal pre-processing applied before the three clustering methods (R script seurat_cca_hockley_usoskin.R)) — https://github.com/nicococo/scRNA/blob/master/R/seurat_cca_hockley_usoskin.R
- **MAGIC** (Optional imputation pre-processing to overcome zero-inflation before clustering (Matlab script magic_hockley_usoskin.m)) — https://github.com/nicococo/scRNA/blob/master/matlab/magic_hockley_usoskin.m

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git; scRNA-generate-data.sh; scRNA-source.sh --data src --labels src_labels.tsv; scRNA-target.sh --model src_c8.npz --data trg
```

## Evaluation signals

- Adjusted Rand Index (ARI) of TransferCluster vs TargetCluster on cells with known ground truth: transfer deemed successful if TransferCluster ARI is not significantly lower (overlapping 95% confidence intervals)
- TransferCluster should outperform both TargetCluster and ConcatenateCluster baselines (e.g. on the mNP/mNFa cluster split)
- Robustness over 1000 SC3 replications: paper reports with MAGIC imputation 919/1000 mNP/mNFa, 944/1000 pPEP, 831/1000 pNF cluster identifications — majority separation expected
- KTA scores over the pre-specified theta range should yield a valid automatic theta selection; theta must lie in (0,1) for the convex combination
- Per-run outputs present: predicted label .tsv per cluster-range value, .npz source models, and summarizing .png accuracy figures

## Limitations

- SC3 is non-convex, so results must be repeated many times (1000 replications in the paper) to assess robustness
- If source and target share few genes, only shared genes can be transferred and vital information may be lost; handling differing gene sets is deferred to future work
- The method was demonstrated with SC3; extension to other downstream methods is left open
- No software version numbers are stated for SC3, Seurat, or MAGIC
- The pipeline scripts are MacOS and Linux only

## Evidence

- [abstract] Our approach modifies the dataset of interest while incorporating key information from the larger reference dataset via Non-negative Matrix Factorization (NMF): "Our approach modifies the dataset of interest while incorporating key information from the larger reference dataset via Non-negative Matrix Factorization (NMF)"
- [methods] NMF with elastic net, one-hot initialization: [evidence span withheld — anchor status: not_found]
- [methods] theta selection via KTA: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores76"
- [methods] simplified max-based W'trg performed best: "it was found that taking the simplified version as described above performed best and most consistently for all scenarios under investigation"
- [readme] installation via pip, three command-line scripts: "The *scRNA* software package can be installed using the _pip install git+https://github.com/nicococo/scRNA.git_ command."
- [readme] MAGIC imputation improves results: "We have also shown, that batch effect removal approaches (Butler et al.) and imputation methods (Van Dijk et al.) can be used to further improve clustering results when applying our method."
- [discussion] real-data transfer outperformed baselines: "transferring knowledge from a large well-annotated source dataset to a smaller target dataset was not only more successful than applying SC3 to the original target set alone"
