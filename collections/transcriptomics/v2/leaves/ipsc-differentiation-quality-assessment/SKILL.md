---
name: ipsc-differentiation-quality-assessment
description: Use when bulk RNA-seq profiles of iPSC-derived cells (e.g., hiPSC-NPC
  n=13, hiPSC-neuron n=11) need cell type assignment to judge whether differentiation
  toward a target lineage succeeded or stalled.
when_to_use_negative:
- Query cells are malignant/cancer cells, since deCS 'might be challenging for cancer
  cell annotations due to the expression profile difference between healthy and malignant
  cells'
- The goal is to discover novel cell types, as deCS is a mapping-based toolkit that
  'fails to characterize novel cell types'
- Input is raw single-cell data without prior normalization, scaling, and clustering
  to produce a markers_expression matrix
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Performs correlation-based cell type annotation of the normalized bulk RNA-seq
    profiles against pre-calculated cell type specificity score reference panels
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:74f93980f1561e83f0ff397e6b3bdbffb954a698d998e46ce88334e52ea872ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# iPSC-differentiation-quality-assessment

## Summary

Assesses the differentiation quality of iPSC-derived neural progenitor cells (NPCs) and neurons by annotating normalized bulk RNA-seq sample profiles against human cell type reference panels using deCS correlation analysis, verifying that NPCs map to fetal progenitor/hESC-like types and neurons to neuronal types.

## When to use

Use when bulk RNA-seq profiles of iPSC-derived cells (e.g., hiPSC-NPC n=13, hiPSC-neuron n=11) need cell type assignment to judge whether differentiation toward a target lineage succeeded or stalled.

## When NOT to use

- Query cells are malignant/cancer cells, since deCS 'might be challenging for cancer cell annotations due to the expression profile difference between healthy and malignant cells'
- The goal is to discover novel cell types, as deCS is a mapping-based toolkit that 'fails to characterize novel cell types'
- Input is raw single-cell data without prior normalization, scaling, and clustering to produce a markers_expression matrix

## Inputs

- Bulk RNA-seq count matrices for hiPSC-NPC (n=13) and hiPSC-neuron (n=11) samples with human gene symbols
- deCS pre-calculated cell type specificity score reference panels (e.g., MonacoImmune_main, BlueprintEncode_main, Human_cell_atlas_of_fetal)

## Outputs

- Tab-delimited annotation table (hipsc_decs_result.txt) with columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels for all 24 samples

## How to apply

Load bulk RNA-seq count matrices for hiPSC-NPC and hiPSC-neuron samples (human gene symbols required) and preprocess with the deCS example scripts, log-transforming and z-scaling to obtain a markers_expression matrix. Load pre-calculated reference panels from the installed deCS package via data(), e.g. data(MonacoImmune_main), data(BlueprintEncode_main), and data(Human_cell_atlas_of_fetal) for fetal references. Run deCS.correlation(markers_expression, ref_panel), which computes Pearson correlation coefficients between each query profile and each reference cell type, optionally setting cor_threshold to flag low-confidence calls as 'Undetermined cells'. Write the annotation table (Query, Max_correlation, Top1, Top2, Top3, Cell_labels) with write.table(..., sep = '\t', quote = F), then inspect Cell_labels: correct differentiation yields NPC samples enriched in fetal progenitor cells or human embryonic stem cells (hESCs) and neuron samples in neuronal cell types.

## Related tools

- **deCS** (Performs correlation-based cell type annotation of the normalized bulk RNA-seq profiles against pre-calculated cell type specificity score reference panels) — https://github.com/bsml320/deCS

## Examples

```
hipsc_decs_result <- deCS.correlation(markers_expression, Human_cell_atlas_of_fetal); write.table(hipsc_decs_result, file = "hipsc_decs_result.txt", sep = "\t", quote = F)
```

## Evaluation signals

- Output table contains exactly 24 rows (13 hiPSC-NPC + 11 hiPSC-neuron samples) with the columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels
- NPC samples are assigned Cell_labels enriched in fetal progenitor cells or human embryonic stem cells (hESCs) in 12 of 13 samples
- Neuron samples are assigned to neuronal cell types in the Cell_labels column
- Max_correlation values are Pearson correlation coefficients with plausible ranges; samples below cor_threshold should be labeled 'Undetermined cells'
- Annotation is reproduced against multiple reference panels (e.g., MonacoImmune_main, BlueprintEncode_main, Human_cell_atlas_of_fetal) for consistency

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias the annotation results
- deCS fails to characterize novel cell types as a mapping-based annotation toolkit
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [methods] correlation analysis against reference panels: "we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell type in the reference dataset, then the most relevant cell type(s) will be"
- [readme] reference panels loading: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal et al."
- [methods] filter threshold: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [results] finding: [evidence span withheld — anchor status: not_found]
- [readme] correlation analysis is for expression profiles: "If the query is gene expression profile, we provide function `deCS.correlation()` for cell type enrichment analysis."
