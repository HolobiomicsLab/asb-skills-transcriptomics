---
name: upset-plot-method-comparison-visualization
description: Use when comparing which genes multiple DS analysis methods (pseudobulk
  pbDS engines vs. cell-level mmDS mixed models) call significant at FDR < 0.05, and
  you need to visualize set intersections across methods and replicates.
when_to_use_negative:
- The analysis goal is per-method performance metrics (TPR, FDR, AUC) rather than
  between-method agreement of significant calls
- Results come from a single method or single replicate, so there are no cross-method
  or cross-replicate gene sets to intersect
- Input is raw counts or pseudobulk aggregates rather than completed DS result tables
  with adjusted p-values
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: R package providing the pbDS and mmDS DS analysis engines whose top-ranked
    detections are intersected and visualized; installed via BiocManager::install('muscat')
  repo: https://github.com/HelenaLC/muscat
- name: pbDS
  role: pseudobulk aggregation-based DS engine (edgeR, DESeq2, limma-voom methods)
    whose significant calls form one class of sets in the UpSet comparison
  repo: https://github.com/HelenaLC/muscat
- name: mmDS
  role: cell-level mixed-model DS engine whose significant calls form the second class
    of sets in the UpSet comparison
  repo: https://github.com/HelenaLC/muscat
- name: edgeR
  role: bulk RNA-seq differential engine applied to pseudobulk counts within pbDS;
    one of the aggregation-based methods grouped in the UpSet plot
- name: R
  role: language in which the result tables are loaded, gene sets filtered at p_adj.loc
    < 0.05, and the UpSet plot built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:3af7708229aa5e0c6da86c6249117c9592007d42467cfa1541ed5b947c332211
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HelenaLC/muscat
  tools:
  - R
  - edgeR
  - mmDS
  - muscat
  - pbDS
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# upset-plot-method-comparison-visualization

## Summary

Builds UpSet plots of intersection sizes between top-ranked differential state (DS) detections (Benjamini-Hochberg p_adj.loc < 0.05) from the 16 muscat DS methods, to visualize between-method concordance and isolate method-unique hits for truth assessment in simulation replicates.

## When to use

Use when comparing which genes multiple DS analysis methods (pseudobulk pbDS engines vs. cell-level mmDS mixed models) call significant at FDR < 0.05, and you need to visualize set intersections across methods and replicates.

## When NOT to use

- The analysis goal is per-method performance metrics (TPR, FDR, AUC) rather than between-method agreement of significant calls
- Results come from a single method or single replicate, so there are no cross-method or cross-replicate gene sets to intersect
- Input is raw counts or pseudobulk aggregates rather than completed DS result tables with adjusted p-values

## Inputs

- DS analysis result tables (gene x subpopulation p-values and adjusted p-values) from muscat pbDS and mmDS across five simulation replicates per DS category
- Significant gene sets per method (p_adj.loc < 0.05) per cell subpopulation
- Simulation ground-truth labels distinguishing true differential state genes from null genes

## Outputs

- UpSet plot of method-wise intersection sizes of top-ranked DS detections
- Per-intersection gene lists tagged as shared across methods or method-unique
- Proportion of isolated (method-unique) intersections that are true vs. false discoveries

## How to apply

Load the per-method DS result tables from the muscat simulation framework (five replicates per DS category, produced by pbDS pseudobulk engines such as edgeR and limma-voom and by mmDS mixed-model engines). For each method, replicate, and cell subpopulation, extract genes whose Benjamini-Hochberg-adjusted p-value (p_adj.loc, computed via p.adjust with method='fdr') is below 0.05. Intersect significant gene sets across replicates within each DS category, then compute intersection sizes across all methods, grouping aggregation-based methods separately from mixed-model (MM) methods. Build an UpSet plot ordering sets to highlight agreement among aggregation- and MM-based methods versus isolated, method-unique discoveries. Validate by checking that the plot shows high concordance among aggregation- and MM-based methods and that isolated intersections are predominantly vstresiduals-based false discoveries, matching the paper's reported structure.

## Related tools

- **muscat** (R package providing the pbDS and mmDS DS analysis engines whose top-ranked detections are intersected and visualized; installed via BiocManager::install('muscat')) — https://github.com/HelenaLC/muscat
- **pbDS** (pseudobulk aggregation-based DS engine (edgeR, DESeq2, limma-voom methods) whose significant calls form one class of sets in the UpSet comparison) — https://github.com/HelenaLC/muscat
- **mmDS** (cell-level mixed-model DS engine whose significant calls form the second class of sets in the UpSet comparison) — https://github.com/HelenaLC/muscat
- **edgeR** (bulk RNA-seq differential engine applied to pseudobulk counts within pbDS; one of the aggregation-based methods grouped in the UpSet plot)
- **R** (language in which the result tables are loaded, gene sets filtered at p_adj.loc < 0.05, and the UpSet plot built)

## Examples

```
sig <- subset(res$table, p_adj.loc < 0.05)  # per method/replicate/cluster, then intersect gene sets and plot UpSetR intersections
```

## Evaluation signals

- The UpSet plot shows high concordance (large shared intersections) among aggregation-based methods and among MM-based methods, matching the reported Fig. 3 structure
- Filtering check: extracted gene sets equal exactly the rows with Benjamini-Hochberg p_adj.loc < 0.05 recomputed via p.adjust(method='fdr') on p_val.loc
- The majority of genes in common (multi-method) intersections are truly differential according to simulation ground truth
- Most isolated (method-unique) intersections are false discoveries, with a high proportion attributable to vstresiduals-based methods
- Intersection sizes are computed after intersecting across the five replicates within each DS category, not on single-replicate calls

## Limitations

- Isolated discoveries are dominated by false positives, so method-unique hits should not be treated as novel true detections without validation
- Conclusions depend on the muscat simulation framework's scenarios; other scenarios of interest (e.g., different percentages of DM mixtures) were not tested
- LogFC estimates were attenuated for some combinations depending on the scale of the data analyzed, which can affect ranking-based comparisons

## Evidence

- [full_text] Top-ranked DS detections showed overall high concordance between methods, with the majority of common hits being truly differential, while most isolated intersections (hits unique to a single method) were false discoveries.: [evidence span withheld — anchor status: not_found]
- [methods] p_adj.loc computed via p.adjust with method='fdr': [evidence span withheld — anchor status: not_found]
- [readme] README quick guide pbDS and mmDS commands: [evidence span withheld — anchor status: external_doc]
- [readme] README mmDS invocation: "ds_mm <- mmDS(sce, method = "dream")"
- [abstract] Article abstract on surveyed methods including cell-level mixed models and pseudobulk aggregation: "Here, we surveyed methods to perform cross-condition differential state analyses, including cell-level mixed models and methods based on aggregated pseudobulk data."
