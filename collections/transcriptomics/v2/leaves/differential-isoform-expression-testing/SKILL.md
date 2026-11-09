---
name: differential-isoform-expression-testing
description: Use when you have per-gene isoform (or TSS/polyA-site) count tables for
  two cell types or regions and need to identify genes whose percent-isoform usage
  shifts significantly between them.
when_to_use_negative:
- The input is short-read gene-level expression data without isoform-resolved counts
  — a standard differential gene expression test, not the chi-square isoform-proportion
  test, is appropriate
- You need exon-level usage changes rather than full-isoform proportion shifts — ExonQuant
  is the alternative, though isoform tests perform better for DIE
- Comparing more than two conditions at once — DiffSplicingAnalysis as used here requires
  pairwise comparisons via config rows (e.g. RGL->NIPC, NIPC->GranuleNB, GranuleNB->EN)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: scisorseqr
  role: Provides DiffSplicingAnalysis for per-gene n x 2 chi-square tests on isoform/TSS/polyA
    proportions between two conditions
  repo: https://github.com/noush-joglekar/scisorseqr
- name: R
  role: Runtime environment for scisorseqr and post-hoc FDR filtering, delta-PI computation,
    and classification tables (R/pandas)
- name: IsoQuant
  role: Sister package producing the per-cell-type isoform/TSS count tables that serve
    as DiffSplicingAnalysis input
  repo: https://github.com/ablab/isoquant
- name: pandas
  role: Python library for building config files and writing DIE/model-classification
    CSV outputs
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:36fb3a6a89789eee3eda8bffe0fa64bb1356e6081451cfc7f029abca33d3cdd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - DiffSplicingAnalysis
  - IsoQuant
  - Python
  - R
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-isoform-expression-testing

## Summary

Gene-wise differential isoform expression (DIE) testing between two biological conditions (e.g. mouse hippocampus vs prefrontal cortex) using scisorseqr's per-gene n x 2 chi-square tests on isoform (and optionally TSS/polyA-site) proportions, with Benjamini-Hochberg FDR <= 0.05 and delta-PI >= 0.1 cutoffs.

## When to use

Use when you have per-gene isoform (or TSS/polyA-site) count tables for two cell types or regions and need to identify genes whose percent-isoform usage shifts significantly between them.

## When NOT to use

- The input is short-read gene-level expression data without isoform-resolved counts — a standard differential gene expression test, not the chi-square isoform-proportion test, is appropriate
- You need exon-level usage changes rather than full-isoform proportion shifts — ExonQuant is the alternative, though isoform tests perform better for DIE
- Comparing more than two conditions at once — DiffSplicingAnalysis as used here requires pairwise comparisons via config rows (e.g. RGL->NIPC, NIPC->GranuleNB, GranuleNB->EN)

## Inputs

- Per-gene isoform count table (e.g. bulk HIPP vs PFC from www.isoformAtlas.com) with rows as genes and columns as isoform counts per condition
- Per-gene TSS and polyA-site count tables for feature-attribution tests
- Tab-separated config file in Comparison1/Celltypes/Comparison2/Celltypes format naming the two conditions per comparison

## Outputs

- Results table (TSV) with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- List of significant DIE genes (e.g. 395 of 3958 tested for HIPP vs PFC) with FDR <= 0.05 and dPI >= 0.1
- Feature-attribution flags (e.g. 141 genes with differential TSS or polyA-site usage)
- Per-comparison sub-directories under TreeTraversal_Iso for downstream triHeatmap/sigSplitPie visualization

## How to apply

Load the deposited per-gene isoform count tables (e.g. bulk HIPP vs bulk PFC from www.isoformAtlas.com) and build a tab-separated config file with one row per comparison. Run scisorseqr's DiffSplicingAnalysis with typeOfTest='Iso', minNumReads=25, and is.hier=FALSE, which performs per-gene n x 2 chi-square tests on isoform proportions between conditions; apply the same framework with typeOfTest='TSS' or 'PolyA' to attribute DIE to transcript-feature usage. Filter results (columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR) using Benjamini-Hochberg FDR <= 0.05 and require delta-PI >= 0.1 change in percent-isoform usage. Rationale: isoform-level tests outperform exon-level tests (ExonQuant) for detecting DIE, and the minNumReads=25 floor guards against sparse long-read counts. For cell-type attribution of bulk DIE, compute delta-PI(subtype)/delta-PI(bulk) ratios to classify genes as Single-, Both-, or No-Cell-Type-Model.

## Related tools

- **scisorseqr** (Provides DiffSplicingAnalysis for per-gene n x 2 chi-square tests on isoform/TSS/polyA proportions between two conditions) — https://github.com/noush-joglekar/scisorseqr
- **R** (Runtime environment for scisorseqr and post-hoc FDR filtering, delta-PI computation, and classification tables (R/pandas))
- **IsoQuant** (Sister package producing the per-cell-type isoform/TSS count tables that serve as DiffSplicingAnalysis input) — https://github.com/ablab/isoquant
- **pandas** (Python library for building config files and writing DIE/model-classification CSV outputs)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- Bulk HIPP-vs-PFC reproduction must yield 395 significant genes out of 3958 tested (9.06%) at FDR <= 0.05, delta-PI >= 0.1
- 141 of the 395 DIE genes should be flagged for differential TSS or polyA-site usage under the same chi-square framework
- Cell-type tracing of DIE genes should reproduce 151/81/26 Single-/Both-/No-Cell-Type-Model genes (75% ± 2.3 SE, 24.4%, 0.3%)
- Trajectory transitions should yield 73 (RGL->NIPC, 5.1%), 359 (NIPC->GranuleNB), and 423 (GranuleNB->EN) significant genes
- Every results TSV must contain the six required columns: Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR

## Limitations

- Long-read isoform matrices are sparse due to lower long-read throughput, which can cause clustering and testing discrepancies
- minNumReads=25 and delta-PI >= 0.1 thresholds filter genes with low isoform coverage; rare isoforms may go untested
- Generalizability of DIE patterns to disease case-control studies, adult brain regions, and other species is untested
- Traditional transcriptional cell-type definitions may not correspond to isoform expression programs

## Evidence

- [methods] DiffSplicingAnalysis parameters and test type: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [discussion] Published DIE count for HIPP vs PFC: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [readme] scisorseqr purpose from README: [evidence span withheld — anchor status: external_doc]
- [abstract] Isoform tests outperform exon tests: "Isoform tests for DIE show better performance than exon tests."
- [abstract] Cell-type attribution of regional DIE: "Mostly, one cell type is responsible for brain-region specific DIE. However, for fewer genes, multiple cell types influence DIE."
