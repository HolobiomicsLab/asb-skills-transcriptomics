---
name: scatac-seq-peak-accessibility-processing
description: Use when a scATAC-seq peak-by-cell count matrix must be reduced to an
  informative peak subset before clustering or trajectory inference, and standard
  HVF selection is suspected to obscure cell type structure.
when_to_use_negative:
- The input is already a reduced or curated feature set, so correlation-based reselection
  adds no information.
- The analysis is supervised cell-type classification (e.g., SingleR-style annotation
  against a reference) rather than unsupervised feature selection for clustering or
  trajectory inference.
- The data modality is bulk ATAC-seq or other non-single-cell accessibility data where
  the cell-level correlation structure DUBStepR relies on is absent.
tools:
- name: DUBStepR
  role: Correlation-based feature selection of informative peaks from the log-normalized
    peak accessibility matrix
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Container and processing environment for the peak-by-cell data; recommended
    pairing per the article's methods
  repo: https://satijalab.org/seurat/
- name: Monocle 3
  role: Trajectory inference on PCA/UMAP embeddings to compare differentiation topologies
    across peak sets
- name: R
  role: Analysis language for running DUBStepR (requires R >= 3.5.0)
- name: hdf5r
  role: Loads the count data into the Seurat object from hdf5 files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:7ee7f35955e3895164f5e302f77f19bf724aad12720386e1da2b44cef869055c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - Monocle 3
  - R
  - Seurat
  - hdf5r
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scatac-seq-peak-accessibility-processing

## Summary

Applies DUBStepR correlation-based feature selection to single-cell ATAC-seq peak-by-cell accessibility matrices (e.g., Buenrostro et al. human bone marrow, GSE96583), adapting the gene-based algorithm to peaks, and evaluates selected peaks by whether downstream Monocle 3 trajectories recover known differentiation topology.

## When to use

Use when a scATAC-seq peak-by-cell count matrix must be reduced to an informative peak subset before clustering or trajectory inference, and standard HVF selection is suspected to obscure cell type structure.

## When NOT to use

- The input is already a reduced or curated feature set, so correlation-based reselection adds no information.
- The analysis is supervised cell-type classification (e.g., SingleR-style annotation against a reference) rather than unsupervised feature selection for clustering or trajectory inference.
- The data modality is bulk ATAC-seq or other non-single-cell accessibility data where the cell-level correlation structure DUBStepR relies on is absent.

## Inputs

- scATAC-seq peak-by-cell count matrix (e.g., Buenrostro et al. human bone marrow, GEO accession GSE96583)
- Seurat object of peak accessibility data (loadable with the hdf5r package)

## Outputs

- DUBStepR-selected informative peak set
- Comparator all-peaks and HVF peak sets
- PCA/UMAP cell embeddings per peak set
- Monocle 3 differentiation trajectories per peak set

## How to apply

Download the peak-by-cell count matrix (e.g., Buenrostro et al. human bone marrow scATAC-seq from GEO accession GSE96583) and load it into R, optionally as a Seurat object via hdf5r. Log-normalize the peak accessibility matrix, then run DUBStepR with optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, and min.cells set to 5% of cells, adapting the gene-based call to peaks. Independently define comparator feature sets: the all-peaks set and a highly-variable-feature (HVF) peak set. For each feature set, run PCA followed by UMAP embedding and Monocle 3 trajectory inference, then compare the inferred topologies. Rationale: DUBStepR exploits structure in the feature-feature correlation matrix — prioritizing well-correlated marker features via correlation range z-scores and stepwise regression — which the paper reports transfers to single-cell ATAC-seq.

## Related tools

- **DUBStepR** (Correlation-based feature selection of informative peaks from the log-normalized peak accessibility matrix) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Container and processing environment for the peak-by-cell data; recommended pairing per the article's methods) — https://satijalab.org/seurat/
- **Monocle 3** (Trajectory inference on PCA/UMAP embeddings to compare differentiation topologies across peak sets)
- **R** (Analysis language for running DUBStepR (requires R >= 3.5.0))
- **hdf5r** (Loads the count data into the Seurat object from hdf5 files)

## Examples

```
sel.peaks <- DUBStepR(logNormalize(peak.data), min.cells = 0.05 * ncol(peak.data), species = "Homo_sapiens", optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The DUBStepR-selected peak set's Monocle 3 trajectory recovers the three-branch lymphoid/myeloid/megakaryocyte-erythroid topology matching Fig 7.
- The all-peaks and HVF comparator peak sets fail to recover that branching topology, confirming the selection step (not the embedding) drives the result.
- DUBStepR selects a relatively small number of features (the paper notes it approaches maximum silhouette index at 200–500 feature genes), so a selection retaining nearly all peaks signals misconfiguration.
- The DUBStepR call uses the paper's parameters: optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, min.cells = 5% of cells.
- Selected peaks correspond to coherent correlation structure (well-correlated marker peaks), consistent with the guilt-by-association expansion step.

## Limitations

- DUBStepR was designed for scRNA-seq gene expression; its application to scATAC-seq peaks is an adaptation, and the paper provides only the statement that it 'can be straightforwardly applied' without scATAC-specific parameter tuning guidance.
- Key quantitative details (elbow point computation in Supp. Note; silhouette results in Supp. Fig. S3) are deferred to supplementary material not in the main text.
- No quantitative thresholds or effect sizes are reported for scalability claims in the main text.
- Existing feature selection methods can perform inconsistently across datasets, so per-dataset validation against known topology (as done here) is required.

## Evidence

- [abstract] DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq.: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [methods] DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error): "DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)"
- [readme] After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering.: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [intro] DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features: "DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features"
