---
name: immune-cell-type-hierarchy-mapping
description: Use when cell-type interpretability scores from Pollock explain mode
  (captum feature scores) need validation against a curated marker database such as
  PanglaoDB, and cell states must be mapped to marker sets through a hierarchical
  parent-state fallback.
when_to_use_negative:
- No explain-mode captum feature scores or DWG lists exist yet — this skill validates
  DWGs and does not generate them; run Pollock explain mode first
- Cell states of interest cannot be matched even to a next-most-macro parent cell
  type in the PanglaoDB hierarchy, so no marker set can be assigned
- The goal is differential expression analysis on the raw counts themselves (e.g.
  sc.pp.filter/normalize workflow) rather than validating model feature scores against
  a marker database
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Pollock
  role: Pretrained cell-state classification module whose explain-mode output supplies
    the top-100 DWGs per immune cell state for overlap testing
  repo: https://github.com/ding-lab/pollock
- name: captum
  role: Feature-attribution backend behind Pollock explain mode that produces the
    interpretability (feature) scores used to rank DWGs
  repo: https://github.com/ding-lab/pollock
- name: scanpy
  role: Part of the Pollock environment used to load and handle the .h5ad single-cell
    objects during explain-mode runs
  repo: https://github.com/ding-lab/pollock
- name: PanglaoDB
  role: Curated marker gene reference (release 27_Mar_2020, human-supported entries)
    providing the marker sets matched to each cell state
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:99c3ab0bb7851c659ea17afb1f523f7219c87cace76e8192eca8a627561af741
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - PanglaoDB
  - Pollock
  - captum
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# immune-cell-type-hierarchy-mapping

## Summary

Validates Pollock's top-100 differentially weighted genes (DWGs) per immune cell state against curated PanglaoDB marker sets, using a cell-state-to-PanglaoDB-cell-type mapping with a parent-state fallback. Quantifies marker overlap per state to identify which learned states (e.g. B, pDC, Treg) best recover known biology.

## When to use

Use when cell-type interpretability scores from Pollock explain mode (captum feature scores) need validation against a curated marker database such as PanglaoDB, and cell states must be mapped to marker sets through a hierarchical parent-state fallback.

## When NOT to use

- No explain-mode captum feature scores or DWG lists exist yet — this skill validates DWGs and does not generate them; run Pollock explain mode first
- Cell states of interest cannot be matched even to a next-most-macro parent cell type in the PanglaoDB hierarchy, so no marker set can be assigned
- The goal is differential expression analysis on the raw counts themselves (e.g. sc.pp.filter/normalize workflow) rather than validating model feature scores against a marker database

## Inputs

- Top-100 DWG lists per immune cell state, derived from Pollock explain-mode captum feature scores on the pretrained module
- PanglaoDB marker gene files, release 27_Mar_2020, filtered to entries with human support

## Outputs

- panglao_overlap_table.csv with per-state overlap percentages (intersection size divided by 100)
- Ranking of cell states by marker overlap (B, pDC and Treg show the most overlap at 25–29%)

## How to apply

Load the top-100 DWG lists per immune cell state, derived from Pollock explain-mode captum feature scores on the pretrained module (e.g. the scRNAseq_brca_panimmune module, which was trained with a specific immune cell state breast cancer dataset). Download the PanglaoDB marker gene files (release 27_Mar_2020, filtered to entries with human support). Map each cell state to its corresponding PanglaoDB cell type; when no direct match exists, assign the marker set of the next-most-macro parent cell state. Compute the overlap percentage for each state as the size of the intersection between its top-100 DWGs and the matched PanglaoDB marker set divided by 100, then assemble the per-state percentages into panglao_overlap_table.csv. Correctness is judged by B, pDC and Treg states reproducing the reported 25–29% overlap range.

## Related tools

- **Pollock** (Pretrained cell-state classification module whose explain-mode output supplies the top-100 DWGs per immune cell state for overlap testing) — https://github.com/ding-lab/pollock
- **captum** (Feature-attribution backend behind Pollock explain mode that produces the interpretability (feature) scores used to rank DWGs) — https://github.com/ding-lab/pollock
- **scanpy** (Part of the Pollock environment used to load and handle the .h5ad single-cell objects during explain-mode runs) — https://github.com/ding-lab/pollock
- **PanglaoDB** (Curated marker gene reference (release 27_Mar_2020, human-supported entries) providing the marker sets matched to each cell state)

## Examples

```
pollock explain from_scanpy --module-filepath <path_to_module_directory> --scanpy-h5ad-filepath <filepath_to_h5ad_object> --background-filepath <filepath_to_background_h5ad> --background-sample-size 100 --output-prefix explain
```

## Evaluation signals

- B, pDC and Treg cell states reproduce the reported 25–29% overlap range with their matched PanglaoDB marker sets
- Each overlap percentage equals the intersection size between the state's top-100 DWGs and its matched PanglaoDB marker set divided by 100, so values are integers in [0, 100]%
- Every cell state with no direct PanglaoDB match was assigned the marker set of the next-most-macro parent cell state before computing overlap
- Marker sets come from PanglaoDB release 27_Mar_2020 filtered to entries with human support
- Output table panglao_overlap_table.csv contains one row per immune cell state with its computed overlap percentage

## Limitations

- Cell states without a direct PanglaoDB cell-type match must borrow the marker set of the next-most-macro parent cell state, which coarsens the overlap estimate
- Explain mode can have excessive runtimes for very large numbers of cells; ~100 cells with a background sample size of ~100 takes about 15 minutes, so downsampling is recommended
- DWG lists are limited to genes present in the model — during prediction on the pbmc3k example only 12679 genes overlapped with the model after filtering and 4859 genes were missing from the dataset
- The pretrained pollock model can only predict 17 cell type classes (e.g. 'B cell', 'CD4 T cell', 'Treg'), so unmapped states must rely on the hierarchy fallback

## Evidence

- [abstract] Interpretability-score provenance for DWGs: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [methods] Explain mode is the Pollock mode behind feature-score generation: "An example of explaining a model for a Seurat .RDS object that has cell type labels in a metadata column named 'cell_type'"
- [methods] Pretrained module source for the immune cell state model: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] Background sampling parameter for explain runs: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
- [readme] Runtime caveat motivating downsampling before deriving DWG lists: "Note: explain mode can have excessive runtimes for very large numbers of cells, so we recommend downsampling"
