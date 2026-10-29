---
name: cell-shape-embedding-visualization
description: Use when MIRIAM autoencoder shape latent vectors are available for held-out
  cells and you need a 2D t-SNE embedding to test whether similar cell geometries
  cluster, or whether p-ERK-high cells form a discrete shape-defined population.
when_to_use_negative:
- Input is already a marker-intensity-based t-SNE (Figure 3A style) — use marker intensities
  directly instead of shape latent vectors
- Latent vectors include cells used to train the autoencoder — the skill is defined
  on held-out (non-training) latent vectors only
- Data comes from a non-multiplexed or non-tissue imaging modality where per-cell
  compartments (nucleus, membrane, cytoplasm) were not quantified by MIRIAM
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Produces the autoencoder shape latent vectors and per-cell marker intensity
    quantifications that serve as t-SNE input
  repo: github.com/Coffey-Lab/MIRIAM
- name: Rtsne
  role: R package used to run t-SNE on the shape latent vectors to produce the 2D
    embedding
- name: Python
  role: Used with pandas/scipy to align latent vectors with marker intensity CSVs
    cell-by-cell and compute Spearman rank correlations
- name: Matlab
  role: Alternative implementation environment for the MIRIAM pipeline generating
    the latent vectors
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:aac8e88b838c9e12a70820beaabe93f9d59224eae1715dbbfb1dd9eec97193df
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Python
  - Rtsne
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cell-shape-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run t-SNE on MIRIAM autoencoder-encoded cell shape latent vectors (held-out cells only) to produce a 2D embedding, then verify that geometrically similar colon adenoma cells co-localize and overlay marker intensities (p-ERK, p-EGFR, PCNA, Ki67) to test whether shape maps to discrete phenotypic populations.

## When to use

Use when MIRIAM autoencoder shape latent vectors are available for held-out cells and you need a 2D t-SNE embedding to test whether similar cell geometries cluster, or whether p-ERK-high cells form a discrete shape-defined population.

## When NOT to use

- Input is already a marker-intensity-based t-SNE (Figure 3A style) — use marker intensities directly instead of shape latent vectors
- Latent vectors include cells used to train the autoencoder — the skill is defined on held-out (non-training) latent vectors only
- Data comes from a non-multiplexed or non-tissue imaging modality where per-cell compartments (nucleus, membrane, cytoplasm) were not quantified by MIRIAM

## Inputs

- MIRIAM pipeline output: autoencoder-encoded latent vectors for cell shape (held-out cells, excluding training vectors)
- Per-cell median marker intensity CSVs from the MIRIAM quantification stage (colonic adenoma multiplexed immunofluorescence data)

## Outputs

- 2D t-SNE embedding of cells by shape similarity (Rtsne)
- Annotated t-SNE embedding figure colored by marker intensity (tsne_shape_latent_figure3c.png)
- Optional: shape latent dimension x marker Spearman correlation matrix with multiple-testing-corrected p-values (shape_marker_correlations.csv)

## How to apply

Load the encoded latent vectors for cell shape from the MIRIAM pipeline output, explicitly excluding the latent vectors used to train the autoencoder. Run t-SNE on these shape latent vectors with the Rtsne package to produce a 2D embedding of cells by shape similarity. Color the embedding by per-cell marker intensities to confirm that cells of similar geometry are located near each other in t-SNE space. Overlay p-ERK staining intensity to check whether p-ERK-high cells form a discrete cluster, then compare that cluster's overlap against the p-EGFR, PCNA, and Ki67 distributions reported for Figure 3C. In the colonic adenoma setting, the paper found cells of similar shape co-localized in the embedding, but the shape-based t-SNE revealed no discernible cellular populations or clear patterns in marker expression — so an empty/population-free embedding can be a valid, expected outcome rather than a failure.

## Related tools

- **MIRIAM** (Produces the autoencoder shape latent vectors and per-cell marker intensity quantifications that serve as t-SNE input) — github.com/Coffey-Lab/MIRIAM
- **Rtsne** (R package used to run t-SNE on the shape latent vectors to produce the 2D embedding)
- **Python** (Used with pandas/scipy to align latent vectors with marker intensity CSVs cell-by-cell and compute Spearman rank correlations)
- **Matlab** (Alternative implementation environment for the MIRIAM pipeline generating the latent vectors)

## Examples

```
Rscript -e 'library(Rtsne); tsne <- Rtsne(shape_latents, dims=2, check_duplicates=FALSE); plot(tsne$Y, col=marker_color, main="shape latent t-SNE")'
```

## Evaluation signals

- Cells with similar geometries are located near each other in t-SNE space when shape latent vectors are the only input (positive control for the embedding)
- The embedding was built exclusively from held-out latent vectors — verify training-set latent vectors were excluded before running Rtsne
- Overlaying p-ERK intensity should be checked against the Figure 3C expectation that p-ERK-high cells occupy a discrete cluster overlapping p-EGFR, PCNA, and Ki67 distributions
- Correlation summary (shape_marker_correlations.csv) should report corrected p-values alongside Spearman coefficients; absence of significant corrected associations matches the paper's reported negative result
- Output figure tsne_shape_latent_figure3c.png is produced with marker-intensity color annotations

## Limitations

- Shape-based t-SNE analysis did not reveal any discernible cellular populations or clear patterns in marker expression — cell shape may not correspond to marker signal intensity in this setting
- No software version numbers stated for Matlab or Python implementations, so reproducibility of the exact environment is uncertain
- Example data in the repo omits image stacks (too large), which must be regenerated by the user from a Dropbox download

## Evidence

- [results] t-SNE on held-out shape latent vectors clusters similar geometries but reveals no marker-expression populations: "When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B)"
- [results] Shape-based t-SNE negative finding for populations/marker patterns: "However, our t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5)."
- [results] t-SNE via marker intensities as the alternative Figure 3A analysis: "Figure 3A shows the results of t-stochastic neighbor embedding (t-SNE) using only the intensities for each marker in the data set."
- [readme] Repo tooling context: Matlab pipeline and ilastik dependency: "Matlab code to run cell segmentation on multiplexed imaging data. This pipeline requires Iltastik for some tasks."
- [intro] Autoencoder shape characterization upstream of the embedding: "Finally, cell shapes are characterized using an autoencoder neural network."
