---
name: marker-intensity-correlation-analysis
description: Use when per-cell shape descriptors (autoencoder latent vectors) and
  per-cell marker quantifications from MIRIAM segmentation must be aligned cell-by-cell
  and tested for association, e.g. after t-SNE shows no obvious shape-marker patterns.
when_to_use_negative:
- Input is only raw image stacks with no MIRIAM segmentation/quantification output
  yet — run MIRIAM's segmentation, per-cell marker quantification, and autoencoder
  shape characterization first
- The goal is shape-based clustering or population discovery itself (t-SNE on latent
  vectors) rather than statistical testing of shape-marker association
- Marker and shape tables come from different segmentation runs and cannot be aligned
  cell-by-cell — correlations would be spurious
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MIRIAM
  role: Produces the upstream per-cell marker intensity quantifications and autoencoder
    shape latent vectors that this correlation analysis consumes
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Runs the Spearman correlation computation and multiple-testing correction
    (pandas/scipy)
- name: Matlab
  role: Alternative implementation environment for MIRIAM pipeline outputs feeding
    the analysis
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:e4f920d679e53d453dc53e121a22231d435e2d7a938c3e529f38c1b7d1beeb8b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# marker-intensity-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically tests whether cell shape (encoded as MIRIAM autoencoder latent vectors) corresponds to per-cell marker signal intensity in multiplexed imaging data, via Spearman rank correlation across every latent-dimension x marker pair with multiple-testing correction. Mirrors the MIRIAM paper's negative result for colonic adenoma t-SNE shape analysis.

## When to use

Use when per-cell shape descriptors (autoencoder latent vectors) and per-cell marker quantifications from MIRIAM segmentation must be aligned cell-by-cell and tested for association, e.g. after t-SNE shows no obvious shape-marker patterns.

## When NOT to use

- Input is only raw image stacks with no MIRIAM segmentation/quantification output yet — run MIRIAM's segmentation, per-cell marker quantification, and autoencoder shape characterization first
- The goal is shape-based clustering or population discovery itself (t-SNE on latent vectors) rather than statistical testing of shape-marker association
- Marker and shape tables come from different segmentation runs and cannot be aligned cell-by-cell — correlations would be spurious

## Inputs

- Shape latent vectors from the MIRIAM autoencoder (training vectors excluded)
- Per-cell median marker intensity CSVs from the MIRIAM quantification stage (multiplexed immunofluorescence images, e.g. colonic adenoma dataset)
- Optional t-SNE embedding of shape latent vectors for visual corroboration

## Outputs

- shape_marker_correlations.csv containing Spearman correlation coefficients and multiple-testing-corrected p-values for every latent-dimension x marker pair
- Summary interpretation of shape-marker association strength (positive, negative, or absent)

## How to apply

Load the saved shape latent vectors — excluding vectors used to train the MIRIAM autoencoder — and the per-cell median marker intensity CSVs produced by the MIRIAM quantification stage (nucleus, membrane, cytoplasm and whole-cell compartments). Align the two tables cell-by-cell on cell ID so every latent dimension pairs with every marker's per-cell median intensity. Compute Spearman rank correlations for each latent-dimension x marker pair using Python (pandas/scipy). Apply a multiple-testing correction across the full correlation matrix (e.g. Benjamini-Hochberg) and record both corrected p-values and correlation coefficients. Summarize magnitude and significance in shape_marker_correlations.csv; a null outcome (no discernible populations or patterns, as reported for the colonic adenoma data) is a valid, expected result of this analysis.

## Related tools

- **MIRIAM** (Produces the upstream per-cell marker intensity quantifications and autoencoder shape latent vectors that this correlation analysis consumes) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Runs the Spearman correlation computation and multiple-testing correction (pandas/scipy))
- **Matlab** (Alternative implementation environment for MIRIAM pipeline outputs feeding the analysis) — https://github.com/Coffey-Lab/MIRIAM

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(shape_latents, marker_medians, axis=0)  # shape_latents: autoencoder vectors (training excluded); marker_medians: MIRIAM per-cell median intensities
```

## Evaluation signals

- Cell-by-cell alignment succeeds: number of rows in latent-vector table matches marker intensity table after the join, with no dropped cells
- Correlation matrix dimensions equal (number of shape latent dimensions) x (number of markers), each entry with both a coefficient and a corrected p-value
- shape_marker_correlations.csv is produced with corrected (not raw) p-values alongside coefficients
- Exclusion check: autoencoder training vectors are not present in the analyzed latent set
- Conclusion is consistent with the t-SNE evidence — for the colonic adenoma data, weak/absent correlations (no discernible cellular populations or clear patterns in marker expression) is the paper-reported expected outcome

## Limitations

- The paper reports a negative finding: 'our t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression', so absence of association is not evidence of pipeline failure
- Only the colonic adenoma multiplexed immunofluorescence dataset is grounded here; behavior on other datasets (e.g. MIBI breast carcinoma) is not established by this analysis
- No code or data availability statement and no software version numbers are provided in the article discussion for reproducing the statistical analysis

## Evidence
