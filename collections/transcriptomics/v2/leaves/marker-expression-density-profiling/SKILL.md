---
name: marker-expression-density-profiling
description: 'Use when validating a completed integration of CyTOF or similar single-cell
  datasets: you need to confirm dataset effects are removed by checking that per-marker
  density curves overlap across datasets while cell-type-specific marker expression
  is preserved.'
when_to_use_negative:
- Integration has not yet been performed — density profiling of raw, unadjusted CyTOF
  matrices will show residual dataset effects and cannot validate the integration
- Datasets lack shared markers or harmonized cell-type annotations, so density curves
  cannot be meaningfully overlaid per marker or per cell type
- The goal is quantitative batch-effect metrics (e.g., kBET, LISI) rather than marker-level
  distributional inspection — use a dedicated metric workflow instead
tools:
- name: scMerge2
  role: Produces the dataset-effect-adjusted single-cell expression matrix whose per-marker
    densities are profiled for validation
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Used to compute the joint UMAP embedding on the scMerge2-adjusted matrix that
    accompanies the density plots in the validation figure
- name: CyTOF
  role: Cytometry by Time-Of-Flight data modality whose marker-expression measurements
    are the input being validated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:cb6f7703c2ab3afdf796a77ec452af3be47e5ec5b9325b7cd1b453e09f926cad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - CyTOF
  - Seurat
  - scMerge2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# marker-expression-density-profiling

## Summary

Overlay per-marker expression density distributions across datasets after single-cell integration (e.g., CyTOF integration via scMerge2) to visually and quantitatively confirm that dataset-level unwanted variation has been removed while biological structure is retained.

## When to use

Use when validating a completed integration of CyTOF or similar single-cell datasets: you need to confirm dataset effects are removed by checking that per-marker density curves overlap across datasets while cell-type-specific marker expression is preserved.

## When NOT to use

- Integration has not yet been performed — density profiling of raw, unadjusted CyTOF matrices will show residual dataset effects and cannot validate the integration
- Datasets lack shared markers or harmonized cell-type annotations, so density curves cannot be meaningfully overlaid per marker or per cell type
- The goal is quantitative batch-effect metrics (e.g., kBET, LISI) rather than marker-level distributional inspection — use a dedicated metric workflow instead

## Inputs

- scMerge2-adjusted single-cell CyTOF marker-expression matrix
- Cell-level metadata with dataset label, sample, condition, and unified cell-type annotations (18 unified cell types)
- Joint UMAP embedding computed on the adjusted expression matrix (Seurat)

## Outputs

- Per-marker overlaid density plots comparing expression distributions across datasets (pre- vs post-integration)
- Composite validation figure of integrated UMAP (colored by dataset and cell type) plus per-marker density curves, compared qualitatively against FigS7_v1.png

## How to apply

After running scMerge2 on the marker-expression matrices (e.g., the COMBAT and Geanon CyTOF datasets totaling more than 11 million cells from healthy controls, COVID-19 and sepsis patients), propagate adjustment factors back to the single-cell level and compute a joint UMAP (Seurat) on the adjusted matrix. Then, for each marker, render expression density distributions per dataset on the scMerge2-adjusted values and overlay the curves across datasets. Correct integration is indicated by overlapping per-marker density curves across datasets, consistent with the UMAP showing datasets mixed within shared cell types. Rationale: cell-type-specific marker expression distributions provide a direct check that dataset-effect removal did not distort biological signal, complementing embedding-based inspection. In the task this supports, a clearly separable Granulocyte (Neutrophil/Eosinophil) cluster unique to Geanon must remain distinct even as densities align.

## Related tools

- **scMerge2** (Produces the dataset-effect-adjusted single-cell expression matrix whose per-marker densities are profiled for validation) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Used to compute the joint UMAP embedding on the scMerge2-adjusted matrix that accompanies the density plots in the validation figure)
- **CyTOF** (Cytometry by Time-Of-Flight data modality whose marker-expression measurements are the input being validated)

## Examples

```
BiocManager::install("scMerge")
```

## Evaluation signals

- Per-marker density curves overlap across datasets after integration (dataset effects removed), matching FigS7_v1.png qualitatively
- In the joint UMAP, datasets are mixed within shared cell types rather than separating by dataset
- A clearly separable Granulocyte (Neutrophil/Eosinophil) cluster unique to the Geanon dataset is retained after adjustment
- Cell-type-specific marker expression distributions remain distinguishable across the 18 unified cell types, confirming biological signal is not erased
- Same density plots computed on raw (pre-integration) data show visible dataset separation, confirming the comparison detects dataset effects

## Limitations

- Density alignment is a qualitative visual check; it does not replace quantitative batch-removal metrics
- scMerge2 integration described here was performed within each modality only — joint use of multi-modal information is deferred to future work, so density profiling cannot validate cross-modal integration
- Dataset-unique populations (e.g., the Geanon Granulocyte cluster) will legitimately show non-overlapping densities for lineage markers; the check must be interpreted per cell type, not globally

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [intro] scMerge2 scales to millions of cells across single-cell technologies: "Our scMerge2 algorithm is able to integrate many millions of cells from single-cell studies generated from various single-cell technologies, including scRNA-seq, CyTOF"
- [readme] scMerge2 tooling and vignettes are available via the scMerge package: [evidence span withheld — anchor status: external_doc]
