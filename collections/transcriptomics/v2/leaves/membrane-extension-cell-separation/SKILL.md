---
name: membrane-extension-cell-separation
description: Use when segmenting single cells from multiplexed imaging data (e.g.,
  multiplexed immunofluorescence of colorectal adenomas or MIBI breast carcinoma TMAs)
  where cells have internal or incomplete membranes that defeat dilation- or Voronoi-based
  segmentation.
when_to_use_negative:
- Input consists of single-cell suspension images or dissociated cells without tissue
  architecture and aberrant internal membranes — the membrane-extension logic targets
  in-tissue, membrane-incomplete cells.
- Input is already a per-cell feature/intensity table; this skill operates on raw
  multiplexed image stacks before quantification.
- Only a single nuclear channel is available with no membrane markers — pixel classification
  cannot label the membrane compartment that drives the extension step.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Provides the membrane-extension algorithm that identifies and separates cells
    with internal aberrant membranes; Matlab and Python implementations show largely
    similar results.
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: 'Random forest-based machine learning pixel classification that supplies the
    compartment probability input to the membrane-extension step (README: ''This pipeline
    requires Iltastik for some tasks'').'
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation language for running the MIRIAM membrane-extension pipeline
    on MIBI images (workflow step 2 of the transfer task).
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Alternative implementation of the same segmentation pipeline; results largely
    similar to the Python implementation (Supplemental Figure 6).
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:885920613d9ab5040c835c56c04378e00cd687df18a4de787c2509dccde26af5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - Ilastik
  - MIRIAM
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# membrane-extension-cell-separation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's novel membrane-extension method identifies and separates cells with internal aberrant membranes in multiplexed tissue images, extending incomplete cell membranes to produce per-cell segmentation masks with per-compartment (nucleus, membrane, cytoplasm) marker quantification.

## When to use

Use when segmenting single cells from multiplexed imaging data (e.g., multiplexed immunofluorescence of colorectal adenomas or MIBI breast carcinoma TMAs) where cells have internal or incomplete membranes that defeat dilation- or Voronoi-based segmentation.

## When NOT to use

- Input consists of single-cell suspension images or dissociated cells without tissue architecture and aberrant internal membranes — the membrane-extension logic targets in-tissue, membrane-incomplete cells.
- Input is already a per-cell feature/intensity table; this skill operates on raw multiplexed image stacks before quantification.
- Only a single nuclear channel is available with no membrane markers — pixel classification cannot label the membrane compartment that drives the extension step.

## Inputs

- Multi-marker multiplexed tissue image stacks (e.g., 2048x2048 pixel TMA images at 0.5 um/pixel from MIBI or multiplexed immunofluorescence)
- Random forest pixel-classification probability maps labeling nucleus, membrane, and cytoplasm compartments (produced via ilastik within the MIRIAM pipeline)

## Outputs

- Per-cell segmentation masks separating cells with internal aberrant membranes
- Per-marker intensity quantification over the entire cell and in nucleus, membrane, and cytoplasm compartments
- Image-level segmentation metrics (Dice Similarity Coefficient and Jaccard Similarity Coefficient) when evaluated against manual annotations

## How to apply

First run random forest-based machine learning pixel classification (e.g., via ilastik, which the MIRIAM pipeline requires) on the multi-marker image stack to label intracellular structures — nucleus, membrane, cytoplasm. Feed these pixel-classification probabilities into the membrane-extension step: the method detects cells containing internal aberrant membranes and extends incomplete membrane traces to close and separate adjacent cells, generating per-cell segmentation masks. Then quantify each marker's intensity over the entire cell and within the nucleus, membrane, and cytoplasm compartments. Evaluate segmentation quality with image-level Dice Similarity Coefficient (DSC) and Jaccard Similarity Coefficient (JSC) against manual annotations (the adenoma benchmark reported DSC 0.65 ± 0.08, JSC 0.48 ± 0.08, outperforming Mesmer's 0.59 ± 0.10 / 0.42 ± 0.08 and Voronoi's 0.32 ± 0.11 / 0.20 ± 0.07). Rationale: membrane-aware separation handles columnar epithelial cells with elongated, irregular morphologies that dilation-from-nuclei and Voronoi tessellation undersegment (Voronoi undersegments cells towards the lumen because it ignores cell membranes).

## Related tools

- **MIRIAM** (Provides the membrane-extension algorithm that identifies and separates cells with internal aberrant membranes; Matlab and Python implementations show largely similar results.) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random forest-based machine learning pixel classification that supplies the compartment probability input to the membrane-extension step (README: 'This pipeline requires Iltastik for some tasks').) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation language for running the MIRIAM membrane-extension pipeline on MIBI images (workflow step 2 of the transfer task).) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Alternative implementation of the same segmentation pipeline; results largely similar to the Python implementation (Supplemental Figure 6).) — github.com/Coffey-Lab/MIRIAM

## Examples

```
git clone https://github.com/Coffey-Lab/MIRIAM && python miriam_pipeline.py --input mibi_tma_stack --output cell_masks
```

## Evaluation signals

- Image-level DSC and JSC computed per image (e.g., written to miriam_mibi_segmentation_metrics.csv) should fall near the reported ranges (DSC 0.65 ± 0.08, JSC 0.48 ± 0.08 on the adenoma benchmark; representative MIBI results in Supplemental Figure 7).
- Segmentation masks should show high correspondence with Mesmer segmentation in epithelial cells while outperforming Mesmer (DSC 0.59 ± 0.10) and Voronoi (DSC 0.32 ± 0.11) baselines.
- Cells towards the lumen should not be undersegmented/merged, unlike Voronoi-based tessellation that ignores membranes.
- Quantified marker intensities must exist for each of the three compartments (nucleus, membrane, cytoplasm) per cell and marker.
- Matlab and Python implementations of the same data should produce largely similar segmentation results (Supplemental Figure 6).

## Limitations

- Benchmarked image-level performance is moderate (DSC 0.65 ± 0.08, JSC 0.48 ± 0.08), not near-perfect, even when outperforming alternatives.
- The MIBI breast carcinoma data set had lower image quality than the colorectal adenoma data set; performance may degrade with low-quality images.
- Deep learning methods for nuclei detection were not implemented; the authors note such additions 'may help improve initial seeds'.
- The README ships example data without the large image stacks; users must regenerate or download image data separately (Dropbox link in README).
- No software version numbers are stated for the Matlab or Python implementations.

## Evidence
