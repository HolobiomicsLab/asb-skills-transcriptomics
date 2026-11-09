---
name: scrnaseq-reference-based-deconvolution
description: Use when spatial transcriptomics data (e.g., Visium spots with multi-cell
  content, or Slide-seq singlet/doublet spots) must be assigned cell-type labels using
  a paired, cell-type-annotated scRNA-seq/snRNA-seq reference, and accuracy must be
  benchmarked per slice vs. jointly across slices.
when_to_use_negative:
- When the goal is only spot-level cell-type proportions without single-cell-resolution
  labels — simple deconvolution (Cell2location, CARD, spatialDWLS) suffices and the
  benchmark here targets single-cell label accuracy
- When no cell-type-annotated scRNA-seq/snRNA-seq reference exists for the tissue,
  since all compared methods are reference-based
- When the task is gene expression imputation rather than cell type labeling — gene
  imputation methods (Tangram, gimVI, SpaGE) were reported insufficiently accurate
  for label inference here
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpatialScope
  role: 'Primary method: Step 2 cell type identification with discrete RCTD-style
    probabilistic model and Potts prior (nu=10); only method whose error rate decreased
    under the all-slices setting'
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: RCTD (spacexr)
  role: 'Comparator: run in full doublet mode within StarDist+RCTD, with mean cell-type
    expression as the per-cell profile'
  repo: https://github.com/dmcable/spacexr
- name: Tangram
  role: 'Comparator: run per slice and across slices; cell-type labels recovered via
    project_genes mapping'
- name: CytoSPACE
  role: 'Comparator: its mapped single-cell IDs are taken as the assigned labels for
    each spot'
- name: StarDist
  role: Nucleus segmentation feeding the StarDist+RCTD comparator pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:db7b08d31c362800df686fd81ca25b8ac322ca69b66cad43072ac5e48bcb7fb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CytoSPACE
  - RCTD
  - RCTD (spacexr)
  - SpatialScope
  - StarDist
  - Tangram
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scrnaseq-reference-based-deconvolution

## Summary

Method for labeling spatial transcriptomics spots or single cells with cell types by leveraging a labeled scRNA-seq reference, comparing deconvolution/labeling approaches (RCTD, SpatialScope, Tangram, CytoSPACE, StarDist+RCTD) by single-cell-level error rate against ground-truth labels on simulated multi-slice Visium benchmarks.

## When to use

Use when spatial transcriptomics data (e.g., Visium spots with multi-cell content, or Slide-seq singlet/doublet spots) must be assigned cell-type labels using a paired, cell-type-annotated scRNA-seq/snRNA-seq reference, and accuracy must be benchmarked per slice vs. jointly across slices.

## When NOT to use

- When the goal is only spot-level cell-type proportions without single-cell-resolution labels — simple deconvolution (Cell2location, CARD, spatialDWLS) suffices and the benchmark here targets single-cell label accuracy
- When no cell-type-annotated scRNA-seq/snRNA-seq reference exists for the tissue, since all compared methods are reference-based
- When the task is gene expression imputation rather than cell type labeling — gene imputation methods (Tangram, gimVI, SpaGE) were reported insufficiently accurate for label inference here

## Inputs

- Multi-slice simulated Visium spot datasets with ground-truth single-cell labels (Benchmarking Datasets 5-6)
- Paired cell-type-annotated scRNA-seq/snRNA-seq reference (e.g., mouse brain cortical Smart-seq2 reference, 14,249 cells / 23 cell types)
- Histology images for StarDist nucleus segmentation (StarDist+RCTD pipeline)

## Outputs

- Per-spot or per-cell cell-type label assignments for each method and setting
- cell_type_identification_error_rates.csv — error-rate comparison table (single-slice vs all-slices)
- Cell-type proportion estimates per spot from deconvolution methods (e.g., RCTD full doublet mode)

## How to apply

Load the simulated multi-slice Visium benchmark datasets (Datasets 5-6) with ground-truth single-cell labels plus their paired scRNA-seq references from the SpatialScope demos directory. Run SpatialScope Step 2 (cell type identification) using its discrete RCTD-style probabilistic model with the Potts spatial smoothness prior at the default nu = 10, once per slice and once jointly on all slices; for Slide-seq-like inputs replace nucleus segmentation with singlet/doublet classification assuming at most two cell types per spot. Run comparators: StarDist+RCTD with RCTD in full doublet mode (using the mean gene expression of each identified cell type as the per-cell profile, after RCTD's platform-effect normalization), Tangram with label recovery via project_genes, and CytoSPACE taking its mapped single-cell IDs as per-spot labels. Score all methods by single-cell-level error rate under both the single-slice and all-slices settings and assemble the comparison table (cell_type_identification_error_rates.csv). Rationale: RCTD-style platform-effect normalization corrects systematic capture-rate differences between scRNA-seq and ST platforms, and SpatialScope's Potts prior exploits spatial coherence of cell types.

## Related tools

- **SpatialScope** (Primary method: Step 2 cell type identification with discrete RCTD-style probabilistic model and Potts prior (nu=10); only method whose error rate decreased under the all-slices setting) — https://github.com/YangLabHKUST/SpatialScope
- **RCTD (spacexr)** (Comparator: run in full doublet mode within StarDist+RCTD, with mean cell-type expression as the per-cell profile) — https://github.com/dmcable/spacexr
- **Tangram** (Comparator: run per slice and across slices; cell-type labels recovered via project_genes mapping)
- **CytoSPACE** (Comparator: its mapped single-cell IDs are taken as the assigned labels for each spot)
- **StarDist** (Nucleus segmentation feeding the StarDist+RCTD comparator pipeline)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml
```

## Evaluation signals

- Single-cell-level error rate against ground-truth labels: expect SpatialScope to reduce error by 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6 versus comparators
- All-slices-vs-single-slice direction: only SpatialScope should show a decreased error rate under the all-slices setting
- RCTD platform-effect step: confirm scRNA-seq cell-type profiles are normalized to the ST platform effects before scoring
- Output table schema check: cell_type_identification_error_rates.csv contains per-method, per-dataset, per-setting error rates for SpatialScope, Tangram, CytoSPACE, and StarDist+RCTD

## Limitations

- The Potts prior assumes neighboring cells belong to the same cell type, which may not always be valid
- Platform effects must be appropriately corrected between scRNA-seq and ST data or cell type identification results are unsatisfactory
- Training the SpatialScope deep generative model is time-consuming (hours or days)

## Evidence

- [methods] three steps of SpatialScope pipeline: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [intro] deconvolution method landscape: "There are now several cell-type deconvolution methods for ST data, including RCTD14, Cell2location15, CARD16 and spatialDWLS17."
- [readme] RCTD platform effect normalization: [evidence span withheld — anchor status: not_found]
- [readme] RCTD label and proportion output: [evidence span withheld — anchor status: external_doc]
- [methods] Slide-seq singlet/doublet mode: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [discussion] Potts prior caveat: "the assumption that neighboring cells belong to the same cell type may not always be valid"
