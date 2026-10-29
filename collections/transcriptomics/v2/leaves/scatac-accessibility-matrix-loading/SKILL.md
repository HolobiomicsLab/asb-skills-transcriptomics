---
name: scatac-accessibility-matrix-loading
description: Use when an analysis needs to run VIA or other TI methods on Buenrostro
  scATAC-seq hematopoiesis data preprocessed with TF-motif or k-mer Z Scores protocols
  and must first load these matrices into a VIA-compatible AnnData container.
when_to_use_negative:
- Input is raw scATAC-seq fragment files or FASTQ/bam rather than already-preprocessed
  motif or k-mer z-score matrices
- The target method does not consume adata.obsm['X_pca'] (e.g. a tool requiring raw
  counts in adata.X)
- Working with non-hematopoiesis scATAC data not covered by the VIA Datasets folder
  loaders in test_pyVIA.py
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Downstream consumer of the assembled AnnData; via_wrapper expects PC projections
    in adata.obsm['X_pca']
  repo: https://github.com/ShobiStassen/VIA
- name: scanpy
  role: Provides sc.AnnData used to assemble the loaded matrices into the analysis
    container
- name: pandas
  role: Reads the Buenrostro TF-motif and k-mer z-score matrices from the VIA Datasets
    folder
- name: UMAP
  role: Provides the embedding used for single-cell level visualization after loading
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:cc2d3bacc7e7defac7058f5579563d1e3dafd5ebffccf51bbc2a6e71eb3fe27b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - UMAP
  - VIA
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scatac-accessibility-matrix-loading

## Summary

Loads scATAC-seq hematopoiesis accessibility matrices (TF-motif z-scores and ChromVAR k-mer accessibility z-scores) from the VIA Datasets folder with pandas and assembles them into an AnnData object whose PC projections are placed in adata.obsm['X_pca'], the format required by pyVIA's via_wrapper for trajectory inference on Buenrostro et al. (2018) data.

## When to use

Use when an analysis needs to run VIA or other TI methods on Buenrostro scATAC-seq hematopoiesis data preprocessed with TF-motif or k-mer Z Scores protocols and must first load these matrices into a VIA-compatible AnnData container.

## When NOT to use

- Input is raw scATAC-seq fragment files or FASTQ/bam rather than already-preprocessed motif or k-mer z-score matrices
- The target method does not consume adata.obsm['X_pca'] (e.g. a tool requiring raw counts in adata.X)
- Working with non-hematopoiesis scATAC data not covered by the VIA Datasets folder loaders in test_pyVIA.py

## Inputs

- Buenrostro scATAC-seq hematopoiesis TF-motif z-score matrix (from VIA Datasets folder)
- ChromVAR k-mer accessibility z-score matrix (from VIA Datasets folder)
- pandas-readable delimited/CSV accessibility score files

## Outputs

- AnnData object with PC projections in adata.obsm['X_pca']
- UMAP or PHate embedding for single-cell trajectory visualization

## How to apply

Retrieve the Buenrostro scATAC-seq hematopoiesis matrices (TF-motif z-scores and ChromVAR k-mer accessibility z-scores) from the VIA Datasets folder (github.com/ShobiStassen/VIA/tree/master/Datasets) using pandas, following the run logic of the packaged test_pyVIA.py script. Assemble the loaded values into a scanpy AnnData object and write the PC projections into adata.obsm['X_pca'], which is the exact placement demonstrated in the paper's methods (adata = sc.AnnData(data_genes); adata.obsm['X_pca'] = data.values). Use a UMAP or PHate embedding for downstream single-cell visualization, and prepare to rerun across increasing PC counts (e.g. 10–100 PCs) for both input protocols so VIA's robustness beyond 50 PCs can be assessed. Verify the load by confirming the matrices feed into via.via_wrapper with documented defaults such as knn=30, edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, and pseudotime_threshold_TS=30.

## Related tools

- **VIA** (Downstream consumer of the assembled AnnData; via_wrapper expects PC projections in adata.obsm['X_pca']) — https://github.com/ShobiStassen/VIA
- **scanpy** (Provides sc.AnnData used to assemble the loaded matrices into the analysis container)
- **pandas** (Reads the Buenrostro TF-motif and k-mer z-score matrices from the VIA Datasets folder)
- **UMAP** (Provides the embedding used for single-cell level visualization after loading)

## Examples

```
import pandas as pd, scanpy as sc; data = pd.read_csv('Datasets/buenrostro_kmer_zscores.csv'); adata = sc.AnnData(data_genes); adata.obsm['X_pca'] = data.values
```

## Evaluation signals

- adata.obsm['X_pca'] is populated with the loaded matrix values (shape matches cell count x PC count)
- Loaded object feeds successfully into via.via_wrapper with knn=30, edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30 without errors
- Reruns across 10–100 PCs for both TF-motif and k-mer Z Scores protocols execute reproducibly
- Downstream VIA outputs (pseudotime, terminal states, single_cell_pt_markov, single_cell_bp) reproduce the MPP/LMPP/CMP to CLP/MEP/GMP lineage hierarchy
- Reproduced figure matches the reference scATAC_BuenrostroPCs_MainFig.png hierarchy and scatac_buenrostro_f1_table.csv exists

## Limitations

- Applies to the preprocessed Buenrostro scATAC-seq hematopoiesis matrices distributed in the VIA Datasets folder, not raw scATAC-seq sequencing data
- The article gives no accession identifiers for the hematopoiesis validation datasets, so source retrieval must go through the Datasets folder
- Loading alone does not guarantee good trajectory results: parameter choices (e.g. number of PCs) affect baseline TI methods, which degrade beyond 50 PCs on k-mer Z Scores input

## Evidence

- [other] Load the Buenrostro scATAC-seq hematopoiesis matrices (TF-motif z-scores and ChromVAR k-mer accessibility z-scores) from the VIA Datasets folder using pandas: [evidence span withheld — anchor status: not_found]
- [methods] AnnData with PCA in obsm: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] Datasets folder availability: "The data for the Jupyter Notebooks and Examples are available in the `Datasets folder"
- [readme] README scATAC dataset availability: "scATAC-seq Hematopoiesis</a></td>
<td>Human hematopoiesis</td>
<td><a href="https://github.com/ShobiStassen/VIA/tree/master/Datasets">scATAC-seq</a>"
