---
name: cross-database-annotation-accuracy-comparison
description: Use when you need to quantify or compare annotation accuracy across alternative
  marker gene databases for a single-cell cluster annotation workflow with known ground-truth
  cell-type labels.
when_to_use_negative:
- You have no ground-truth (simulated or manually curated) cell-type labels, so accuracy
  cannot be scored per database
- You only need a single annotation for one database — use the standard easyct annotation
  workflow instead of a cross-database comparison
- Input is raw count data rather than a processed marker list from differential expression
  analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: Provides easyct() run per db option ('cellmarker' plus corresponding PanglaoDB
    and Clustermole options) with test='GSEA' or test='fisher', plus plot_dot/plot_bar
    visualization
  repo: https://github.com/rx-li/EasyCellType
- name: clusterProfiler
  role: Performs the gene set enrichment analysis scoring underlying the GSEA test
    mode of easyct
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:ef20ee89560aecfff2c1a0f7d22be5117fe7d72fe692363bcb6239ea17ad8280
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-database-annotation-accuracy-comparison

## Summary

Benchmarks how the choice of cell marker database (CellMarker, PanglaoDB, or Clustermole) affects EasyCellType annotation accuracy by running easyct with GSEA and modified Fisher's exact test on the same simulated PBMC marker input and scoring results against known labels.

## When to use

Use when you need to quantify or compare annotation accuracy across alternative marker gene databases for a single-cell cluster annotation workflow with known ground-truth cell-type labels.

## When NOT to use

- You have no ground-truth (simulated or manually curated) cell-type labels, so accuracy cannot be scored per database
- You only need a single annotation for one database — use the standard easyct annotation workflow instead of a cross-database comparison
- Input is raw count data rather than a processed marker list from differential expression analysis

## Inputs

- Simulated 2400-cell PBMC marker dataset with known cluster-level cell-type labels (Section 3.4 simulation design)
- EasyCellType input data frame of Entrez IDs, cluster labels and expression scores, sorted by score within each cluster
- Tissue keyword tables: cellmarker_tissue, panglao_tissue, clustermole_tissue (EasyCellType package data)

## Outputs

- annotation_accuracy_comparison.csv (accuracy per database and per test)
- Per-cluster annotation recommendation dot plots (plot_dot) and bar plots (plot_bar) for each database/test combination

## How to apply

Load (or regenerate per the Section 3.4 simulation design) the simulated 2400-cell PBMC marker dataset with known cluster-level cell-type labels, formatted as an EasyCellType input data frame of Entrez IDs, cluster labels and expression scores sorted by score within each cluster. Inspect candidate tissue keywords with data(cellmarker_tissue), data(panglao_tissue) and data(clustermole_tissue) and apply the tissue filters specified in Section 3.4 per database. Run easyct with test='GSEA', species='Human', p_cut=0.3 once per db option (enrichment via clusterProfiler's GSEA), then repeat with test='fisher' for the modified Fisher's exact test. Summarize per-cluster recommendations with plot_dot and plot_bar, then score accuracy per database and per test against the simulated labels and export annotation_accuracy_comparison.csv. The expected ordering — CellMarker highest and PanglaoDB lowest accuracy under both tests — serves as the validation target.

## Related tools

- **EasyCellType** (Provides easyct() run per db option ('cellmarker' plus corresponding PanglaoDB and Clustermole options) with test='GSEA' or test='fisher', plus plot_dot/plot_bar visualization) — https://github.com/rx-li/EasyCellType
- **clusterProfiler** (Performs the gene set enrichment analysis scoring underlying the GSEA test mode of easyct)

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", test="GSEA", p_cut=0.3)
```

## Evaluation signals

- annotation_accuracy_comparison.csv reproduces the paper's ordering: CellMarker attains the highest and PanglaoDB the lowest annotation accuracy under both GSEA and the modified Fisher's exact test
- All six runs (3 databases x 2 tests) use identical inputs: test='GSEA'/'fisher', species='Human', p_cut=0.3, with clusterProfiler GSEA for enrichment
- Input data frame contains Entrez IDs, cluster labels and expression scores, sorted by score within each cluster
- Tissue filters applied per database match the Section 3.4 specifications derived from cellmarker_tissue/panglao_tissue/clustermole_tissue
- Each run yields per-cluster annotation recommendations visualizable via plot_dot and plot_bar

## Limitations

- Findings rest on a simulated 2400-cell PBMC dataset; ordering of database accuracy may differ on other tissues or species
- The article enumerates '3 public available marker gene data bases' without fully listing tissue database names, so tissue filter setup requires consulting the package data objects
- Simply overlapping gene lists without considering gene ranking might lead to unreliable results, so ranked input is required

## Evidence

- [results] CellMarker highest accuracy: [evidence span withheld — anchor status: not_found]
- [abstract] two statistical tests: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [abstract] marker ranking rationale: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [methods] easyct invocation: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [intro] visualization output: "The package generates straight-forward visualization of the test results, which help the researchers to understand the results and pick the optimal annotations"
