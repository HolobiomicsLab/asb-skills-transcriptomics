---
name: pseudotime-trajectory-inference
description: Use when single-cell RNA-Seq data from two or more conditions must be
  placed on a common trajectory to ask whether conditions differ in topology, pseudotime
  progression, fate selection between lineages, or gene expression along lineages.
when_to_use_negative:
- The system is at steady state with no dynamic process — clustering plus differential
  abundance methods (milo, DAseq) is the appropriate approach instead
- Only one condition is present, so there is no between-condition comparison for topologyTest,
  progressionTest, or fateSelectionTest
- The goal is differential abundance without any trajectory structure, where cluster-based
  methods suffice
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Core package providing imbalance_score, topologyTest, progressionTest, fateSelectionTest,
    and toy_dataset; installed from Bioconductor via BiocManager::install('condiments')
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory (lineages and pseudotime) on the reduced-dimension
    matrix using cluster labels
- name: tradeSeq
  role: Fits gene-wise smoothers with fitGAM(counts, sds, conditions) to support conditionTest
    gene-level differential expression testing
  repo: statomics.github.io/tradeSeq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:d80526e8f85fec46ffb695fc9469ab167396c679b796027d0bffcd617d22f328
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pseudotime-trajectory-inference

## Summary

A three-step condiments workflow for inferring and testing cell trajectories across multiple conditions: differential topology (topologyTest), trajectory-level abundance changes (progressionTest, fateSelectionTest), and gene-level differential expression along lineages (conditionTest), applied here to the fibrosis ILD scRNA-Seq case study.

## When to use

Use when single-cell RNA-Seq data from two or more conditions must be placed on a common trajectory to ask whether conditions differ in topology, pseudotime progression, fate selection between lineages, or gene expression along lineages.

## When NOT to use

- The system is at steady state with no dynamic process — clustering plus differential abundance methods (milo, DAseq) is the appropriate approach instead
- Only one condition is present, so there is no between-condition comparison for topologyTest, progressionTest, or fateSelectionTest
- The goal is differential abundance without any trajectory structure, where cluster-based methods suffice

## Inputs

- Single-cell count matrix (e.g. fibrotic lung scRNA-Seq from Habermann et al., control vs ILD)
- Per-cell condition labels (control/ILD)
- Cluster labels for cells
- Low-dimensional reduced representation (e.g. Dim1/Dim2 coordinates) for slingshot
- toy_dataset from the condiments package for workflow testing

## Outputs

- Imbalance scores per cell visualizing local condition imbalance
- A slingshot SlingshotDataSet (sds) defining lineages and pseudotime
- TopologyTest, progressionTest (global and per-lineage), and fateSelectionTest statistics and p-values
- Gene-level conditionTest results with FDR 5% significant differential expression genes
- condiments_results_table.csv summarizing all test results

## How to apply

First assess local condition imbalance with imbalance_score, then infer a single common trajectory with slingshot on the reduced-dimension matrix using cluster labels. Run the permutation-based topologyTest (sds, conditions) to decide between a common and condition-specific topology; if the common topology is retained, compare pseudotime distributions per lineage with progressionTest (global = TRUE, lineages = TRUE, default Kolmogorov-Smirnov test) and test differential fate selection with fateSelectionTest (pairwise = TRUE, default classifier test of Lopez-Paz and Oquab, 2016) on lineage weights. Finally fit gene-wise smoothers with tradeSeq fitGAM(counts, sds, conditions) and call conditionTest, selecting genes at FDR 5% (patternTest lineage-level DE as a secondary check). In the fibrosis case study this workflow recovered strong differential fate selection (fateSelectionTest p ≤ 2.2 × 10−16): control cells belong almost exclusively to Lineage 2 (AT2-derived toward AT1) while ILD cells also arise via the SCGB3A2+ lineage.

## Related tools

- **condiments** (Core package providing imbalance_score, topologyTest, progressionTest, fateSelectionTest, and toy_dataset; installed from Bioconductor via BiocManager::install('condiments')) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory (lineages and pseudotime) on the reduced-dimension matrix using cluster labels)
- **tradeSeq** (Fits gene-wise smoothers with fitGAM(counts, sds, conditions) to support conditionTest gene-level differential expression testing) — statomics.github.io/tradeSeq

## Examples

```
library(condiments); data("toy_dataset", package = "condiments"); sds <- slingshot(toy_dataset[,c('Dim1','Dim2')], clusterLabels = toy_dataset$cl); topologyTest(sds, toy_dataset$conditions)
```

## Evaluation signals

- TopologyTest fails to reject the common-trajectory null, confirming a shared topology across conditions
- FateSelectionTest is significant (in the fibrosis case study, p ≤ 2.2 × 10−16) with control cells mapping almost exclusively to Lineage 2 and ILD cells also via the SCGB3A2+ lineage
- ProgressionTest reports a global difference in pseudotime distributions, driven by lineage 2
- conditionTest returns genes selected at FDR 5% consistent with patternTest lineage-level DE as a secondary check
- All topology, progression, fate selection, and DE results are written to condiments_results_table.csv

## Limitations

- No independent benchmarking of the workflow has been performed; the authors call for it
- Real-world benchmarking case studies are still lacking and only becoming available as more datasets are published
- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved
- P-values are not given a probabilistic interpretation; they serve as numerical summaries only
- Batch-condition confounding can be inherent and unresolvable in some settings

## Evidence

- [abstract] we present condiments, a method for the inference and downstream interpretation of cell trajectories across multiple conditions: "we present condiments, a method for the inference and downstream interpretation of cell trajectories across multiple conditions"
- [intro] Our proposed analysis workflow is divided into three steps.: "Our proposed analysis workflow is divided into three steps."
- [methods] sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions): "sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)"
- [discussion] while in the Fibrosis case study we compare a normal system and its disease counterpart.: "while in the Fibrosis case study we compare a normal system and its disease counterpart."
- [readme] BiocManager::install("condiments"): [evidence span withheld — anchor status: external_doc]
