---
name: simulation-replicate-method-concordance
description: Use when you have DS analysis result tables from multiple differential
  methods (e.g., pbDS/edgeR/limma vs. mmDS) run over simulation replicates per DS
  category, and need to quantify and visualize which genes are jointly vs. uniquely
  detected at FDR < 0.05.
when_to_use_negative:
- You are analyzing real experimental data (e.g., the LPS-treated mouse cortex snRNA-seq
  application) rather than simulation replicates with known ground truth — method-unique
  hits cannot be classified as true/false discoveries without a truth set.
- You need to rank or compare methods by sensitivity/FDR/computational cost rather
  than characterize agreement among top-ranked detections.
- Only a single method (or a single aggregation-based method) was run — no between-method
  intersections can be computed.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: muscat
  role: R package providing pbDS (pseudobulk aggregation-based DS analysis) and mmDS
    (cell-level mixed-model DS analysis) whose result tables are intersected across
    simulation replicates
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Aggregation-based differential engine applied to pseudobulk counts via pbDS;
    one of the methods whose significant hits enter the concordance analysis
- name: limma-voom
  role: Aggregation-based differential engine applied to pseudobulk counts via pbDS;
    one of the methods whose significant hits enter the concordance analysis
- name: R
  role: Implementation environment for extracting p_adj.loc hits, computing intersections,
    and building the UpSet plot
- name: Snakemake
  role: Workflow manager used to run the simulation replicates supplying the per-method
    result tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:48ced4072fe06ed6658c1367da35fb150aa736d80f660dcba530dcf08a4350af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Snakemake
  - edgeR
  - limma-voom
  - mmDS
  - muscat
  - pbDS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# simulation-replicate-method-concordance

## Summary

Assesses between-method concordance of differential state (DS) detections across simulation replicates by intersecting method-wise significant hits (BH-adjusted p < 0.05) from muscat's pseudobulk (pbDS) and mixed-model (mmDS) engines and visualizing overlaps as an UpSet plot, distinguishing true from isolated, often false, discoveries.

## When to use

Use when you have DS analysis result tables from multiple differential methods (e.g., pbDS/edgeR/limma vs. mmDS) run over simulation replicates per DS category, and need to quantify and visualize which genes are jointly vs. uniquely detected at FDR < 0.05.

## When NOT to use

- You are analyzing real experimental data (e.g., the LPS-treated mouse cortex snRNA-seq application) rather than simulation replicates with known ground truth — method-unique hits cannot be classified as true/false discoveries without a truth set.
- You need to rank or compare methods by sensitivity/FDR/computational cost rather than characterize agreement among top-ranked detections.
- Only a single method (or a single aggregation-based method) was run — no between-method intersections can be computed.

## Inputs

- DS analysis result tables (per gene, per cluster) for five simulation replicates per DS category from pbDS (pseudobulk, e.g., edgeR/limma-voom methods)
- DS analysis result tables from mmDS mixed-model methods (e.g., dream) run on cell-level SingleCellExperiment data
- Simulated multi-sample multi-group scRNA-seq data with known true differential state genes (from the muscat simulation framework)

## Outputs

- Per-method sets of significant genes (BH-adjusted p < 0.05) per replicate and subpopulation
- Intersection size matrix of method-wise gene sets per DS category
- UpSet plot showing method-wise overlaps, highlighting concordant vs. isolated detections
- Qualitative call on whether isolated intersections are mostly true or false discoveries (against simulated ground truth)

## How to apply

Load the DS result tables for the five simulation replicates per DS category produced by muscat's pseudobulk engine pbDS (e.g., with edgeR or limma on aggregated counts) and the mixed-model engine mmDS run directly on cell-level measurements. For each method, replicate, and cell subpopulation, extract genes with Benjamini-Hochberg-adjusted p-value < 0.05 (p_adj.loc computed via p.adjust with method='fdr'). Intersect the significant gene sets across replicates within each DS category and compute intersection sizes across all methods, grouping aggregation-based methods (edgeR, limma) separately from mixed-model methods. Build an UpSet plot of the method-wise intersections, ordering sets to highlight agreement among aggregation- and MM-based methods versus isolated discoveries. The reported structure is high concordance between methods with most common hits truly differential, while most method-unique (isolated) hits are false discoveries attributable largely to vstresiduals-based methods.

## Related tools

- **muscat** (R package providing pbDS (pseudobulk aggregation-based DS analysis) and mmDS (cell-level mixed-model DS analysis) whose result tables are intersected across simulation replicates) — github.com/HelenaLC/muscat
- **edgeR** (Aggregation-based differential engine applied to pseudobulk counts via pbDS; one of the methods whose significant hits enter the concordance analysis)
- **limma-voom** (Aggregation-based differential engine applied to pseudobulk counts via pbDS; one of the methods whose significant hits enter the concordance analysis)
- **R** (Implementation environment for extracting p_adj.loc hits, computing intersections, and building the UpSet plot)
- **Snakemake** (Workflow manager used to run the simulation replicates supplying the per-method result tables)

## Examples

```
pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")); ds_pb <- pbDS(pb, method = "edgeR"); sig <- subset(res$table$cluster[[k]], p_adj.loc < 0.05)
```

## Evaluation signals

- Adjusted p-values use the muscat p_adj.loc slot computed with p.adjust(method='fdr'); significance threshold is BH-adjusted p < 0.05, applied per method, replicate, and subpopulation.
- Each DS category must contribute five simulation replicates whose significant gene sets are intersected before computing cross-method intersection sizes.
- Reproduction is validated by the UpSet plot showing high concordance among aggregation-based (edgeR, limma) and MM-based methods for top-ranked detections.
- A high proportion of isolated intersections (method-unique hits) should be attributable to vstresiduals-based methods and classified as false discoveries against simulation ground truth.
- Intersection sets are grouped consistently: aggregation-based methods separately from mixed-model methods when ordering UpSet sets.

## Limitations

- Aggregation-based DS methods showed overall stable high performance, but mixed-model (MM) methods performed similarly well at higher computational cost, which may not be worth the flexibility they provide.
- Instability of FDR control with the number of cells was observed and its origins are not yet understood.
- There is potentially strong bias in statistical power toward detecting changes in larger cell populations, with decreased power for rarer populations, which can shape which methods yield unique detections.
- Depending on the scale of the data analyzed, logFCs were attenuated for some method/data combinations, complicating effect-size comparisons across methods.

## Evidence

- [intro] pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25.: "pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25."
- [intro] All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates.: "All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates."
- [discussion] While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide: "While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide"
- [readme] Mixed models can be run directly on cell-level measurements, e.g.: ds_mm <- mmDS(sce, method = "dream"): [evidence span withheld — anchor status: external_doc]
- [readme] # run pseudobulk (aggregation-based) DS analysis ds_pb <- pbDS(pb, method = "edgeR"): "# run pseudobulk (aggregation-based) DS analysis
ds_pb <- pbDS(pb, method = "edgeR")"
