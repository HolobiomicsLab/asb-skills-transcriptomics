---
name: prediction-contour-estimation
description: Use when initial per-cell cell-type predictions from Census's hierarchical
  models are noisy and need stabilization via UMAP-space contours and cluster label
  propagation before descending to the next hierarchy node.
when_to_use_negative:
- The Seurat object does not contain a UMAP dimensionality reduction — contours are
  computed in UMAP space, so the input must already include one
- The R session uses Seurat version 5 or another version — Census requires Seurat
  v4 to be loaded before the Census library
- Labels are already stabilized or the analysis goal is model training on a reference
  rather than annotation of a new dataset
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: 'Implements the label-stabilizing algorithm: MASS-based prediction contours,
    overlap detection, dispute adjudication, and iterative UMAP SNN-cluster label
    propagation'
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Constructs the shared nearest-neighbor graph and UMAP embedding on which contours
    and label propagation operate; must be loaded before the Census library
  repo: https://github.com/sjdlabgroup/Census
- name: MASS
  role: Used to compute prediction contours in UMAP space for each candidate label
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:861638459461f5f9c8f8d469edd63cf5b5c3b533cb4b6d07ab516f7d14c58afa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - MASS
  - Seurat 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# prediction-contour-estimation

## Summary

Census's label-stabilizing step: compute MASS-based prediction contours for each candidate cell-type label in UMAP space, detect overlapping contours, adjudicate disputed cells, and iterate with UMAP SNN-cluster label propagation until contours no longer overlap or labels stop changing.

## When to use

Use when initial per-cell cell-type predictions from Census's hierarchical models are noisy and need stabilization via UMAP-space contours and cluster label propagation before descending to the next hierarchy node.

## When NOT to use

- The Seurat object does not contain a UMAP dimensionality reduction — contours are computed in UMAP space, so the input must already include one
- The R session uses Seurat version 5 or another version — Census requires Seurat v4 to be loaded before the Census library
- Labels are already stabilized or the analysis goal is model training on a reference rather than annotation of a new dataset

## Inputs

- Seurat object containing a UMAP dimensionality reduction and root-node cell-type predictions from the pretrained Census model
- Pretrained Census model (Tabula Sapiens + Cancer Cell Line Encyclopedia) providing candidate labels at the current hierarchy node

## Outputs

- stabilized_annotations.csv with converged per-cell stabilized cell-type labels
- Resolved (disambiguated) label assignments per UMAP SNN cluster

## How to apply

Load the Seurat object (Seurat v4 must be loaded before the Census library) carrying root-node predictions from the pretrained Census model, then build the shared nearest-neighbor graph and UMAP embedding with Seurat. Compute MASS-based prediction contours in UMAP space for each candidate label, detect contour overlaps, and resolve disputed cells by adjudicating between the overlapping label contours. Re-propagate the most common (average) label across each UMAP SNN cluster to spread higher-confidence binary prediction outcomes, then recompute contours. Iterate the propagate–contour-dispute loop until convergence, defined as either no overlapping contours remaining or no further label changes between iterations. Write the converged per-cell annotations to stabilized_annotations.csv.

## Related tools

- **Census** (Implements the label-stabilizing algorithm: MASS-based prediction contours, overlap detection, dispute adjudication, and iterative UMAP SNN-cluster label propagation) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Constructs the shared nearest-neighbor graph and UMAP embedding on which contours and label propagation operate; must be loaded before the Census library) — https://github.com/sjdlabgroup/Census
- **MASS** (Used to compute prediction contours in UMAP space for each candidate label)

## Examples

```
library(Seurat); library(Census); res = census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Convergence invariant: iterating the propagate–dispute loop must terminate with either no overlapping contours or no further label changes between successive iterations
- After dispute resolution, each UMAP SNN cluster should carry a single dominant (most common) propagated label rather than mixed disputed labels
- Converged per-cell annotations are written to stabilized_annotations.csv with one label per cell
- Stabilized assignments should be spatially coherent in UMAP space (contours for each candidate label do not overlap)
- Session check: Seurat v4 is loaded before library(Census), otherwise the run fails

## Limitations

- Census requires Seurat version 4 to run; loading the Census library before Seurat v4 breaks execution
- The stabilization runs on clusters and contours in UMAP space, so it inherits the sensitivity of UMAP/SNN clustering to embedding parameters
- Convergence is iterative and may require multiple propagate–contour-dispute cycles before labels stabilize

## Evidence

- [intro] Label-stabilizing algorithm uses clusters and contours in UMAP space: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm"
- [readme] Contours and clusters propagate higher-confidence binary outcomes: "This involves using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes."
- [readme] Seurat v4 must be loaded before Census: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Input must be a Seurat object with a UMAP reduction: "The input data is a Seurat object that contains a UMAP dimensionality reduction."
