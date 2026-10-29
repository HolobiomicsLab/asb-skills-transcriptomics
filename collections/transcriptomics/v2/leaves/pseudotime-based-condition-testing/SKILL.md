---
name: pseudotime-based-condition-testing
description: Use when single-cell RNA-seq data from two or more conditions (e.g.,
  mock vs TGF-β treatment) must be compared along a shared developmental trajectory,
  and you need to test differential topology, pseudotime progression, fate selection,
  or condition-specific gene expression along lineages.
when_to_use_negative:
- Cells are in a steady-state system with no dynamic transition, where cluster-based
  differential abundance methods (e.g., milo, DAseq) are appropriate
- Batch and condition are inherently confounded and cannot be separated, invalidating
  cross-condition trajectory comparison
- You lack per-cell condition labels or a reduced-dimension representation suitable
  for trajectory inference
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: condiments
  role: Runs topologyTest, progressionTest, fateSelectionTest, and imbalance_score
    for multi-condition trajectory testing
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory on reduced dimensions and produces pseudotime
    and lineage weights used by condiments tests
- name: tradeSeq
  role: Fits gene-wise negative-binomial GAMs (fitGAM) on counts with pseudotime (sds)
    and conditions, and runs conditionTest for gene-level DE along lineages
  repo: statomics.github.io/tradeSeq
- name: fgsea
  role: Gene-set enrichment on the ranked DE gene list to recover biological adhesion/locomotion
    enrichment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:3eb3fc717bc2932161a37b755beb71e19ddbfd2ae24f0f4ec43f390db3ae356b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - fgsea
  - ggplot2
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pseudotime-based-condition-testing

## Summary

A three-step condiments workflow that tests whether single-cell dynamic processes differ between conditions by leveraging trajectory structure: differential topology (topologyTest), differential progression and fate selection along lineages (progressionTest, fateSelectionTest), and gene-level differential expression along lineages (tradeSeq conditionTest).

## When to use

Use when single-cell RNA-seq data from two or more conditions (e.g., mock vs TGF-β treatment) must be compared along a shared developmental trajectory, and you need to test differential topology, pseudotime progression, fate selection, or condition-specific gene expression along lineages.

## When NOT to use

- Cells are in a steady-state system with no dynamic transition, where cluster-based differential abundance methods (e.g., milo, DAseq) are appropriate
- Batch and condition are inherently confounded and cannot be separated, invalidating cross-condition trajectory comparison
- You lack per-cell condition labels or a reduced-dimension representation suitable for trajectory inference

## Inputs

- Integrated, normalized count matrix with per-cell condition labels (e.g., mock vs TGF-beta)
- Low-dimensional cell embeddings / reduced-dimension coordinates (e.g., PCA or UMAP)
- Cluster labels used for slingshot trajectory inference
- Slingshot SlingshotDataSet (sds) with pseudotime and lineage weights

## Outputs

- topologyTest p-value deciding common vs separate trajectory topology
- progressionTest p-values (global and per-lineage) for differential pseudotime progression
- fateSelectionTest p-values for differential fate selection between lineages
- Gene-level conditionTest results with FDR-significant condition-specific DE genes (e.g., 1,993 at FDR 5%)
- Summary table of test statistics matching paper tables/figures (e.g., condiments_emt_results.csv)

## How to apply

First integrate and normalize counts across conditions, infer a common trajectory with slingshot on reduced dimensions, and run condiments topologyTest — which permutes condition labels and re-infers trajectories — to decide common vs separate topology (in the TGF-β EMT case study p = 0.38, so a common trajectory was retained). If topologies are shared, run progressionTest (default custom KS/Smirnov test, global = TRUE and lineages = TRUE) on slingshot pseudotime to detect differential progression (EMT case: p <= 2.2e-16), and fateSelectionTest (default classifier test on curve weights) for differential fate selection. Finally, fit gene-wise negative-binomial GAMs with tradeSeq fitGAM(counts, sds, conditions) and run conditionTest to call condition-specific DE genes, selecting at FDR 5% (1,993 DE genes in the EMT case), then optionally rank the DE list for fgsea enrichment. Optionally pre-screen candidate regions with imbalance_score before formal testing.

## Related tools

- **condiments** (Runs topologyTest, progressionTest, fateSelectionTest, and imbalance_score for multi-condition trajectory testing) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory on reduced dimensions and produces pseudotime and lineage weights used by condiments tests)
- **tradeSeq** (Fits gene-wise negative-binomial GAMs (fitGAM) on counts with pseudotime (sds) and conditions, and runs conditionTest for gene-level DE along lineages) — statomics.github.io/tradeSeq
- **fgsea** (Gene-set enrichment on the ranked DE gene list to recover biological adhesion/locomotion enrichment)

## Examples

```
library(condiments); topologyTest(sds, conditions); progressionTest(sds, conditions, global = TRUE, lineages = TRUE); cond_genes <- conditionTest(sds)
```

## Evaluation signals

- topologyTest p-value > 0.05 (e.g., 0.38 in the EMT case) supports retaining the common trajectory before running progression/fate tests
- progressionTest should report a well-defined p-value; a strongly significant result (p <= 2.2e-16) is expected when conditions genuinely alter pseudotime distributions
- conditionTest DE gene counts should be reproducible at FDR 5% (1,993 genes reported for the TGF-β EMT case study)
- fgsea on the ranked DE list should recover the reported adhesion and locomotion pathway enrichment
- All statistics should assemble into a single table matching the paper's Table 2 row and Fig. 3 annotations

## Limitations

- Dependence of pseudotime/weight estimates on trajectory inference (double-dipping) is ignored rather than resolved
- P-values are given only a numerical-summary status, not strong probabilistic interpretation
- No independent benchmarking of the workflow has been performed; the authors call for it, and real-world benchmarking case studies are still lacking
- Some amount of batch-condition confounding can be inherent and unresolvable in some settings

## Evidence

- [intro] In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology.: "In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions"
- [methods] The __topologyTest__ assess the quality of the common trajectory inference: "The __topologyTest__ assess the quality of the common trajectory inference"
- [methods] sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions): "sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)"
- [discussion] taking into account the dynamic nature of biological systems via the trajectory representation enables condiments to better detect true changes: "taking into account the dynamic nature of biological systems via the trajectory representation enables condiments to better detect true changes"
- [readme] BiocManager::install("condiments"): [evidence span withheld — anchor status: external_doc]
