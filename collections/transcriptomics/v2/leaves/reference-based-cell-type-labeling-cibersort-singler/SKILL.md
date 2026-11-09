---
name: reference-based-cell-type-labeling-cibersort-singler
description: Use when annotating held-out TME single cells (e.g. Yost et al. BCC data
  from TISCH) with reference-derived cell type labels, or when establishing a marker/reference-based
  baseline to compare against higher-resolution learned annotations.
when_to_use_negative:
- When the goal is fine-grained cell-state resolution rather than coarse cell type
  labels — the CIBERSORT baseline yields only 14 cell types versus 38 states from
  MetaTiME MeC annotation
- When input is not tumor microenvironment single-cell data consistent with the reference
  panels' scope
- When cells have already been projected into a learned latent space and the task
  is to interpret states there rather than re-derive marker-based labels
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CIBERSORT
  role: Provides cell type definition signatures used by Seurat's automated reference
    labeling to assign 14 cell types to BCC cells as a baseline annotation
- name: SingleR
  role: Reference-based automated cell type annotation against the Human Primary Cell
    Atlas and Blueprint/ENCODE panels for comparison with MetaTiME states
- name: MetaTiME
  role: Provides the metatime.loaddata normalization utilities and the MeC-based annotation
    against which the CIBERSORT/SingleR labels are cross-tabulated
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:2f3df0dc32ebfc770a6bfa4b0fed1e07481b7b2c2b116d622a063ef4e41650d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIBERSORT
  - MetaTiME
  - SingleR
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Reference-based cell type labeling with CIBERSORT and SingleR

## Summary

Baseline annotation strategy for tumor microenvironment scRNA-seq that assigns cell type labels by scoring cells against curated reference signatures (CIBERSORT-derived marker sets) or reference expression panels (SingleR with Human Primary Cell Atlas and Blueprint/ENCODE), used to benchmark the resolution of learned annotation methods such as MetaTiME's 86-meta-component (MeC) space.

## When to use

Use when annotating held-out TME single cells (e.g. Yost et al. BCC data from TISCH) with reference-derived cell type labels, or when establishing a marker/reference-based baseline to compare against higher-resolution learned annotations.

## When NOT to use

- When the goal is fine-grained cell-state resolution rather than coarse cell type labels — the CIBERSORT baseline yields only 14 cell types versus 38 states from MetaTiME MeC annotation
- When input is not tumor microenvironment single-cell data consistent with the reference panels' scope
- When cells have already been projected into a learned latent space and the task is to interpret states there rather than re-derive marker-based labels

## Inputs

- Normalized log1p library-size-normalized scRNA-seq count matrix (e.g. Yost et al. BCC cells downloaded from the TISCH portal)
- CIBERSORT-derived cell type signature gene sets used by Seurat's automated reference labeling
- SingleR reference expression panels: Human Primary Cell Atlas and Blueprint/ENCODE

## Outputs

- Per-cell CIBERSORT-based cell type labels (e.g. 14 cell types for BCC cells)
- Per-cell SingleR labels under both HPCA and Blueprint/ENCODE references
- Cross-tabulation table of MetaTiME MeC-based cell states vs. CIBERSORT- and SingleR-based labels, saved alongside yost_bcc_cell_annotations.csv

## How to apply

Annotate the identical normalized count matrix used for the learned method: run Seurat's automated reference labeling with CIBERSORT-derived signatures to assign coarse cell types (e.g. 14 cell types on Yost et al. BCC), and run SingleR using both the Human Primary Cell Atlas and Blueprint/ENCODE reference panels. Library-size normalization with log1p transform (via metatime.loaddata utilities) precedes labeling so all methods operate on the same input. Cross-tabulate the reference-based labels against MeC-based cell-state annotations (38 states) to quantify the resolution gap — CIBERSORT-style marker scoring collapses cells into broad types, while the 86 MeCs resolve lineage-specific states. Judge correctness by verifying every cell receives a label from each annotator and that the crosstab of labels vs. states is exported for comparison.

## Related tools

- **CIBERSORT** (Provides cell type definition signatures used by Seurat's automated reference labeling to assign 14 cell types to BCC cells as a baseline annotation)
- **SingleR** (Reference-based automated cell type annotation against the Human Primary Cell Atlas and Blueprint/ENCODE panels for comparison with MetaTiME states)
- **MetaTiME** (Provides the metatime.loaddata normalization utilities and the MeC-based annotation against which the CIBERSORT/SingleR labels are cross-tabulated) — github.com/yi-zhang/MetaTiME

## Examples

```
seu <- celltypist/seurat_automated_annotation; SingleR(sce, ref = list(HPCA, BlueprintEncode), labels = ref_labels)
```

## Evaluation signals

- Every cell in the held-out BCC dataset receives a label from each annotator (CIBERSORT-based, SingleR-HPCA, SingleR-Blueprint/ENCODE)
- CIBERSORT-based Seurat labeling recovers approximately 14 cell types on Yost et al. BCC cells, consistent with the published baseline
- Crosstab of reference-based labels vs. MetaTiME MeC states shows the expected resolution gap (14 types vs. 38 states), with fine states subdividing within broad types
- All methods were applied to the identically normalized count matrix (library-size normalization with log1p) so label differences reflect annotation method, not preprocessing
- Comparison table saved alongside yost_bcc_cell_annotations.csv with consistent cell identifiers across all annotation columns

## Limitations

- Cell type definitions in reference databases were based on representative genes chosen differently across studies, producing inconsistent prior definitions that limit resolution and consistency
- Marker/reference-based annotation cannot resolve lineage-specific cell states or signature continuums that the 86 data-driven MeCs capture
- Labels depend on reference panel choice — HPCA and Blueprint/ENCODE panels may yield different SingleR assignments on the same cells

## Evidence

- [results] CIBERSORT signatures as Seurat baseline: [evidence span withheld — anchor status: not_found]
- [discussion] Reference definitions differ across studies: "Cell type definitions in the previous studies were based on representative genes, which were chosen differently in the respective studies."
- [intro] Reference databases named: "cell type definitions in reference databases such as CIBERSORT, Azimuth, and Human Primary Cell Atlas"
- [readme] Pretrained MeCs for annotation: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
