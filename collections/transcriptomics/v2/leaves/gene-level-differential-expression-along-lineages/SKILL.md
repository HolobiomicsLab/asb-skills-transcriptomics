---
name: gene-level-differential-expression-along-lineages
description: Use when a common multi-condition trajectory (slingshot sds) already
  exists and the goal is to find genes whose expression patterns differ between conditions
  along its lineages after topology, progression, and fate-selection tests.
when_to_use_negative:
- The topologyTest or trajectory-level tests have not yet been run and differential
  topology has not been assessed — condiments prescribes ordering DE after the trajectory-level
  steps
- The data are not single-cell count data ordered along a trajectory with known condition
  labels (e.g., already aggregated bulk expression tables)
- The question is differential abundance of cell populations rather than gene-level
  expression change along lineages
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tradeSeq
  role: Fits gene-wise negative-binomial GAMs (fitGAM) on the sds trajectory with
    condition labels and runs conditionTest to call condition-specific DE genes
  repo: http://statomics.github.io/tradeSeq
- name: condiments
  role: Provides the three-step workflow context and toy_dataset used to demonstrate
    the DE step
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory whose lineages and pseudotime feed fitGAM as
    the sds input
- name: fgsea
  role: Gene-set enrichment on the ranked DE gene list to recover biological themes
    (adhesion and locomotion enrichment in the EMT case study)
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:b151bc0df9db75dc86fcc9014cd317d69b31237c0aa4c40b2bb7c979308e86c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - fgsea
  - ggplot2
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Gene-level differential expression along lineages

## Summary

Step 3 of the condiments workflow: fit gene-wise negative-binomial GAMs with tradeSeq fitGAM on a slingshot trajectory under multiple conditions and call condition-specific expression changes with conditionTest, selecting genes at FDR 5% (e.g., 1,993 DE genes in the TGF-β EMT case study).

## When to use

Use when a common multi-condition trajectory (slingshot sds) already exists and the goal is to find genes whose expression patterns differ between conditions along its lineages after topology, progression, and fate-selection tests.

## When NOT to use

- The topologyTest or trajectory-level tests have not yet been run and differential topology has not been assessed — condiments prescribes ordering DE after the trajectory-level steps
- The data are not single-cell count data ordered along a trajectory with known condition labels (e.g., already aggregated bulk expression tables)
- The question is differential abundance of cell populations rather than gene-level expression change along lineages

## Inputs

- Gene count matrix (integrated and normalized across conditions, e.g., EMT count matrix)
- slingshot trajectory object (sds) with lineages and pseudotime
- Per-cell condition labels (e.g., mock vs TGF-beta, control vs ILD)

## Outputs

- Table of per-gene conditionTest statistics and adjusted p-values
- Set of condition-specific DE genes called at FDR 5%
- Ranked DE gene list for downstream fgsea gene-set enrichment (e.g., condiments_emt_results.csv)

## How to apply

After running topologyTest, progressionTest, and fateSelectionTest on the integrated slingshot trajectory, fit gene-wise negative-binomial generalized additive models with tradeSeq fitGAM, supplying the count matrix, the sds object, and per-cell condition labels as inputs. Then run tradeSeq conditionTest on the fitted GAMs to test whether gene expression patterns differ between conditions along each lineage. Select significant genes at FDR 5% (Benjamini-Hochberg adjusted p-values). Optionally rank the DE gene list and run fgsea gene-set enrichment to interpret biological themes, as done in the EMT case study (1,993 DE genes; adhesion and locomotion enrichment) and the fibrosis ILD case study, where patternTest serves as a secondary lineage-level DE check.

## Related tools

- **tradeSeq** (Fits gene-wise negative-binomial GAMs (fitGAM) on the sds trajectory with condition labels and runs conditionTest to call condition-specific DE genes) — http://statomics.github.io/tradeSeq
- **condiments** (Provides the three-step workflow context and toy_dataset used to demonstrate the DE step) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory whose lineages and pseudotime feed fitGAM as the sds input)
- **fgsea** (Gene-set enrichment on the ranked DE gene list to recover biological themes (adhesion and locomotion enrichment in the EMT case study))

## Examples

```
sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions); cond_genes <- conditionTest(sds)
```

## Evaluation signals

- fitGAM converged on all genes without error and sds/counts/conditions inputs are dimensionally consistent (one cell per count column and label)
- conditionTest output contains one row per gene with test statistics and FDR-adjusted p-values
- Number of genes selected at FDR 5% matches expectations, e.g., 1,993 DE genes in the TGF-β EMT case study
- fgsea on the ranked DE list recovers the reported biological adhesion and locomotion enrichment
- All test statistics and gene counts assemble into a single table matching Table 2 row 1 and Fig. 3 annotations (e.g., condiments_emt_results.csv)

## Limitations

- The article notes that no independent benchmarking of the workflow has been performed and that real-world benchmarking case studies are still lacking
- Dependence of pseudotime/weight estimates on the trajectory inference (double-dipping) is ignored rather than resolved
- P-values are not given a strong probabilistic interpretation, only numerical-summary status

## Evidence

- [intro] Lastly, in Step 3, it estimates gene expression profiles similarly to Van den Berge et al.8 and tests whether gene expression patterns differ between conditions along lineages: "Lastly, in Step 3, it estimates gene expression profiles similarly to Van den Berge et al.8 and tests whether gene expression patterns differ between conditions along lineages"
- [methods] sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions): "sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)"
- [methods] cond_genes <- conditionTest(sds): "cond_genes <- conditionTest(sds)"
- [discussion] and finally gene-level differences in expression with the conditionTest.: "and finally gene-level differences in expression with the conditionTest."
- [abstract] finding genes that exhibit different behaviors between these conditions along a differentiation path: "finding genes that exhibit different behaviors between these conditions along a differentiation path"
