---
name: neural-network-deconvolution-workflow-execution
description: Use when you have a paired scRNA-seq reference and spatial transcriptomics
  dataset (e.g. 10x Visium spots) and need cell type deconvolution via a trained neural
  network, including benchmarking runtime/RAM against RCTD and cell2location.
when_to_use_negative:
- No single-cell RNA-seq reference is available for the tissue — SpatialDDLS requires
  scRNA-seq data to simulate mixed transcriptional profiles
- Input is bulk RNA-seq or single-cell data without spatial coordinates — the skill
  targets spatial transcriptomics spot deconvolution
- A deterministic, non-TensorFlow environment is required — the package depends on
  the tensorflow and keras R packages with a working Python interpreter
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpatialDDLS
  role: Core package implementing object construction, mixed-profile simulation, neural
    network training, and spatial deconvolution
  repo: https://github.com/diegommcc/SpatialDDLS
- name: SpatialDDLSdata
  role: Data package supplying the MouseDLN.SCE scRNA-seq reference and MouseDLN.ST
    Visium slides
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: SingleCellExperiment
  role: Container for the scRNA-seq reference loaded into createSpatialDDLSobject
- name: SpatialExperiment
  role: Container for the spatial transcriptomics slides loaded into createSpatialDDLSobject
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:462e796c151cb13fdddd69c44cf5bc4e353016f2208095bc1c51157dbb627ca3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/diegommcc/SpatialDDLS
  tools:
  - R
  - RCTD
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - cell2location
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# neural-network-deconvolution-workflow-execution

## Summary

End-to-end execution of the SpatialDDLS pipeline: load scRNA-seq (SingleCellExperiment) and spatial (SpatialExperiment) data into a SpatialDDLS object, simulate mixed transcriptional profiles, train a fully-connected neural network with TensorFlow/Keras, and deconvolute 10x Visium spots into cell type proportions.

## When to use

Use when you have a paired scRNA-seq reference and spatial transcriptomics dataset (e.g. 10x Visium spots) and need cell type deconvolution via a trained neural network, including benchmarking runtime/RAM against RCTD and cell2location.

## When NOT to use

- No single-cell RNA-seq reference is available for the tissue — SpatialDDLS requires scRNA-seq data to simulate mixed transcriptional profiles
- Input is bulk RNA-seq or single-cell data without spatial coordinates — the skill targets spatial transcriptomics spot deconvolution
- A deterministic, non-TensorFlow environment is required — the package depends on the tensorflow and keras R packages with a working Python interpreter

## Inputs

- SingleCellExperiment scRNA-seq reference (e.g. MouseDLN.SCE from GSE173778)
- SpatialExperiment spatial transcriptomics data (e.g. MouseDLN.ST, 10x Visium slides)
- Python environment with the TensorFlow library (spatialddls-env conda env via installTFpython)

## Outputs

- SpatialDDLS object with simulated mixed profiles and trained deconvolution model
- Predicted per-spot cell type proportions for the spatial slides
- Model evaluation metrics (MAE, MSE per cell type) from calculateEvalMetrics

## How to apply

Load the murine lymph node 10x Visium ST data (MouseDLN.ST) and paired 10x Chromium scRNA-seq reference (MouseDLN.SCE) from the SpatialDDLSdata package, then build a SpatialDDLS object with createSpatialDDLSobject using sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, and sc.min.mean.counts = 2; expect shared-gene selection (~10941 shared genes) after mitochondrial-gene removal and logFC-based filtering. Simulate cell composition matrices with genMixedCellProp (min.zero.prop = 5) and generate 10,000 mixed transcriptional profiles of 50 cells each via simMixedProfiles (threads = 3). Train the fully-connected network for 60 epochs with trainDeconvModel (requires a Python interpreter with TensorFlow, installable via installTFpython), and deconvolute the ST slides with deconvSpatialDDLS (index.st = 1, k.spots = 6, fast.pca = TRUE) to obtain per-spot cell type proportions. Evaluate the model with calculateEvalMetrics (MAE and MSE per cell type) and confirm predicted proportions colocalize with known cell type marker expression.

## Related tools

- **SpatialDDLS** (Core package implementing object construction, mixed-profile simulation, neural network training, and spatial deconvolution) — https://github.com/diegommcc/SpatialDDLS
- **SpatialDDLSdata** (Data package supplying the MouseDLN.SCE scRNA-seq reference and MouseDLN.ST Visium slides) — https://github.com/diegommcc/SpatialDDLSdata
- **SingleCellExperiment** (Container for the scRNA-seq reference loaded into createSpatialDDLSobject)
- **SpatialExperiment** (Container for the spatial transcriptomics slides loaded into createSpatialDDLSobject)

## Examples

```
mouseDLN.SDDLS <- createSpatialDDLSobject(sc.data = MouseDLN.SCE, st.data = MouseDLN.ST, sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2)
```

## Evaluation signals

- Filter log reports show the expected sequence: mitochondrial genes removed, genes without expression in any cell dropped, and ~10941 shared genes between single-cell and spatial datasets
- The trained model achieves satisfactory performance on the test subset (MAE/MSE per cell type from calculateEvalMetrics)
- Predicted cell type proportions colocalize with expression of known cell type markers in the Visium slides
- Extrinsic predictions are a smoothed version of the final spot predictions
- In benchmarking, runtime is comparable to RCTD while peak RAM consumption is lower than both RCTD and cell2location

## Limitations

- Requires a working Python interpreter with the TensorFlow Python library installed (conda env spatialddls-env); setup failures block all training
- Filtration choices (sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2, logFC-based filtering, min.zero.prop) strongly shape the simulated profiles and downstream model
- Mixed-profile simulation (10,000 profiles of 50 cells) is computationally intensive; parallelize with threads (e.g. threads = 3)

## Evidence

- [abstract] we introduce SpatialDDLS, a fast neural network-based algorithm for cell type deconvolution of spatial transcriptomics data: "we introduce SpatialDDLS, a fast neural network-based algorithm for cell type deconvolution of spatial transcriptomics data"
- [methods] sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2: "sc.filt.genes.cluster = TRUE, 
  sc.n.genes.per.cluster = 150,
  sc.min.mean.counts = 2"
- [methods] mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3): "mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3)"
- [methods] This function computes mean absolute error (MAE) and mean squared error (MSE) metrics per cell type: "This function computes mean absolute error (MAE) and mean squared error (MSE) metrics per cell type"
- [readme] The package depends on the tensorflow and keras R packages, so a working Python interpreter with the Tensorflow Python library installed is needed.: [evidence span withheld — anchor status: not_found]
- [methods] markers for each cell type colocalize with the cell proportions predicted by `SpatialDDLS`: "markers for each cell type colocalize with the cell proportions predicted by `SpatialDDLS`"
