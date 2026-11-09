---
name: condition-grouped-single-cell-visualization
description: Use when a Seurat object carries per-cell signature scores or cell cycle
  labels and you must compare distributions or fractions across conditions (e.g.,
  Vehicle vs STM2457, WT vs Mettl3 KO) and save comparison plots.
when_to_use_negative:
- Input is a raw count matrix that has not yet been annotated — run accordion()/accordion_cellcycle()/accordion_custom()
  first before any condition-grouped visualization
- The analysis goal is cell type annotation itself rather than cross-condition comparison
  of scores, fractions, or marker impact plots
- No condition grouping exists in the metadata (single-sample dataset), so split-by-condition
  plots are meaningless
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cell Marker Accordion
  role: R package producing the per-cluster annotations, cell cycle phases, and per-cell
    custom signature scores stored in the Seurat object that are then visualized by
    condition; its dot plots report gene impact scores per cluster
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Holds annotations, cell cycle labels, and signature scores in the Seurat object
    metadata/misc slot and supplies plotting functions (DimPlot, FeaturePlot-style
    condition-split plots) for visualization
- name: R
  role: Language in which the annotation, scoring, comparison, and plotting workflow
    is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:7f39c5f8fef2e66053c99f9e4f037e09457d4b0647c89a7af1342c265f5cdfbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# condition-grouped-single-cell-visualization

## Summary

Visualize and compare per-cell annotation results, custom signature scores, and cell cycle states split by experimental condition in Seurat objects, as done in the Cell Marker Accordion Mettl3 KO/STM2457 bone marrow analysis (Fig. 8C,D,G,H,I).

## When to use

Use when a Seurat object carries per-cell signature scores or cell cycle labels and you must compare distributions or fractions across conditions (e.g., Vehicle vs STM2457, WT vs Mettl3 KO) and save comparison plots.

## When NOT to use

- Input is a raw count matrix that has not yet been annotated — run accordion()/accordion_cellcycle()/accordion_custom() first before any condition-grouped visualization
- The analysis goal is cell type annotation itself rather than cross-condition comparison of scores, fractions, or marker impact plots
- No condition grouping exists in the metadata (single-sample dataset), so split-by-condition plots are meaningless

## Inputs

- Processed Seurat objects with accordion() cell type annotations (accordion_per_cluster), accordion_cellcycle() phase labels (G0, G1, S, G2M), and accordion_custom() innate immune response per-cell scores
- Condition metadata column (e.g., 'condition': Vehicle vs STM2457, WT vs Mettl3 KO)
- Top 5 marker genes per condition and cell type extracted from the Seurat misc slot

## Outputs

- Condition-split feature plots of per-cell innate immune response scores
- Dot plots of gene impact scores showing top markers driving condition- and cell-type-specific differences
- Cell cycle fraction comparisons (G0, G1, S, G2M) between conditions
- Saved result files containing cell type annotations, cell cycle assignments, per-cell scores, and comparison plots

## How to apply

After running accordion(), accordion_cellcycle(), and accordion_custom() on the Mettl3 KO/WT and STM2457/vehicle bone marrow Seurat objects, extract per-cell innate immune response scores and the top 5 marker genes per condition and cell type from the Seurat misc slot. Build condition-split feature plots of per-cell scores and dot plots of gene impact scores to show which markers drove condition-specific differences. Compare score distributions between Vehicle vs STM2457 and WT vs Mettl3 KO, and compare cell cycle fractions (G0, G1, S, G2M) across conditions. Acceptance follows the reported Fig. 8 patterns: a statistically significant innate immune response score increase in both KO and STM2457 conditions, a higher G0 fraction upon Mettl3 modulation, and the expected lineage shifts (increased stem cell and megakaryocyte fractions, decreased erythroid fractions). Save the annotations, cell cycle assignments, scores, and plots as result files.

## Related tools

- **Cell Marker Accordion** (R package producing the per-cluster annotations, cell cycle phases, and per-cell custom signature scores stored in the Seurat object that are then visualized by condition; its dot plots report gene impact scores per cluster) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Holds annotations, cell cycle labels, and signature scores in the Seurat object metadata/misc slot and supplies plotting functions (DimPlot, FeaturePlot-style condition-split plots) for visualization)
- **R** (Language in which the annotation, scoring, comparison, and plotting workflow is executed)

## Examples

```
DimPlot(data, group.by = "accordion_per_cluster")
```

## Evaluation signals

- Per-cell innate immune response score distributions show a statistically significant increase in both Mettl3 KO and STM2457 conditions relative to their controls, matching Fig. 8C,D,G,H,I
- G0 fraction is higher upon Mettl3 modulation (KO or STM2457) compared to the control condition
- Lineage fractions shift as reported: increased stem cell and megakaryocyte fractions with decreased erythroid fractions
- Saved plots include both dot plots of gene impact scores and condition-split feature plots, with the top 5 marker genes per condition and cell type extracted from the Seurat misc slot
- Result files reproducibly contain cell type annotations (accordion_per_cluster), cell cycle assignments, and per-cell scores for both datasets

## Limitations

- The reproduction requires acceptance criteria tied to the specific Fig. 8 patterns; datasets or tissues other than murine bone marrow Mettl3 perturbations need re-derived expectations
- The Cell Marker Accordion benchmark is context-limited ('our current benchmark, focused on two liquid and two solid tumors, demonstrates improved performance within these specific contexts'), so generalization of annotations feeding the plots should be checked per tissue
- Correct cell type labels depend on consistent marker databases; heterogeneous marker sources can lead to inconsistent annotations that would propagate into condition-split visualizations

## Evidence

- [readme] The dot size is proportional to the impact score, and the winning annotation is highlighted.: "The dot size is proportional to the impact score, and the winning annotation is highlighted."
- [discussion] The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types, consistent with the formation of aberrant endogenous dsRNAs: "The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types"
- [methods] Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
