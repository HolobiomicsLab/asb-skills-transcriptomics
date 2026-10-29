---
name: trajectory-topology-testing
description: Use when scRNA-Seq data from ≥2 conditions (e.g. control vs ILD) are
  projected onto a common slingshot trajectory and you must test whether conditions
  differ in topology, pseudotime progression, fate selection between lineages, or
  gene expression along lineages.
when_to_use_negative:
- Steady-state systems where cells are best clustered into identities and assessed
  per cluster for differential abundance rather than projected onto a trajectory
- Data from a single condition with no comparison to test, since topology, progression,
  and fate-selection tests require ≥2 conditions
- You need a probabilistic interpretation of the p-values — the authors only attach
  numerical-summary status to them
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: condiments
  role: Core package providing imbalance_score, topologyTest, progressionTest, fateSelectionTest;
    installed via BiocManager
  repo: https://github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory (sds) on reduced-dimension coordinates and cluster
    labels, on which condiments tests operate
- name: tradeSeq
  role: Fits gene-wise smoothers (fitGAM) per condition along the trajectory and provides
    conditionTest for gene-level differential expression
  repo: http://statomics.github.io/tradeSeq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:6b56c833d44fdc5eff402b4d69af754b5227495105521eae2241baf58bf1277e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - condiments
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# trajectory-topology-testing

## Summary

The condiments workflow for comparing single-cell trajectories across multiple conditions: it integrates conditions into a single slingshot trajectory, then hierarchically tests differential topology (topologyTest), differential progression and fate selection (progressionTest, fateSelectionTest), and gene-level differential expression (tradeSeq conditionTest).

## When to use

Use when scRNA-Seq data from ≥2 conditions (e.g. control vs ILD) are projected onto a common slingshot trajectory and you must test whether conditions differ in topology, pseudotime progression, fate selection between lineages, or gene expression along lineages.

## When NOT to use

- Steady-state systems where cells are best clustered into identities and assessed per cluster for differential abundance rather than projected onto a trajectory
- Data from a single condition with no comparison to test, since topology, progression, and fate-selection tests require ≥2 conditions
- You need a probabilistic interpretation of the p-values — the authors only attach numerical-summary status to them

## Inputs

- Single-cell count matrices for each condition (e.g. scRNA-Seq UMI counts from Habermann fibrotic lung dataset or condiments toy_dataset)
- Per-cell condition labels (data frame, e.g. df$conditions)
- Cluster labels per cell for slingshot trajectory inference
- Low-dimensional cell embedding (e.g. Dim1/Dim2 reduced-dimension matrix)

## Outputs

- topologyTest p-value deciding common vs separate trajectory
- progressionTest statistics per lineage and globally (pseudotime distribution shifts)
- fateSelectionTest p-values (pairwise) for differential fate selection
- Imbalance scores per cell for local condition-imbalance visualization
- Gene-level differential expression results from conditionTest / patternTest at FDR 5% (e.g. exported to condiments_results_table.csv)

## How to apply

Integrate count matrices from each condition and infer a single common trajectory with slingshot on a low-dimensional representation (e.g. Dim1/Dim2), optionally computing local imbalance_score values to visualize condition imbalance. Run the permutation-based topologyTest(sds, conditions) to decide between a common and separate trajectory; if it fails to reject the common-trajectory null, proceed on the common trajectory. Test global differences with progressionTest (global = TRUE, lineages = TRUE, default Kolmogorov-Smirnov custom test) for pseudotime differences along lineages and fateSelectionTest (pairwise = TRUE, default classifier test of Lopez-Paz and Oquab, 2016) for between-lineage fate differences. Finally fit gene-wise smoothers with tradeSeq fitGAM(counts, sds, conditions) and call conditionTest, selecting genes at FDR 5%; patternTest serves as a secondary lineage-level DE check. In the fibrosis ILD case study this hierarchy correctly detected a significant fateSelectionTest (p ≤ 2.2e-16) with control cells on Lineage 2 and ILD cells also arising via the SCGB3A2+ lineage.

## Related tools

- **condiments** (Core package providing imbalance_score, topologyTest, progressionTest, fateSelectionTest; installed via BiocManager) — https://github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory (sds) on reduced-dimension coordinates and cluster labels, on which condiments tests operate)
- **tradeSeq** (Fits gene-wise smoothers (fitGAM) per condition along the trajectory and provides conditionTest for gene-level differential expression) — http://statomics.github.io/tradeSeq

## Examples

```
data("toy_dataset", package="condiments"); sds <- slingshot(df[,c("Dim1","Dim2")], df$clusters); top <- topologyTest(sds = sds, conditions = df$conditions)
```

## Evaluation signals

- In the fibrosis case study, topologyTest should fail to reject the common-trajectory null before proceeding to downstream tests
- fateSelectionTest should yield p ≤ 2.2e-16 with control cells assigned almost exclusively to Lineage 2 and ILD cells also via the SCGB3A2+ lineage
- progressionTest with global = TRUE should report a global pseudotime difference driven by lineage 2
- conditionTest gene calls should be selected at FDR 5% (patternTest lineage-level DE as secondary check)
- All test results should be assembled and written to condiments_results_table.csv for validation

## Limitations

- No independent benchmarking of the workflow has been performed; the authors explicitly call for it
- The dependence of pseudotime/weight estimates on the same data being tested (double-dipping) is ignored rather than resolved
- P-values are not given a probabilistic interpretation, only a numerical-summary status
- Batch-condition confounding can be inherent and unresolvable in some settings
- Real-world benchmarking case studies are still limited and only becoming available as more datasets are published

## Evidence

- [abstract] we present condiments, a method for the inference and downstream interpretation of cell trajectories across multiple conditions: "we present condiments, a method for the inference and downstream interpretation of cell trajectories across multiple conditions"
- [intro] In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection.: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [methods] sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions): "sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)"
- [readme] Start with the vignette [online](https://hectorrdb.github.io/condiments/articles/condiments.html) or explore on of our case studies in the paper: [evidence span withheld — anchor status: external_doc]
