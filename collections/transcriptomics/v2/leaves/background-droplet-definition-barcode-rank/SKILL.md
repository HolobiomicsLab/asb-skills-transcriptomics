---
name: background-droplet-definition-barcode-rank
description: Use when running dsb (DSBNormalizeProtein) on CITE-seq ADT data with
  both raw and filtered Cell Ranger output available, and a set of empty droplets
  must be defined to estimate ambient antibody background.
when_to_use_negative:
- Only the filtered matrix is available and no raw-output barcodes exist to serve
  as empty droplets — dsb instead offers a workflow 'when empty droplets are not available'
  (separate vignette).
- Input data are already dsb-normalized or otherwise background-corrected ADT values;
  redefining background droplets is not applicable.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: dsb
  role: Consumes the defined cells and empty-droplet matrices via DSBNormalizeProtein
    to estimate per-antibody ambient background noise
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: Container for the raw/filtered matrices and downstream workflow into which
    dsb-normalized values are added
- name: R
  role: Language in which background barcodes are subset from the raw matrix after
    removing filtered barcodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:79e576554e387195965db9f00056074e17c9daea6329c4b0ac0cbd32d867b981
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# background-droplet-definition-barcode-rank

## Summary

Define the set of empty/background droplets used by dsb's ambient-noise estimation by taking barcodes from the raw Cell Ranger output that are not in the filtered output, using the EmptyDrops/Cell Ranger barcode rank to confirm they are genuine empty droplets (>50,000 in the 10X NextGem ~5K-cell 29-protein dataset). These background droplets provide the per-antibody ambient ADT mean mu_N and SD sigma_N for dsb step I.

## When to use

Use when running dsb (DSBNormalizeProtein) on CITE-seq ADT data with both raw and filtered Cell Ranger output available, and a set of empty droplets must be defined to estimate ambient antibody background.

## When NOT to use

- Only the filtered matrix is available and no raw-output barcodes exist to serve as empty droplets — dsb instead offers a workflow 'when empty droplets are not available' (separate vignette).
- Input data are already dsb-normalized or otherwise background-corrected ADT values; redefining background droplets is not applicable.

## Inputs

- Raw Cell Ranger output UMI matrices (Gene Expression and Antibody Capture)
- Filtered Cell Ranger output UMI matrices (Gene Expression and Antibody Capture)
- EmptyDrops/Cell Ranger barcode rank plot or barcode count statistics

## Outputs

- Cell protein count matrix (barcodes in filtered output)
- Background/empty-droplet ADT count matrix (remaining raw barcodes with protein reads, e.g. >50,000 droplets)
- Ambient background estimates mu_N and sigma_N per antibody consumed by DSBNormalizeProtein

## How to apply

Start from the raw and filtered Cell Ranger UMI matrices (Gene Expression and Antibody Capture); define cells as barcodes present in the filtered output and background drops as the remaining raw-output barcodes that carry ADT reads. Consult the EmptyDrops/Cell Ranger barcode rank to confirm the retained barcodes are truly empty — the 10X NextGem ~5K-cell external dataset yielded >50,000 empty droplets, often more than tenfold the number of cell-containing droplets. Pass the cell matrix and the background matrix to DSBNormalizeProtein, which uses the empty droplets to estimate the expected ambient background noise mu_N and sigma_N per antibody after the log(cell + 10) transformation. The paper warns that dsb normalized values are sensitive to how background droplets are defined, so a reasonable subset of barcodes as background requires care.

## Related tools

- **dsb** (Consumes the defined cells and empty-droplet matrices via DSBNormalizeProtein to estimate per-antibody ambient background noise) — https://cran.r-project.org/package=dsb
- **Seurat** (Container for the raw/filtered matrices and downstream workflow into which dsb-normalized values are added)
- **R** (Language in which background barcodes are subset from the raw matrix after removing filtered barcodes)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = dsb::cells_citeseq_mtx, empty_drop_matrix = dsb::empty_drop_citeseq_mtx)
```

## Evaluation signals

- Number of retained background droplets is large relative to cells (e.g. >50,000 empty droplets identified from the barcode rank for a ~5K-cell 29-protein dataset, often more than tenfold the number of cell-containing droplets).
- Background barcodes must have zero overlap with the filtered (cell) barcodes and carry ADT reads.
- Downstream dsb noise-model correlations (mu1 vs isotype controls; mu1 vs protein library size lambda) are positive and statistically significant across single cells.
- Resulting dsb-normalized ADT clusters/UMAP and biaxial plots (e.g. CD14 vs CD4) reproduce known population structure such as Fig. 2a-h.

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background still requires care.
- Assessment of empty-droplet-definition sensitivity is detailed in the Supplementary Note rather than the main text.
- Low-signal proteins (maximum raw UMI count <5 across all cells, e.g. CD34 in two datasets) should be removed prior to normalization.

## Evidence

- [discussion] Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [discussion] In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [readme] ADT reads from empty droplets—often more than tenfold the number of cell-containing droplets—closely match levels in: [evidence span withheld — anchor status: external_doc]
- [discussion] defining a reasonable subset of barcodes as background droplets still requires care: "defining a reasonable subset of barcodes as background droplets still requires care"
- [discussion] Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization"
