---
name: pseudotemporal-gene-clustering-and-visualization
description: Use when Lamian XDE test results (per-gene fdr.overall from lamian_test/lamian_test_h5,
  e.g. sex tested at testvar=2 with batch columns adjusted) are available and significant
  genes must be grouped into pseudotemporal pattern clusters and plotted.
when_to_use_negative:
- No genes pass the XDE significance cutoff (FDR.overall < 0.05), so there are no
  differential pseudotemporal patterns to cluster or plot.
- The question is differential topology or cell-density change (XCD) rather than gene-expression
  change along pseudotime; clusterGene/plotXDEHm operate on XDE gene results, not
  abundance or topology tests.
- The trajectory was built with Monocle2, Monocle3, or Slingshot, which Lamian's modules
  1 and 2 do not support.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Lamian
  role: Provides getPopulationFit, getCovariateGroupDiff, clusterGene, plotXDEHm,
    and plotClusterMeanAndDiff to fit, cluster, and visualize covariate-associated
    pseudotemporal expression patterns of XDE genes
  repo: https://github.com/Winnie09/Lamian
- name: ComplexHeatmap
  role: Bioconductor dependency behind Lamian's heatmap plotting (plotXDEHm); required
    for the heatmap visualization output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:4ececd4c1809287a67241ddd384b77b2604c7b7480ccf0d14b68bc490db55728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - ComplexHeatmap
  - Lamian
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pseudotemporal-gene-clustering-and-visualization

## Summary

Clusters Lamian XDE genes passing FDR.overall < 0.05 by their covariate-specific fitted pseudotemporal expression patterns (clusterGene, e.g. k=14) and visualizes original, population-fitted, and covariate-group-difference profiles along pseudotime with plotXDEHm and plotClusterMeanAndDiff.

## When to use

Use when Lamian XDE test results (per-gene fdr.overall from lamian_test/lamian_test_h5, e.g. sex tested at testvar=2 with batch columns adjusted) are available and significant genes must be grouped into pseudotemporal pattern clusters and plotted.

## When NOT to use

- No genes pass the XDE significance cutoff (FDR.overall < 0.05), so there are no differential pseudotemporal patterns to cluster or plot.
- The question is differential topology or cell-density change (XCD) rather than gene-expression change along pseudotime; clusterGene/plotXDEHm operate on XDE gene results, not abundance or topology tests.
- The trajectory was built with Monocle2, Monocle3, or Slingshot, which Lamian's modules 1 and 2 do not support.

## Inputs

- Lamian differential test result object with per-gene statistics including fdr.overall (output of lamian_test or lamian_test_h5)
- Gene-by-cell expression matrix stored as HDF5 via Lamian's saveh5() for atlas-scale data (e.g., 337,191 TB memory T cells)
- Per-cell pseudotime values (e.g., NAM-PC2 pseudotime)
- Cell-to-sample annotation and a sample-level design matrix with intercept, the tested covariate (e.g., sex), and batch indicator columns

## Outputs

- Cluster membership assignments partitioning FDR-significant XDE genes into k pattern groups (k=14 in the TB analysis)
- Population-level fitted expression values along pseudotime per gene (getPopulationFit output)
- Covariate group difference values per gene (getCovariateGroupDiff output)
- Heatmaps of original, fitted, and difference expression patterns along pseudotime (plotXDEHm)
- Cluster mean fitted-expression and group-difference plots (plotClusterMeanAndDiff)

## How to apply

Run the XDE test first (lamian_test or, for atlas-scale h5 data, lamian_test_h5 with test.type='variable', testvar=2 for sex, and batch indicator columns in the design) and select genes with FDR.overall < 0.05. Compute each significant gene's population-level fit with getPopulationFit(type='variable') and the covariate group difference with getCovariateGroupDiff, where type='variable' returns the covariate-dependent fitted patterns rather than overall means. Partition genes by fitted pattern with clusterGene(type='variable', k=14); in the TB atlas (GSE158769, 337,191 memory T cells from 184 donors) this grouped the ~1120 sex-associated XDE genes into 14 clusters. Visualize with plotXDEHm to show original, fitted, and difference expression along pseudotime, and plotClusterMeanAndDiff to inspect each cluster's mean fitted trajectory and group difference. Choose k according to pattern diversity and verify that each cluster's mean difference curve agrees with per-gene getCovariateGroupDiff directions before interpreting clusters as biological programs.

## Related tools

- **Lamian** (Provides getPopulationFit, getCovariateGroupDiff, clusterGene, plotXDEHm, and plotClusterMeanAndDiff to fit, cluster, and visualize covariate-associated pseudotemporal expression patterns of XDE genes) — https://github.com/Winnie09/Lamian
- **ComplexHeatmap** (Bioconductor dependency behind Lamian's heatmap plotting (plotXDEHm); required for the heatmap visualization output)

## Examples

```
R: fit <- getPopulationFit(testres, type='variable'); diff <- getCovariateGroupDiff(testres, type='variable'); clt <- clusterGene(testres, type='variable', k=14)
```

## Evaluation signals

- The clustered XDE set is defined at FDR.overall < 0.05 from lamian_test/lamian_test_h5 output; for the TB atlas this should approximate the reported 1120 sex-associated XDE genes.
- clusterGene(type='variable', k=14) returns exactly 14 clusters covering all selected genes, matching the reported 14 clusters.
- plotXDEHm renders aligned original, fitted, and difference heatmaps whose rows are grouped by cluster assignment.
- Cluster-mean difference curves from plotClusterMeanAndDiff agree in sign and direction with per-gene getCovariateGroupDiff values within each cluster.
- With batch indicators included in the design (testvar=2), cluster patterns and XDE calls reproduce across the 12 sample splits, consistent with Fig. 6h.

## Limitations

- The number of clusters k is user-chosen (k=14 in the TB analysis); an inappropriate k can merge or fragment distinct pseudotemporal patterns.
- Heatmap plotting depends on ComplexHeatmap; if it is removed from Lamian's Imports during install troubleshooting, plotXDEHm will not run while other functions still work.
- Only genes passing the XDE FDR threshold are clustered, and results are conditional on the upstream pseudotime and batch-adjusted XDE test; pseudotime inference uncertainty is not simultaneously accounted for.

## Evidence

- [other] Select XDE genes with FDR.overall < 0.05, compute population-level fits and covariate group differences with getPopulationFit(type='variable') and getCovariateGroupDiff: [evidence span withheld — anchor status: not_found]
- [other] and cluster the genes with clusterGene(type='variable', k=14); visualize original, fitted, and difference patterns with plotXDEHm and plotClusterMeanAndDiff: [evidence span withheld — anchor status: not_found]
- [other] the number of sex-associated XDE genes should approximate the reported 1120 genes across 14 clusters: [evidence span withheld — anchor status: not_found]
- [abstract] Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects: "Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects"
- [readme] The function to plot heatmap will not work in this case, but other functions will still be working.: [evidence span withheld — anchor status: external_doc]
