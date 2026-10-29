---
name: fdr-threshold-gene-intersection
description: Use when comparing DS/DE method outputs across replicates to assess concordance
  of significant gene calls (p_adj < 0.05) and classify shared versus method-unique
  hits as true or false discoveries.
when_to_use_negative:
- When a single-method, single-replicate differential expression call list is all
  that is needed — this skill is specifically about cross-method, cross-replicate
  concordance
- When the input data are cell-level raw counts rather than completed DS analysis
  result tables; run muscat pbDS/mmDS first before intersecting significant hits
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: muscat
  role: R package providing pbDS (pseudobulk DS) and mmDS (mixed-model DS) engines
    whose result tables are intersected at the FDR < 0.05 level
  repo: github.com/HelenaLC/muscat
- name: pbDS
  role: Pseudobulk DS analysis function producing result tables (e.g., edgeR, limma
    aggregation-based methods) used for significant-gene extraction
  repo: github.com/HelenaLC/muscat
- name: mmDS
  role: Cell-level mixed-model DS analysis function producing result tables used alongside
    pbDS outputs in the concordance analysis
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Bulk RNA-seq differential engine applied to pseudobulk counts by pbDS; its
    significant calls are one method set in the UpSet intersection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:4bd9658dcfd8a1219fd822d02c638f24cd1711fcbfdfe8e48b49b46f4db50b48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - mmDS
  - muscat
  - pbDS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# fdr-threshold-gene-intersection

## Summary

Extract Benjamini–Hochberg-adjusted p-value < 0.05 genes from differential state (DS) analysis result tables produced by multiple methods (e.g., muscat pbDS and mmDS engines) across simulation replicates, then compute and visualize method-wise gene set intersections (e.g., via UpSet plots) to quantify between-method concordance of top-ranked detections.

## When to use

Use when comparing DS/DE method outputs across replicates to assess concordance of significant gene calls (p_adj < 0.05) and classify shared versus method-unique hits as true or false discoveries.

## When NOT to use

- When a single-method, single-replicate differential expression call list is all that is needed — this skill is specifically about cross-method, cross-replicate concordance
- When the input data are cell-level raw counts rather than completed DS analysis result tables; run muscat pbDS/mmDS first before intersecting significant hits

## Inputs

- DS analysis result tables per method, replicate, and cell subpopulation from pbDS (pseudobulk) and mmDS (mixed-model) engines
- Significance thresholds: BH-adjusted p-value cutoff of 0.05 (p_adj.loc via p.adjust(method='fdr'))
- DS category and replicate identifiers from the muscat simulation framework

## Outputs

- Per-method sets of significant genes (p_adj < 0.05) per cell subpopulation and replicate
- Intersection size table across methods (UpSet input)
- UpSet plot of method-wise intersections showing concordance vs. isolated discoveries

## How to apply

Load DS analysis result tables for each method, replicate, and cell subpopulation, produced by the muscat pseudobulk engine pbDS and mixed-model engine mmDS (with aggregation-based methods like edgeR and limma grouped separately from mixed-model methods). For each method-replicate-subpopulation combination, extract genes with Benjamini–Hochberg-adjusted p-value < 0.05 (p_adj.loc computed via p.adjust with method='fdr'). Intersect significant gene sets across replicates within each DS category, compute intersection sizes across all methods, and build an UpSet plot ordered to highlight agreement among aggregation- and MM-based methods versus isolated discoveries. Correct application is judged by whether the plot reproduces the reported structure: high concordance among aggregation- and MM-based methods, with the majority of common hits truly differential and most isolated (single-method) intersections being false discoveries, notably attributable to vstresiduals-based methods.

## Related tools

- **muscat** (R package providing pbDS (pseudobulk DS) and mmDS (mixed-model DS) engines whose result tables are intersected at the FDR < 0.05 level) — github.com/HelenaLC/muscat
- **pbDS** (Pseudobulk DS analysis function producing result tables (e.g., edgeR, limma aggregation-based methods) used for significant-gene extraction) — github.com/HelenaLC/muscat
- **mmDS** (Cell-level mixed-model DS analysis function producing result tables used alongside pbDS outputs in the concordance analysis) — github.com/HelenaLC/muscat
- **edgeR** (Bulk RNA-seq differential engine applied to pseudobulk counts by pbDS; its significant calls are one method set in the UpSet intersection)

## Examples

```
pb <- aggregateData(sce, assay="counts", fun="sum", by=c("cluster_id","sample_id")); ds_pb <- pbDS(pb, method="edgeR")
```

## Evaluation signals

- Significant gene extraction uses Benjamini–Hochberg-adjusted p-values strictly < 0.05 (p_adj.loc computed via p.adjust with method='fdr')
- Intersection sizes are computed across all methods per DS category, with aggregation-based (edgeR, limma) and mixed-model methods grouped separately
- Reproduced UpSet plot shows high concordance among aggregation- and MM-based methods
- Majority of common hits are truly differential (known simulated DS genes), while most isolated intersections are false discoveries
- High proportion of isolated discoveries is attributable to vstresiduals-based methods

## Limitations

- LogFC estimates were attenuated for some method/data-scale combinations, which can shift which genes pass the FDR threshold
- Statistical power is potentially biased toward larger cell populations, with decreased power for rarer populations, affecting method-unique hits
- MM methods carry higher computational cost that 'may not be worth the flexibility they provide', and FDR control shows instability with the number of cells that is not yet understood

## Evidence

- [intro] pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25.: "pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25."
- [intro] All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates.: "All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates."
- [methods] pbDS or mmDS produce per-celltype differential state analysis results: "`bbhw` can be executed on top of any per-celltype differential state analysis as produced by `pbDS` or `mmDS`"
- [readme] muscat provides methods for DS analyses in multi-sample multi-group scRNA-seq data: "provides methods for *Differential State* (DS) analyses in scRNA-seq data
with multiple samples, groups, and (cell)-subpopulations"
- [discussion] MM methods computational cost may not be worth the flexibility: "While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide"
