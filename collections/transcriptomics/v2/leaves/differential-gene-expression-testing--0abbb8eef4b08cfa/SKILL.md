---
name: differential-gene-expression-testing
description: Use when a merged, quantile-normalized (preprocessCore 'normalize.quantiles')
  expression matrix of staged cells must be reduced to a gene set that captures developmental
  variation before trajectory/pseudotime analysis.
when_to_use_negative:
- Input is already a curated gene set or the downstream step needs per-cluster markers
  — use Seurat FindAllMarkers with p_val_adj < 0.05 (as in the TimeTalk TRN/master-regulator
  step) instead of the trajectory-wide differentialGeneTest
- The goal is cell-type-resolved ligand-receptor communication between defined cell
  types (e.g. CellPhoneDB/CellChat settings) rather than ordering cells along a developmental
  pseudotime
- Cells have no stage/temporal labels to test against, so the differential test cannot
  discriminate developmental progression
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: monocle2
  role: Provides the 'differentialGeneTest' function applied with cutoff qval = 0.01
    to select DEGs for DDRTree trajectory reconstruction
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: R
  role: Execution environment (R 3.6.3) in which the CellDataSet is built and the
    differential gene test is run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:f275dd070d1f9f0e9a68ff76cc9eddf8e0f82e917e9fa691a95ce5fb5fabfbbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cole-trapnell-lab/monocle3
  tools:
  - R
  - TimeTalk
  - monocle2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# differential-gene-expression-testing

## Summary

Selects developmentally informative genes from the quantile-normalized pre-implantation scRNA-seq matrix by running monocle2's 'differentialGeneTest' with a qval = 0.01 cutoff on a CellDataSet of 262 early-embryo cells. The resulting differentially expressed genes are the input gene set for DDRTree trajectory reconstruction.

## When to use

Use when a merged, quantile-normalized (preprocessCore 'normalize.quantiles') expression matrix of staged cells must be reduced to a gene set that captures developmental variation before trajectory/pseudotime analysis.

## When NOT to use

- Input is already a curated gene set or the downstream step needs per-cluster markers — use Seurat FindAllMarkers with p_val_adj < 0.05 (as in the TimeTalk TRN/master-regulator step) instead of the trajectory-wide differentialGeneTest
- The goal is cell-type-resolved ligand-receptor communication between defined cell types (e.g. CellPhoneDB/CellChat settings) rather than ordering cells along a developmental pseudotime
- Cells have no stage/temporal labels to test against, so the differential test cannot discriminate developmental progression

## Inputs

- Merged, quantile-normalized expression matrix (normalize.quantiles from preprocessCore v1.48.0) for 262 pre-implantation cells with stage labels
- monocle2 CellDataSet object built from that matrix

## Outputs

- Table of differential expression test results with per-gene qval
- Filtered set of differentially expressed genes (qval = 0.01 cutoff) used as input to monocle2 'reduceDimension' DDRTree trajectory reconstruction

## How to apply

Build a monocle2 CellDataSet object from the merged expression matrix for the 262 pre-implantation cells (oocyte through late blastocyst). Run monocle2 'differentialGeneTest' across the dataset and retain genes with qval < 0.01 as the differentially expressed gene set. This filtering step is the decision point: only DEGs passing this cutoff are carried forward to DDRTree dimension reduction via 'reduceDimension', which is the article's stated route to reconstructing the development trajectory prior to 'orderCells' pseudotime ordering. The rationale is that testing every gene would dilute the trajectory signal; restricting to qval = 0.01 DEGs is what allowed the ordering to reproduce the stage arrangement of Fig. 1e. Note this is a whole-dataset developmental DE test, distinct from Seurat's per-cluster FindAllMarkers (p_val_adj < 0.05) used elsewhere in the TimeTalk workflow for cell-type marker discovery.

## Related tools

- **monocle2** (Provides the 'differentialGeneTest' function applied with cutoff qval = 0.01 to select DEGs for DDRTree trajectory reconstruction) — https://github.com/cole-trapnell-lab/monocle3
- **R** (Execution environment (R 3.6.3) in which the CellDataSet is built and the differential gene test is run)

## Examples

```
diff_test_res <- differentialGeneTest(cds, fullModelFormulaStr = "~Stage"); sig_genes <- rownames(subset(diff_test_res, qval < 0.01))
```

## Evaluation signals

- Every reported DEG has qval ≤ 0.01 (the stated monocle2 'differentialGeneTest' cutoff)
- The DEG set is computed on all 262 pre-implantation cells — no cells are dropped at this step
- Downstream DDRTree 'reduceDimension' run on the DEG set must yield a trajectory containing all 262 cells ordered oocyte → late blastocyst
- The resulting pseudotime layout matches the developmental-stage arrangement of Fig. 1e
- The DEG test is run on the quantile-normalized matrix (preprocessCore 1.48.0), not the raw counts

## Limitations

- The qval = 0.01 cutoff is dataset-specific to the 262-cell pre-implantation matrix and was not benchmarked against alternative thresholds in the article
- DEG-based gene selection is a prerequisite for the DDRTree trajectory but does not itself establish temporal ordering — pseudotime assignment requires the subsequent 'orderCells' step
- TimeTalk's downstream causal conclusions are parameter-sensitive (e.g. Granger test affected by 'winsz'), so users are advised to test multiple parameter combinations; DEG filtering choices may similarly warrant sensitivity checks

## Evidence

- [discussion] monocle2 differential gene test cutoff: "we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01)"
- [discussion] DEGs feed DDRTree trajectory reconstruction: "the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function."
- [discussion] input matrix is quantile-normalized: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [readme] contrast with per-cluster marker testing in the same codebase: "tmp.all.de.RNA <- FindAllMarkers(object = seu,
                                 assay = "RNA",
                                 only.pos = T)"
