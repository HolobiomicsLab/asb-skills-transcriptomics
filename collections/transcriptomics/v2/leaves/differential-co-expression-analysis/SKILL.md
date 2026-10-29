---
name: differential-co-expression-analysis
description: Use when you have single-cell UMI count data with per-cell group labels
  (e.g. disease status) and per-cell donor/sample labels, and need to test whether
  gene co-expressions within a specific cell type differ between conditions.
when_to_use_negative:
- When the goal is baseline co-expression within one condition rather than a between-group
  comparison — use the CSCORE estimator directly instead of diff_CSCORE
- When group labels are at the cell level but samples are single subjects with no
  donor replication — donor-level permutation cannot be performed and the test is
  underpowered
- When input data are normalized/scaled expression values rather than raw UMI counts
  — CS-CORE's measurement model requires raw counts
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CS-CORE
  role: Framework providing diff_CSCORE for differential co-expression testing with
    donor-level permutation
  repo: github.com/ChangSuBiostats/CS-CORE
- name: Seurat
  role: Holds and subsets the postmortem brain scRNA-seq object (AD and control microglia)
    with group and donor metadata
- name: WGCNA
  role: Extracts gene modules from significant differential co-expressions (adjacency.fromSimilarity
    power = 1, TOMsimilarity)
- name: dynamicTreeCut
  role: cutreeDynamic clustering of the TOM dendrogram with minClusterSize = 10 to
    define differential co-expression modules
- name: clusterProfiler
  role: enrichGO annotation of differential co-expression modules against GO with
    BH p-value adjustment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:a03a5b388f8a01aaf59c26d21127f8b54613f355a45b4849f3952de3ff5c3719
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE
  - Seurat
  - WGCNA
  - clusterProfiler
  - diff_CSCORE
  - dynamicTreeCut
  - p.adjust BH
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# differential-co-expression-analysis

## Summary

Identify gene pairs whose cell-type-specific co-expression differs between disease and control groups using CS-CORE's diff_CSCORE, which tests differential co-expression from raw UMI counts with donor/sample-level permutations. In the Alzheimer's disease postmortem brain study, this surfaces microglia co-expression changes specific to AD versus controls.

## When to use

Use when you have single-cell UMI count data with per-cell group labels (e.g. disease status) and per-cell donor/sample labels, and need to test whether gene co-expressions within a specific cell type differ between conditions.

## When NOT to use

- When the goal is baseline co-expression within one condition rather than a between-group comparison — use the CSCORE estimator directly instead of diff_CSCORE
- When group labels are at the cell level but samples are single subjects with no donor replication — donor-level permutation cannot be performed and the test is underpowered
- When input data are normalized/scaled expression values rather than raw UMI counts — CS-CORE's measurement model requires raw counts

## Inputs

- Seurat object or raw UMI count matrix for a single cell type, with per-cell disease/control group labels (e.g. AD vs control microglia)
- Per-cell donor/sample identifier column for subject-level permutation
- Cell-type subsetting labels (e.g. the five major brain cell types from GSE157827)

## Outputs

- Differentially co-expressed gene pairs with CS-CORE test statistics and BH-adjusted p-values (AD vs control microglia)
- Significant differential co-expressions filtered at BH-adjusted p < 0.05
- Co-expressed gene modules extracted from differential co-expressions (WGCNA dynamicTreeCut modules, minClusterSize = 10)
- GO term enrichment tables per module (enrichGO, org.Hs.eg.db, BH-adjusted p < 0.05)

## How to apply

Subset the Seurat object to the cell type of interest (in the article, microglia from AD patients and controls of GSE157827 postmortem brain data, subsetting by cell type to avoid Simpson's paradox) keeping raw UMI counts. Run diff_CSCORE with group_label set to disease status and sample_label set to donor, using 100 permutations performed at the donor/sample level so that the test respects the structure of repeated measures within subjects. Threshold the resulting differential co-expression p-values with Benjamini-Hochberg adjustment, retaining pairs with BH-adjusted p < 0.05. Optionally extract co-expressed modules from the differential co-expressions via the WGCNA workflow (adjacency.fromSimilarity with power = 1 on absolute estimates, TOMsimilarity, average-linkage hierarchical clustering, dynamicTreeCut::cutreeDynamic with minClusterSize = 10) and annotate modules with clusterProfiler::enrichGO (org.Hs.eg.db, keyType = SYMBOL, pAdjustMethod = BH). The donor-level permutation is essential because cells from the same subject are not independent; permuting cells instead would inflate significance.

## Related tools

- **CS-CORE** (Framework providing diff_CSCORE for differential co-expression testing with donor-level permutation) — github.com/ChangSuBiostats/CS-CORE
- **Seurat** (Holds and subsets the postmortem brain scRNA-seq object (AD and control microglia) with group and donor metadata)
- **WGCNA** (Extracts gene modules from significant differential co-expressions (adjacency.fromSimilarity power = 1, TOMsimilarity))
- **dynamicTreeCut** (cutreeDynamic clustering of the TOM dendrogram with minClusterSize = 10 to define differential co-expression modules)
- **clusterProfiler** (enrichGO annotation of differential co-expression modules against GO with BH p-value adjustment)

## Examples

```
diff_result <- diff_CSCORE(microglia_seu, group_label = 'Disease.status', sample_label = 'Donor', perm = 100)
```

## Evaluation signals

- All reported differential co-expression p-values must come from permutations performed at the donor/sample level (100 permutations), not the cell level
- Significant differential pairs pass BH-adjusted p < 0.05; top modules should show at least one GO term with BH-adjusted p < 0.001 and at least 10 enriched GO terms
- Differential co-expression estimates are reproducible/enriched relative to benchmark methods (rho-sctransform, rho-analytic PR, SpQN) on the same cells
- Results tables (differentially co-expressed pairs, module-GO term tables) compile into ad_coexpression_results.csv without unmatched gene symbols
- The analysis was restricted to a single cell type (e.g. microglia), consistent with the recommendation to subset by cell type to avoid Simpson's paradox

## Limitations

- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as the negative binomial is deferred to future work
- Inflated false positives and biased estimates in competing methods motivated CS-CORE, so cross-method comparisons of differential co-expressions must use the same cells and cutoffs

## Evidence

- [methods] Donor-level permutation with diff_CSCORE: "diff_result <- diff_CSCORE(pbmc_CD14,
                           group_label  = 'Status',
                           sample_label = 'Donor',"
- [other] Task workflow for AD microglia differential analysis: [evidence span withheld — anchor status: not_found]
- [readme] Repository description of the differential vignette: [evidence span withheld — anchor status: external_doc]
- [discussion] Assumption caveat from the discussion: "we have assumed that gene expressions from cells of the same cell type follow the same distribution."
- [other] Subsetting rationale: [evidence span withheld — anchor status: not_found]
