---
name: nmf-dictionary-learning-with-elastic-net
description: Use when a well-annotated source scRNA-Seq dataset (e.g. Usoskin et al.
  2014, GSE59739, 622 cells) is available and its cluster labels should be encoded
  as a dictionary for subsequent transfer learning to a small target dataset.
when_to_use_negative:
- The source dataset lacks any reliable cluster labels and NMF-generated labels would
  not reflect known biology of the reference tissue
- Source and target share only a small set of transcripts, since only the genes present
  in both datasets can be used and vital information is lost
- The goal is direct joint analysis of source and target cells (e.g. Seurat CCA integration)
  rather than dictionary-based transfer without combining datasets
edam_operation: http://edamontology.org/operation_3645
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scRNA
  role: Python framework providing the elastic-net-regularized NMF source-learning
    step (scRNA-source.sh) initialized with one-hot-encoded source labels
  repo: https://github.com/nicococo/scRNA
- name: NMF
  role: Factorization method used on the source data to learn the dictionary Hsrc
    and data matrix Wsrc with elastic net regularization
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:c023bfc8e400cea96a47ccbf95de70299c16d8174483d11f8bd8f1f2abc6ab65
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
  edam_operation: http://edamontology.org/operation_3645
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nmf-dictionary-learning-with-elastic-net

## Summary

Learn a source dictionary Hsrc and data matrix Wsrc via elastic-net-regularized NMF on a well-annotated reference scRNA-Seq dataset, initialized from one-hot-encoded source labels; Hsrc then serves as the knowledge carrier for transfer to a smaller target dataset.

## When to use

Use when a well-annotated source scRNA-Seq dataset (e.g. Usoskin et al. 2014, GSE59739, 622 cells) is available and its cluster labels should be encoded as a dictionary for subsequent transfer learning to a small target dataset.

## When NOT to use

- The source dataset lacks any reliable cluster labels and NMF-generated labels would not reflect known biology of the reference tissue
- Source and target share only a small set of transcripts, since only the genes present in both datasets can be used and vital information is lost
- The goal is direct joint analysis of source and target cells (e.g. Seurat CCA integration) rather than dictionary-based transfer without combining datasets

## Inputs

- Pre-processed source scRNA-Seq expression matrix (e.g. Usoskin et al. 2014, GSE59739, CPM, 622 cells) after cell filter, gene filter, and log(pseudo-count 1) transformation
- Gene id list shared between source and target (e.g. 4402 common genes for Usoskin/Hockley)
- Optional one-hot-encoded source cluster labels from the original publication, or no labels (NMF clustering generates them)

## Outputs

- Source dictionary Hsrc (metagene-by-cluster basis used for knowledge transfer)
- Source data matrix Wsrc
- Source model file in .npz format for each cluster number in the cluster range (e.g. src_c8.npz)
- True and NMF model cluster labels per cell (.tsv) and t-SNE diagnostic plots

## How to apply

After standard pre-processing (cell filter, gene filter removing ubiquitous/rare genes, log-transformation after adding a pseudo-count of 1), run NMF on the source expression matrix Xsrc to learn a dictionary Hsrc and a data matrix Wsrc, regularizing the denseness of the results with an elastic net. Initialize the NMF with one-hot-encoded source labels — either curated labels from the original publication (e.g. the 11 level-3 clusters of Usoskin et al. 2014) or labels generated via NMF clustering when publication labels are unavailable. The learned Hsrc encodes cluster-specific expression patterns (metagenes) and is later fixed when learning the target data matrix Wtrg given Hsrc. In the scRNA Python framework this corresponds to the scRNA-source.sh step, which outputs source model files (.npz) per cluster number in the chosen --cluster-range (default 6,7,8), true and model cluster labels (.tsv), and t-SNE plots; --no-tsne can speed up runs.

## Related tools

- **scRNA** (Python framework providing the elastic-net-regularized NMF source-learning step (scRNA-source.sh) initialized with one-hot-encoded source labels) — https://github.com/nicococo/scRNA
- **NMF** (Factorization method used on the source data to learn the dictionary Hsrc and data matrix Wsrc with elastic net regularization)

## Examples

```
scRNA-source.sh --input /foo/source_data.tsv --genes /foo/gene_ids.tsv --labels /foo/source_labels.tsv --cluster-range 6,7,8 --no-tsne
```

## Evaluation signals

- NMF-reconstructed source clusters (model labels) should agree with the one-hot-encoded initialization labels or, when labels are generated via NMF clustering, t-SNE plots per cluster number should show coherent separated clusters
- The resulting model files (.npz, e.g. src_c8.npz) exist for every cluster number in --cluster-range and load without error in the target transfer step
- The elastic-net regularization yields a non-degenerate dictionary: Hsrc columns (metagenes) should be non-negative and correspond to distinct source clusters
- Downstream check: fixing Hsrc and learning Wtrg on a target dataset should produce a W'trg binary membership matrix whose clusters align with expected target cell types (e.g. mNP/mNFa, pPEP, pNF in the Hockley application)

## Limitations

- If Xsrc and Xtrg only share a small set of transcripts, a loss of vital information is inevitable since only the genes present in both datasets can be used
- NMF with a cluster-range sweep can be slow depending on the data and cluster range; t-SNE plotting can be disabled via --no-tsne to speed it up
- MKL runtime issue with Anaconda ('Intel MKL FATAL ERROR: Cannot load libmkl_avx.so') requires conda install mkl

## Evidence

- [methods] We use NMF73,74 of our source data ... to learn a dictionary Hsrc and a data matrix Wsrc while regularizing the denseness of the results with an elastic net: [evidence span withheld — anchor status: not_found]
- [methods] Log-transformation: Log-transform the expression matrix after adding a pseudo-count of 1: "Log-transformation: Log-transform the expression matrix after adding a pseudo-count of 1"
- [readme] You can provide source labels to be used as a starting point for NMF. If not those labels will be generated via NMF Clustering.: "You can provide source labels to be used as a starting point for NMF. If not those labels will be generated via NMF Clustering."
- [readme] Output source model in .npz format for every number of cluster as specified in the --cluster-range argument (default 6,7,8): "Output source model in .npz format for every number of cluster as specified in the --cluster-range argument (default 6,7,8)"
