---
name: gene-set-enrichment-analysis
description: Use when you have a ranked gene list (e.g. FDR-ranked conditionTest DE
  genes from a multi-condition trajectory analysis) and need to interpret which biological
  pathways explain between-condition differences.
when_to_use_negative:
- Input is a raw count matrix with no prior gene-level differential expression ranking
  — run tradeSeq conditionTest first to obtain the ranked DE list
- 'The system is steady-state rather than a dynamic trajectory: cluster-based differential
  abundance methods (milo, DAseq) rather than trajectory-informed DE plus enrichment
  are the appropriate comparison frame'
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Provides the multi-condition trajectory framework (topologyTest, progressionTest,
    fateSelectionTest) whose findings the enrichment results are interpreted against
  repo: github.com/HectorRDB/condiments
- name: tradeSeq
  role: Fits gene-wise negative-binomial GAMs (fitGAM) and runs conditionTest to produce
    the ranked condition-specific DE gene list fed to fgsea
  repo: statomics.github.io/tradeSeq
- name: slingshot
  role: Infers the common EMT trajectory and pseudotime on reduced dimensions that
    tradeSeq's fitGAM requires as input (sds object)
- name: fgsea
  role: Performs gene-set enrichment on the ranked DE gene list to recover biological
    pathway-level interpretation
- name: ggplot2
  role: Used to visualize results matching the paper's Fig. 3 annotations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:3d50b819b9e3881849e9466c823d2bd7275c50058eaf63c365a542425df31f87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - fgsea
  - ggplot2
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-set-enrichment-analysis

## Summary

Run gene-set enrichment (fgsea-style) on the ranked list of condition-specific differentially expressed genes produced by tradeSeq's conditionTest to recover biological processes (e.g. adhesion and locomotion in the TGF-β EMT case study) underlying trajectory-level condition differences.

## When to use

Use when you have a ranked gene list (e.g. FDR-ranked conditionTest DE genes from a multi-condition trajectory analysis) and need to interpret which biological pathways explain between-condition differences.

## When NOT to use

- Input is a raw count matrix with no prior gene-level differential expression ranking — run tradeSeq conditionTest first to obtain the ranked DE list
- The system is steady-state rather than a dynamic trajectory: cluster-based differential abundance methods (milo, DAseq) rather than trajectory-informed DE plus enrichment are the appropriate comparison frame

## Inputs

- Ranked list of genes from tradeSeq conditionTest (e.g. FDR 5% significant genes with test statistics; 1,993 DE genes in the TGF-β EMT case study)
- Gene-set / pathway database compatible with fgsea (e.g. MSigDB Hallmark or GO term sets, mapped to gene identifiers)
- Optional: condiments_emt_results.csv-style table of test statistics for assembling final results

## Outputs

- fgsea enrichment table of gene sets with NES, p-value, and adjusted p-value (e.g. adhesion and locomotion terms enriched in the EMT setting)
- Assembled results table matching Table 2 row 1 and Fig. 3 annotations (condiments_emt_results.csv)

## How to apply

After fitting gene-wise negative-binomial GAMs with tradeSeq fitGAM on the integrated count matrix (counts, slingshot sds, condition labels) and calling condition-specific DE genes with conditionTest at FDR 5% (the TGF-β EMT case study reported 1,993 DE genes), rank the genes by their conditionTest statistic and run fgsea gene-set enrichment on this ranked list. The enrichment step interprets which pathways drive the differential progression detected by condiments' progressionTest (p <= 2.2e-16 between mock and TGF-β along the EMT trajectory). Judge correctness by whether the enrichment recovers biologically coherent terms expected for the system — in the EMT case study, adhesion and locomotion enrichment — and by matching the reported DE-gene count and statistics against the paper's Table 2 and Fig. 3 annotations.

## Related tools

- **condiments** (Provides the multi-condition trajectory framework (topologyTest, progressionTest, fateSelectionTest) whose findings the enrichment results are interpreted against) — github.com/HectorRDB/condiments
- **tradeSeq** (Fits gene-wise negative-binomial GAMs (fitGAM) and runs conditionTest to produce the ranked condition-specific DE gene list fed to fgsea) — statomics.github.io/tradeSeq
- **slingshot** (Infers the common EMT trajectory and pseudotime on reduced dimensions that tradeSeq's fitGAM requires as input (sds object))
- **fgsea** (Performs gene-set enrichment on the ranked DE gene list to recover biological pathway-level interpretation)
- **ggplot2** (Used to visualize results matching the paper's Fig. 3 annotations)

## Examples

```
library(tradeSeq); sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions); cond_genes <- conditionTest(sds); fgseaRes <- fgsea(pathways, stats = rank_from_conditionTest(cond_genes))
```

## Evaluation signals

- The number of DE genes at FDR 5% matches the reference: 1,993 condition-specific DE genes for the TGF-β EMT case study
- fgsea recovers adhesion and locomotion enrichment among top-ranked gene sets, matching the paper's reported biological interpretation
- Upstream progressionTest yields p <= 2.2e-16 (mock vs TGF-β) and topologyTest yields p = 0.38 (fail to reject common trajectory), confirming the enrichment is run on a shared-topology, differential-progression setting
- Final assembled statistics reproduce Table 2 row 1 and Fig. 3 annotations in condiments_emt_results.csv

## Limitations

- No independent benchmarking of the workflow has been performed; the authors explicitly call for independent benchmarking of the condiments pipeline
- P-values from the trajectory tests are not given a strong probabilistic interpretation, only numerical-summary status, which indirectly bounds how confidently enrichment results can be interpreted
- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved, so downstream DE and enrichment statistics inherit this approximation

## Evidence

- [methods] Fit gene-wise negative-binomial GAMs with tradeSeq fitGAM and run conditionTest to call condition-specific DE genes; select genes at FDR 5% (reported: 1,993 DE genes).: "sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)"
- [intro] Step 3 of the condiments workflow estimates gene expression profiles and tests whether gene expression patterns differ between conditions along lineages.: "Lastly, in Step 3, it estimates gene expression profiles similarly to Van den Berge et al.8 and tests whether gene expression patterns differ between conditions along lineages"
- [abstract] The method can detect subtler condition-specific changes by finding genes with different behaviors between conditions along a differentiation path.: "finding genes that exhibit different behaviors between these conditions along a differentiation path"
- [readme] The condiments package is installed from Bioconductor and documented via vignettes and case-study articles.: [evidence span withheld — anchor status: external_doc]
- [discussion] The authors highlight the need for independent benchmarking of the workflow.: "The two issues raised in the previous paragraphs highlight the need for independent benchmarking."
