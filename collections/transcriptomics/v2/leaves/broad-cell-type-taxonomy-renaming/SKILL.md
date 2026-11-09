---
name: broad-cell-type-taxonomy-renaming
description: Use when training a reference-based identifier on a pre-annotated dataset
  and applying it to a test dataset whose cell populations include types absent from
  the reference taxonomy, requiring label harmonization before evaluation of unknown-population
  rejection.
when_to_use_negative:
- Reference and test datasets already share an identical, standardized cell-type taxonomy
  — renaming would destroy informative subtype resolution
- The goal is fine-grained subtype identification within the same tissue (e.g., distinguishing
  T cell subtypes), since collapsing to broad types removes those distinctions
- Marker-based annotation with a curated marker DB (e.g., HiCAT's 3-level taxonomy
  of tissue/cell_type_major/cell_type_minor/cell_type_subset), where the taxonomy
  is supplied by the marker file rather than by dataset annotations
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Reference-based identifier (MarkerCount_Ref) trained on the harmonized/broad-renamed
    reference taxonomy to reject out-of-taxonomy tumor cells
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Loading and manipulating .h5ad scRNA-seq datasets and renaming their annotation
    labels
- name: pandas
  role: Renaming annotation Series and writing the per-dataset metric table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:35843fece5d8923615c06575b52c19dcd412a83aaafd9032445ca6a78615a8df
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  repo_url: https://github.com/combio-dku/MarkerCount
  tools:
  - MarkerCount
  - SCANPY
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# broad-cell-type-taxonomy-renaming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes manual cell-type annotations between reference and test single-cell RNA-seq datasets by collapsing fine-grained subtypes into broad cell types (e.g., T cell subtypes mapped to 'T cell'), so that out-of-taxonomy populations (e.g., tumor cells tested against a blood reference) fall outside the reference taxonomy and can be correctly rejected as unknown (CUA) by MarkerCount_Ref.

## When to use

Use when training a reference-based identifier on a pre-annotated dataset and applying it to a test dataset whose cell populations include types absent from the reference taxonomy, requiring label harmonization before evaluation of unknown-population rejection.

## When NOT to use

- Reference and test datasets already share an identical, standardized cell-type taxonomy — renaming would destroy informative subtype resolution
- The goal is fine-grained subtype identification within the same tissue (e.g., distinguishing T cell subtypes), since collapsing to broad types removes those distinctions
- Marker-based annotation with a curated marker DB (e.g., HiCAT's 3-level taxonomy of tissue/cell_type_major/cell_type_minor/cell_type_subset), where the taxonomy is supplied by the marker file rather than by dataset annotations

## Inputs

- Pre-annotated reference scRNA-seq dataset (e.g., GSE100866 CBMC 8K or GSE93421 PBMC 68K) with fine-grained manual annotations
- Tumor test scRNA-seq datasets with manual annotations (GSE72056 Melanoma 5K, GSE103322 HeadNeck 6K)
- Mapping of fine-grained subtype labels to broad cell-type labels (e.g., T cell subtypes -> T cell)

## Outputs

- Renamed annotation columns in both reference and test datasets using a shared broad taxonomy (as .h5ad/adata obs or pandas Series)
- Per-dataset cross-evaluation metric table (tumor_cross_evaluation_metrics.csv) with C, E, EA, CUA, EUA fields between 0 and 1

## How to apply

Before training, rename the manual annotations of both the reference (e.g., GSE100866 CBMC 8K or GSE93421 PBMC 68K) and the test datasets (e.g., GSE72056 Melanoma 5K, GSE103322 HeadNeck 6K) to a shared broad taxonomy — for instance, map all T cell subtypes to 'T cell' — so tumor cells sit outside the reference taxonomy. This is essential because the rejection evaluation treats tumor cells as correctly unassigned (CUA) only if the reference never offered a matching label. Train MarkerCount_Ref on the harmonized blood reference (binarize expression with bij=1 if xij>0, marker scores with n=2, fth=0.9, top N=18 markers per type), then compute FPR-based rejection thresholds tm and evaluate the tumor predictions with the C, E, EA, CUA, EUA measures. Rationale: cell type taxonomies are not standardized, so the curator must deliberately construct the cross-dataset label mapping rather than assume dataset-native labels are comparable.

## Related tools

- **MarkerCount** (Reference-based identifier (MarkerCount_Ref) trained on the harmonized/broad-renamed reference taxonomy to reject out-of-taxonomy tumor cells) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Loading and manipulating .h5ad scRNA-seq datasets and renaming their annotation labels)
- **pandas** (Renaming annotation Series and writing the per-dataset metric table)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('GSE72056_Melanoma_5K.h5ad'); adata.obs['cell_type'] = adata.obs['cell_type'].replace({'CD4 T cell':'T cell','CD8 T cell':'T cell','Treg':'T cell'})
```

## Evaluation signals

- After renaming, the reference label set contains only broad types (e.g., 'T cell' rather than CD4/CD8/Treg subtypes) and no reference label matches the tumor-specific labels
- The output tumor_cross_evaluation_metrics.csv exists and contains per-dataset C, E, EA, CUA, EUA fields with values between 0 and 1
- CUA (correctly unassigned) is high for tumor cells while EA (erroneously assigned) is low, matching the deposited figshare reference results
- The renamed reference preserves the reference cell-type count expected by the pipeline (e.g., 15 (11) types for CBMC 8K, 11 (6) for PBMC 68K)

## Limitations

- Cell types and taxonomy are not standardized, so researchers must prepare their own label list/mapping for the specific study and objective
- Collapsing subtypes to broad types is appropriate for unknown-population rejection but cannot be reused for subtype-level identification tasks
- Proper downstream identification requires well-clustered cell populations, which is not always the case

## Evidence

- [other] renaming annotations to broad cell types (e.g., T cell subtypes mapped to T cell) so the tumor cells fall outside the reference taxonomy: [evidence span withheld — anchor status: not_found]
- [readme] MarkerCount works in reference mode requiring pre-annotated training data: "MarkerCount works both in reference and marker-based mode, where the latter utilizes only the existing lists of markers, while the former required pre-annotated dataset to train the model."
- [discussion] taxonomy is not standardized: "Cell types and taxonomy are not standardized and debating so that researcher should prepare their own list of markers for the specific study and objective."
