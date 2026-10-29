---
name: multiplexed-imaging-cell-phenotyping
description: Use when analyzing multiplexed immunofluorescence or MIBI tissue images
  where cells have irregular/elongated morphologies (e.g., columnar epithelial cells)
  and per-cell marker intensity plus shape phenotyping is needed.
when_to_use_negative:
- Cells of interest have regular, round morphologies where simpler segmentation (nuclear
  dilation, Voronoi tessellation) suffices and the internal-membrane resegmentation
  step adds unnecessary complexity
- Input is already a per-cell feature/intensity table — skip segmentation and go directly
  to dimensionality reduction or correlation analysis
- The goal is purely intensity-based phenotyping (t-SNE on marker intensities, Figure
  3A) rather than testing shape-marker correspondence
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: 'Core pipeline: pixel classification, cell resegmentation of internal aberrant
    membranes, per-compartment marker quantification, and autoencoder shape characterization'
  repo: github.com/Coffey-Lab/MIRIAM
- name: Rtsne
  role: R package used to produce the 2D t-SNE embedding of cells from shape latent
    vectors or marker intensities
- name: Python
  role: Implementation of the MIRIAM pipeline (interchangeable with Matlab) and used
    with pandas/scipy for Spearman correlations and multiple-testing correction
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Primary implementation for running cell segmentation on multiplexed imaging
    data per the repository README
  repo: github.com/Coffey-Lab/MIRIAM
- name: ilastik
  role: Required by the pipeline for some tasks (interactive machine-learning pixel
    classification)
  repo: https://www.ilastik.org/download.html
- name: Mesmer
  role: Deep learning segmentation baseline for benchmarking MIRIAM segmentation quality
    (DSC/JSC comparison)
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:f0ec3842584e71910c8f9939b6c9debd7f291b6b5f06418e6e7c93f17f220820
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
  - Mesmer
  - Python
  - Rtsne
  - ilastik
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multiplexed-imaging-cell-phenotyping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Single-cell phenotyping of multiplexed tissue imaging using the MIRIAM pipeline: machine-learning pixel classification to segment cellular compartments, per-marker quantification in nucleus/membrane/cytoplasm, autoencoder-derived shape latent vectors, and t-SNE embedding to relate shape to marker intensity.

## When to use

Use when analyzing multiplexed immunofluorescence or MIBI tissue images where cells have irregular/elongated morphologies (e.g., columnar epithelial cells) and per-cell marker intensity plus shape phenotyping is needed.

## When NOT to use

- Cells of interest have regular, round morphologies where simpler segmentation (nuclear dilation, Voronoi tessellation) suffices and the internal-membrane resegmentation step adds unnecessary complexity
- Input is already a per-cell feature/intensity table — skip segmentation and go directly to dimensionality reduction or correlation analysis
- The goal is purely intensity-based phenotyping (t-SNE on marker intensities, Figure 3A) rather than testing shape-marker correspondence

## Inputs

- Multiplexed immunofluorescence or MIBI image stacks of tissue (e.g., precancerous colorectal adenoma images, breast carcinoma MIBI dataset)
- MIRIAM pipeline outputs: per-cell median marker intensity CSVs and held-out autoencoder shape latent vectors
- Manually annotated TMA cores or sub-images if segmentation benchmarking (DSC/JSC) is desired

## Outputs

- Per-cell segmentation masks with nucleus, membrane, and cytoplasm compartments
- Per-cell marker intensity quantification table (median intensity per marker per compartment)
- Autoencoder shape latent vectors for held-out cells
- 2D t-SNE embedding colored by marker intensity (e.g., tsne_shape_latent_figure3c.png)
- Shape-marker Spearman correlation matrix with corrected p-values (shape_marker_correlations.csv)

## How to apply

Run MIRIAM (Matlab or Python implementation, with ilastik for pixel classification) on the multiplexed image stacks to segment cells and quantify each marker's median intensity over the entire cell and in the nucleus, membrane, and cytoplasm compartments. Characterize cell shapes by training the autoencoder on cell masks and encoding held-out cells into latent vectors, deliberately excluding latent vectors used to train the autoencoder from downstream analyses. Run t-SNE on the held-out shape latent vectors (Rtsne in R) and color the 2D embedding by per-cell marker intensities to test whether similar geometries cluster; overlay p-ERK intensity to check whether p-ERK-high cells form a discrete cluster as reported for Figure 3C. Separately compute Spearman rank correlations between each shape latent dimension and each marker's per-cell median intensity (pandas/scipy), applying multiple-testing correction across the latent-dimension x marker matrix.

## Related tools

- **MIRIAM** (Core pipeline: pixel classification, cell resegmentation of internal aberrant membranes, per-compartment marker quantification, and autoencoder shape characterization) — github.com/Coffey-Lab/MIRIAM
- **Rtsne** (R package used to produce the 2D t-SNE embedding of cells from shape latent vectors or marker intensities)
- **Python** (Implementation of the MIRIAM pipeline (interchangeable with Matlab) and used with pandas/scipy for Spearman correlations and multiple-testing correction) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Primary implementation for running cell segmentation on multiplexed imaging data per the repository README) — github.com/Coffey-Lab/MIRIAM
- **ilastik** (Required by the pipeline for some tasks (interactive machine-learning pixel classification)) — https://www.ilastik.org/download.html
- **Mesmer** (Deep learning segmentation baseline for benchmarking MIRIAM segmentation quality (DSC/JSC comparison))

## Examples

```
Rscript -e "library(Rtsne); tsne <- Rtsne(shape_latent_heldout, dims=2); plot(tsne$Y, col=pERK_intensity)"
```

## Evaluation signals

- Segmentation quality vs. manual annotations: MIRIAM image-level DSC ≈ 0.65 ± 0.08 and JSC ≈ 0.48 ± 0.08 on annotated TMA sub-images (240 sub-images across three cores); should outperform Mesmer (DSC 0.59 ± 0.10) and Voronoi (DSC 0.32 ± 0.11)
- t-SNE colored by marker intensities should place cells of similar geometry near each other when shape latent vectors are the input
- Shape-based t-SNE should be checked for the reported negative result: no discernible cellular populations or clear shape–marker patterns (Supplemental Figure 5), so absence of structure is the expected outcome
- Shape-marker correlation output (shape_marker_correlations.csv) must include both Spearman coefficients and multiple-testing-corrected p-values for every latent-dimension x marker pair
- Held-out latent vectors (not used for autoencoder training) must be the input to t-SNE, not the training-set vectors

## Limitations

- Shape-based t-SNE did not reveal discernible cellular populations or clear patterns in marker expression in the colonic adenoma dataset, so this skill may yield negative/null phenotyping results
- Voronoi-based segmentation undersegments cells towards the lumen as it does not take into account cell membranes; the comparison baselines have known failure modes on columnar epithelial cells
- Deep learning methods for nuclei detection were not implemented in MIRIAM and may be needed to improve initial seeds
- No software version numbers are stated for the Matlab or Python implementations; ilastik must be installed separately and image stacks are too large to ship with example data (must be regenerated or downloaded separately)

## Evidence

- [intro] MIRIAM begins with random forest-based machine learning pixel classification, then uses a novel method to identify and separate cells with internal membranes, quantifies markers per cell compartment (nucleus, membrane, cytoplasm), and characterizes cell shapes using an autoencoder neural network.: [evidence span withheld — anchor status: not_found]
