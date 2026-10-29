---
name: gene-module-detection
description: Use when the input is a preprocessed scRNA-seq expression matrix (rows
  = cells, columns = genes) with cell metadata and the goal is to infer gene co-expression
  networks and detect trait-associated gene modules.
when_to_use_negative:
- The input is bulk RNA-seq or already-aggregated pseudobulk expression profiles where
  standard WGCNA applies directly without the single-cell sub-clustering/downsampling
  step
- The goal is co-expression on raw unaggregated single-cell counts without the pseudo-bulk
  pre-processing SCANet expects
- A causal gene regulatory network is required — GCNs rely on correlation, not causation,
  and converting GCNs into GRNs may increase the risk of false negatives
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SCANet
  role: All-in-one Python package orchestrating pseudo-bulk pre-processing, modified-WGCNA
    GCN inference, and co-expression gene module detection from single-cell data
  repo: https://github.com/oubounyt/SCANet
- name: Scanpy
  role: Loads the preprocessed scRNA-seq expression matrix (rows = cells, columns
    = genes) with cell-level metadata
- name: BioNERO
  role: Wraps WGCNA to transform the gene-gene correlation matrix into an adjacency
    matrix using the default signed network
- name: WGCNA
  role: Modified version used for co-expression network inference and module detection
    on pseudo-bulk single-cell data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:679d7d330077395f0f183982d8d7a25155108c3c76c118b38e2138feabbb9dd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - Python
  - SCANet
  - Scanpy
  - WGCNA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-module-detection

## Summary

Detect co-expression gene modules from scRNA-seq data using a modified version of WGCNA inside SCANet, after reducing single-cell data to pseudo-bulk sub-cluster representatives. This yields trait/cell-type-associated gene modules whose hubs and scale-free topology can be validated.

## When to use

Use when the input is a preprocessed scRNA-seq expression matrix (rows = cells, columns = genes) with cell metadata and the goal is to infer gene co-expression networks and detect trait-associated gene modules.

## When NOT to use

- The input is bulk RNA-seq or already-aggregated pseudobulk expression profiles where standard WGCNA applies directly without the single-cell sub-clustering/downsampling step
- The goal is co-expression on raw unaggregated single-cell counts without the pseudo-bulk pre-processing SCANet expects
- A causal gene regulatory network is required — GCNs rely on correlation, not causation, and converting GCNs into GRNs may increase the risk of false negatives

## Inputs

- Preprocessed scRNA-seq expression matrix (rows = cells, columns = genes) with cell-level metadata (e.g. processed data from Cillo et al. 2021, GSE180578)
- Cell-type annotations for sub-clustering cells into pseudo-bulk representatives

## Outputs

- Co-expression gene modules (GCN-derived), e.g. 30–195 genes per module
- Gene-gene adjacency matrix (signed network) from WGCNA/BioNERO
- Network hubs per module (top 10% most connected genes)
- Scale-free topology fit assessment of each module graph
- Module-to-trait/cell-type correlation assignments (e.g. M11 with deceased, M21 with surviving patients)

## How to apply

Load the preprocessed scRNA-seq expression matrix with cell-level metadata using Scanpy, then sub-cluster each cell type into N sub-clusters and downsample by averaging gene expression across all cells in each sub-cluster to create representative pseudo-bulk cells. Build the pairwise gene-gene correlation matrix (Pearson's r, Spearman's, or biweight midcorrelation) and transform it into an adjacency matrix with WGCNA via the BioNERO framework, using the default signed network. Detect co-expression gene modules from the resulting GCN (in the Cillo et al. 2021 classical monocyte analysis this produced 27 modules with 30–195 genes per module). Identify network hubs as the intersection of the top 10% most connected genes within each module, and check whether the module graph fits the scale-free topology characteristic of real-world biological networks. Rationale: aggregating read counts across neighboring cells reduces single-cell sparsity before network inference, and the signed WGCNA network preserves positive co-expression relationships relevant to module-trait association.

## Related tools

- **SCANet** (All-in-one Python package orchestrating pseudo-bulk pre-processing, modified-WGCNA GCN inference, and co-expression gene module detection from single-cell data) — https://github.com/oubounyt/SCANet
- **Scanpy** (Loads the preprocessed scRNA-seq expression matrix (rows = cells, columns = genes) with cell-level metadata)
- **BioNERO** (Wraps WGCNA to transform the gene-gene correlation matrix into an adjacency matrix using the default signed network)
- **WGCNA** (Modified version used for co-expression network inference and module detection on pseudo-bulk single-cell data)

## Examples

```
pip install scanet; import scanet as sn  # full workflow: https://github.com/oubounyt/SCAn/blob/main/docs/full-example.ipynb (3k PBMCs from 10x Genomics)
```

## Evaluation signals

- The inferred GCN yields gene co-expression modules comparable in scale to the 30–195 genes per module reported for classical monocytes (27 modules total)
- Each module graph passes the scale-free topology check, a characteristic of real-world biological networks
- Network hubs are recovered as the intersection of the top 10% most connected genes within each module
- Modules show trait/cell-type association signals (e.g. M11 highest positive correlation with deceased patients, M21 with surviving patients)
- Module sizes are reported (e.g. M11 = 90 genes, M21 = 137 genes) so downstream correlation thresholds (≥ 0.9 for M11 GCN, ≥ 0.8 for GCN–drug interactions) can be applied consistently

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- Module selection requires following the 'Modules selection guidelines' in Supplementary File 1; arbitrary module choice risks misinterpreting trait associations

## Evidence

- [intro] SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection: "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
- [intro] This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data: "This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data"
- [results] We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C): "We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C)"
- [methods] users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module: "users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module"
- [methods] They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks: "They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks"
- [readme] a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data: "a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data"
