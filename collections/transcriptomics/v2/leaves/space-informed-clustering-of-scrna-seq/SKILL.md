---
name: space-informed-clustering-of-scrna-seq
description: Use when scRNA-seq cells have been assigned pseudo-space coordinates
  from a spatial transcriptomics reference (Visium, STARmap, Slide-seq) and the goal
  is to detect spatially variated subpopulations that traditional transcriptome-only
  clustering misses.
when_to_use_negative:
- Input is ST data rather than scRNA-seq — scSpace targets spatial analysis of scRNA-seq
  data, whereas SpaGCN/BayesSpace/DR-SC target spatial domain identification of ST
  data
- The tissue has low spatial coherence (e.g., SCC Patients 5 and 9) — reconstruction
  accuracy is compromised and scSpace should be applied cautiously
- No appropriate spatial reference exists for the target tissue's architecture (e.g.,
  using a DLPFC reference for agranular dACC lacking layer 4) without prior assessment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2228
- http://edamontology.org/topic_3170
tools:
- name: scSpace
  role: 'Core pipeline: TCA to eliminate batch effect and extract shared characteristic
    matrices, MLP to project cells into pseudo-space, and joint embedding of gene
    expression and spatial weight for space-informed cell typing'
  repo: github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Normalize/preprocess expression matrices and run space-informed Leiden clustering
    on the joint embedding
  repo: github.com/scverse/scanpy
- name: igraph
  role: Graph construction for Leiden community detection on the joint expression-spatial
    embedding (pinned 0.10.2)
- name: leidenalg
  role: Leiden algorithm used to assign cells to spatially informed subpopulations
    (pinned 0.9.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:18322faf3ccd91a2d25010f6f53734471c4f9a31633f894609b190411ce7cac2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Visium
  - igraph
  - leidenalg
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2228
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# space-informed-clustering-of-scrna-seq

## Summary

Identify spatially heterogeneous cell subpopulations in scRNA-seq data by jointly embedding gene expression with spatial weights derived from pseudo-space coordinates projected via scSpace's TCA + MLP pipeline, then clustering with Leiden.

## When to use

Use when scRNA-seq cells have been assigned pseudo-space coordinates from a spatial transcriptomics reference (Visium, STARmap, Slide-seq) and the goal is to detect spatially variated subpopulations that traditional transcriptome-only clustering misses.

## When NOT to use

- Input is ST data rather than scRNA-seq — scSpace targets spatial analysis of scRNA-seq data, whereas SpaGCN/BayesSpace/DR-SC target spatial domain identification of ST data
- The tissue has low spatial coherence (e.g., SCC Patients 5 and 9) — reconstruction accuracy is compromised and scSpace should be applied cautiously
- No appropriate spatial reference exists for the target tissue's architecture (e.g., using a DLPFC reference for agranular dACC lacking layer 4) without prior assessment

## Inputs

- Preprocessed scRNA-seq expression matrix (csv read by pandas) with cells assigned pseudo-space coordinates from scSpace's MLP projection
- Shared characteristic matrices from TCA of spatial transcriptomics reference and scRNA-seq data
- Spatial weight matrix encoding pseudo-space position of each cell

## Outputs

- Leiden cluster assignment of cells into spatially informed subpopulations
- Joint gene-expression/spatial-weight embedding used for cell typing
- Cluster-level spatial layout (e.g., dlpfc_pseudospace_results.csv) and visualization (matplotlib)

## How to apply

After scSpace's transfer component analysis (TCA) eliminates batch effect between the ST reference and scRNA-seq data and extracts shared characteristic matrices, train the multilayer perceptron (MLP) model to project cells into pseudo-space coordinates. The rationale is that spatially heterogeneous subpopulations are difficult to identify by traditional single-cell pipelines due to transcriptome similarity, so spatial position must be combined with expression. Embed gene expression and spatial weight of cells jointly, then run Leiden clustering (igraph 0.10.2 + leidenalg 0.9.0) on the joint embedding to assign cells to spatially informed subpopulations. Before clustering, perform an appropriateness assessment of the spatial reference — e.g., a DLPFC reference is unsuitable for agranular dACC which lacks cortical layer 4. Validate subpopulations by layer-restricted marker-gene expression and ordering in pseudo-space.

## Related tools

- **scSpace** (Core pipeline: TCA to eliminate batch effect and extract shared characteristic matrices, MLP to project cells into pseudo-space, and joint embedding of gene expression and spatial weight for space-informed cell typing) — github.com/ZJUFanLab/scSpace
- **scanpy** (Normalize/preprocess expression matrices and run space-informed Leiden clustering on the joint embedding) — github.com/scverse/scanpy
- **igraph** (Graph construction for Leiden community detection on the joint expression-spatial embedding (pinned 0.10.2))
- **leidenalg** (Leiden algorithm used to assign cells to spatially informed subpopulations (pinned 0.9.0))

## Examples

```
sc.tl.leiden(adata_joint, resolution=1.0, flavor='igraph', key_added='space_informed_clusters')
```

## Evaluation signals

- Pseudo-space shows significant positive pairwise-distance correlation with the spatial reference positions (scipy)
- Reconstructed pseudo-space reproduces expected cortical layer ordering with relative positions between layers and neighbor systems of spots preserved
- Cluster-restricted marker genes match known layer-specific expression patterns (Fig. 2a-d of the paper)
- Space-informed clustering assigns cells to subpopulations with both spatial coherence and transcriptional specificity

## Limitations

- Intrinsic difference in spatial homogeneity between tissues may influence the accuracy of scSpace
- The choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment should be performed first
- Traditional single-cell analysis struggles to identify spatially heterogeneous subpopulations due to transcriptome similarity, which is the motivation — but spatially coherent but transcriptionally distinct structures may be missed if the reference architecture mismatches

## Evidence

- [methods] joint embedding rationale: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [discussion] traditional clustering limitation: "However, they are difficult to identify by traditional single-cell analysis pipelines due to the similarity in transcriptome"
- [discussion] reference choice caveat: "performing an appropriateness assessment or necessary pre-processing steps of the spatial reference should be considered"
- [intro] TCA role: "scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data"
- [readme] README environment/dependency: "project single cells into a pseudo space via a multiple layer perceptron (MLP) model"
