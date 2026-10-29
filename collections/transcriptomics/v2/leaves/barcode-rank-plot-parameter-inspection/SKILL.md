---
name: barcode-rank-plot-parameter-inspection
description: Use when running CellBender cell calling on a raw scRNA-seq/snRNA-seq
  count matrix and the required 'expected number of cells' / 'total-droplets-included'
  parameters must be chosen for a new dataset with no prior defaults.
when_to_use_negative:
- When a validated 'expected number of cells' value already exists for the dataset
  — re-inspection may introduce inconsistency across methods being compared
- When the tool in use derives its cell-calling cutoff automatically (e.g., EmptyDrops
  infers the cell-free profile with a Dirichlet-multinomial model, or Cell Ranger
  v.3 integrates EmptyDrops) rather than requiring a user-supplied expected cell count
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_6170
- http://edamontology.org/topic_3170
tools:
- name: CellBender
  role: Neural-network-based cell-calling tool whose 'expected number of cells' and
    'total-droplets-included' parameters are set by inspecting the barcode rank plot
    of the raw matrix
- name: EmptyNN
  role: Cell-calling method whose benchmark pipeline includes this manual parameter-inspection
    step for the CellBender comparison (e.g., on the adult mouse brain snRNA-seq dataset)
  repo: https://github.com/lkmklsmn/empty_nn
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:13bfb57d849dafff42dcf97cde5a78cf6cf3681c70d2d5a8dbcc047b31268f0c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lkmklsmn/empty_nn
  repo_url: https://github.com/lkmklsmn/empty_nn
  tools:
  - CellBender
  - EmptyDrops
  - EmptyNN
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_6170
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# barcode-rank-plot-parameter-inspection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Manual parameter-selection step for cell-calling tools such as CellBender, in which the barcode rank plot (total UMI counts sorted per barcode) of a raw droplet dataset is visually inspected to choose the 'expected number of cells' and 'total-droplets-included' parameters before running the tool.

## When to use

Use when running CellBender cell calling on a raw scRNA-seq/snRNA-seq count matrix and the required 'expected number of cells' / 'total-droplets-included' parameters must be chosen for a new dataset with no prior defaults.

## When NOT to use

- When a validated 'expected number of cells' value already exists for the dataset — re-inspection may introduce inconsistency across methods being compared
- When the tool in use derives its cell-calling cutoff automatically (e.g., EmptyDrops infers the cell-free profile with a Dirichlet-multinomial model, or Cell Ranger v.3 integrates EmptyDrops) rather than requiring a user-supplied expected cell count

## Inputs

- Raw unfiltered droplet count matrix (mtx or h5 format) from a droplet-based scRNA-seq or snRNA-seq dataset
- Barcode rank plot (total UMI counts per barcode, sorted descending)

## Outputs

- Chosen 'expected number of cells' parameter value for CellBender
- Chosen 'total-droplets-included' parameter value for CellBender
- Documented rationale (e.g., in a parameter log or workflow script comment) for the selected values

## How to apply

Generate or examine the barcode rank plot of the raw unfiltered count matrix (total UMI counts per barcode, sorted in decreasing order). The barcode rank plot's characteristic knee/plateau structure is used to identify where genuine cell-containing droplets end and the cell-free background begins. From this inspection, set CellBender's 'expected number of cells' parameter, which in turn guides the 'total-droplets-included' parameter; the EmptyNN paper notes that the expected number of cells depends on each dataset, so this inspection must be repeated per dataset (e.g., for the adult mouse brain snRNA-seq dataset where CellBender called 4,386 droplets). Record the chosen values so the downstream cell-calling comparison (EmptyNN, Cell Ranger v.2/v.3, EmptyDrops, CellBender) is reproducible.

## Related tools

- **CellBender** (Neural-network-based cell-calling tool whose 'expected number of cells' and 'total-droplets-included' parameters are set by inspecting the barcode rank plot of the raw matrix)
- **EmptyNN** (Cell-calling method whose benchmark pipeline includes this manual parameter-inspection step for the CellBender comparison (e.g., on the adult mouse brain snRNA-seq dataset)) — https://github.com/lkmklsmn/empty_nn

## Examples

```
cellbender remove-background --input raw_feature_bc_matrix.h5 --output cellbender_filtered.h5 --expected-cells 2222 --total-droplets-included 25000
```

## Evaluation signals

- The chosen 'expected number of cells' should correspond to the knee/plateau of the barcode rank plot rather than an arbitrary round number, and the rationale should be documented per dataset
- CellBender's resulting number of retained droplets should be comparable in order of magnitude to other methods on the same dataset (e.g., 4,386 vs. 2,181–2,566 for EmptyDrops/Cell Ranger on the mouse brain dataset; large deviations warrant re-checking the parameter choice)
- The parameter values used should be recorded so the cell-calling comparison table (e.g., cell_calling_mouse_brain_results.csv) can be exactly reproduced
- The barcode rank plot itself should be saved as part of the workflow artifacts for reproducibility

## Limitations

- The expected number of cells depends on each dataset — there is no universal default, and the choice is made manually by visual inspection rather than by an automated hyperparameter selection procedure
- Misjudging the knee of the barcode rank plot (e.g., when genuine low-count cells exist) can bias the resulting cell calls; the EmptyNN discussion notes that analogous threshold assumptions 'may not hold when cell-containing droplets with very low total counts exist'

## Evidence

- [methods] Run CellBender, choosing the expected number of cells / total-droplets-included parameter by inspecting the barcode rank plot for this dataset.: [evidence span withheld — anchor status: not_found]
