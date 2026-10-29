---
name: azimuth-reference-based-cell-annotation
description: Use when raw scRNA-seq count matrices (e.g., CD45+ immunotherapy cohorts)
  lack cell-type labels and downstream differential-state analysis such as scDist
  requires annotated cluster identities per cell.
when_to_use_negative:
- Input cells are already annotated with trusted cell-type labels and only the perturbation
  test remains (run scDist directly)
- The analysis goal is differential abundance (proportion changes between conditions)
  rather than differential transcriptomic state — tools like DA-seq, Milo, or Meld
  are the appropriate alternatives
- Only raw counts are available and no scTransform/normalized matrix will be produced
  — scDist cannot be applied directly to raw counts, so annotation alone is insufficient
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Azimuth
  role: 'Reference-based cell annotation: maps query scRNA-seq cells onto a reference
    atlas to transfer cell-type labels'
- name: Seurat
  role: Underlying framework executing the Azimuth reference mapping and integration
    workflow
- name: scTransform
  role: Normalization of counts via Pearson residuals prior to embedding/annotation
    and as recommended input for downstream scDist
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:e13f81a44786a0e366c61362783bd37cd649ca92e239af11b54c71b9cbe05880
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# azimuth-reference-based-cell-annotation

## Summary

Annotates cell populations in scRNA-seq data by mapping query cells onto Azimuth reference atlases via the Seurat-based reference mapping workflow, producing per-cell type labels. In the scDist immunotherapy study, these annotations define the cluster labels ('clusters') that scDist subsequently tests for transcriptomic perturbation between responders and non-responders.

## When to use

Use when raw scRNA-seq count matrices (e.g., CD45+ immunotherapy cohorts) lack cell-type labels and downstream differential-state analysis such as scDist requires annotated cluster identities per cell.

## When NOT to use

- Input cells are already annotated with trusted cell-type labels and only the perturbation test remains (run scDist directly)
- The analysis goal is differential abundance (proportion changes between conditions) rather than differential transcriptomic state — tools like DA-seq, Milo, or Meld are the appropriate alternatives
- Only raw counts are available and no scTransform/normalized matrix will be produced — scDist cannot be applied directly to raw counts, so annotation alone is insufficient

## Inputs

- Raw scRNA-seq expression count matrix (genes x cells) from CD45+ immunotherapy cohorts
- Per-cell metadata (patient/sample identifiers, responder status)
- Azimuth reference atlas (e.g., PBMC reference) for label transfer

## Outputs

- Per-cell cell-type annotations (cluster labels for every cell)
- Filtered cell-type set (types present in >10% of samples) suitable as scDist 'clusters' input

## How to apply

First normalize the raw expression counts with scTransform (Pearson residuals), the normalization the authors recommend and that scDist is calibrated for. Then run the Azimuth reference mapping workflow built on Seurat to project query cells onto a curated reference atlas and transfer cell-type labels (e.g., NK cells, dendritic cells, pDCs) to each query cell. Before passing annotations downstream to scDist, exclude any annotated cell type not present in more than 10% of the samples, as prescribed for scDist input preparation. Keep in mind that clustering/annotation errors bias scDist — condition-specific effects can cause over-clustering and downward bias in estimated distances — so annotation granularity matters: over-clustering drives the median scDist distance toward 0 and produces false negatives.

## Related tools

- **Azimuth** (Reference-based cell annotation: maps query scRNA-seq cells onto a reference atlas to transfer cell-type labels)
- **Seurat** (Underlying framework executing the Azimuth reference mapping and integration workflow)
- **scTransform** (Normalization of counts via Pearson residuals prior to embedding/annotation and as recommended input for downstream scDist)

## Evaluation signals

- Every cell in the query matrix receives exactly one annotation from the reference-mapped label set
- After filtering, no retained cell type is present in fewer than 10% of samples, matching the scDist input prescription
- Annotated populations in the immunotherapy cohorts reproduce expected immune compartments (dendritic cells, pDCs, NK cells including the FCER1G+ NK subpopulation)
- Cluster granularity is not excessive: over-clustering would manifest downstream as scDist median distances decaying toward 0 (false negatives)
- Downstream scDist results CSV contains one row per annotated/tested cell type with distance, 95% CI, and p-value columns, and ranks the NK (FCER1G+) compartment among the top perturbed populations

## Limitations

- Double dipping: using the same expression data for annotation and distance estimation can cause over-clustering and downward bias in scDist-estimated distances
- Significant clustering/annotation errors cause unavoidable bias in scDist; a cluster-free extension of scDist does not yet exist
- Results are sensitive to the number of principal components used downstream, and sub-optimal normalization other than Pearson residuals could affect performance

## Evidence

- [other] workflow step 2: [evidence span withheld — anchor status: not_found]
- [discussion] normalization prerequisite: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [discussion] post-annotation filter: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
- [discussion] annotation-error bias caveat: "condition-specific effects can result in over-clustering and downward bias in the estimated distances"
