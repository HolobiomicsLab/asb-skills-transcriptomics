---
name: multi-sample-scrna-seq-differential-pseudotime-analysis
description: Use when you have scRNA-seq data from multiple biological samples with
  covariates (e.g. sex, disease status) and a reconstructed pseudotime trajectory,
  and you need to test which genes or density patterns change along pseudotime in
  association with those covariates.
when_to_use_negative:
- You have only one biological sample per condition and cannot estimate sample-level
  variance — Lamian's FDR control relies on cross-sample variability (as with condiments'
  limitation).
- You need topology/differential-trajectory modules compatible with Monocle2, Monocle3,
  or Slingshot trajectories — Lamian modules 1 and 2 do not support these.
- Your goal is a systematic comparison of sample-harmonization methods, which is explicitly
  beyond the paper's scope.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: 'Core framework: runs lamian_test() for TDE (''time'') and XDE (''variable'')
    tests using permutation-based likelihood ratio tests'
  repo: https://github.com/Winnie09/Lamian
- name: Slingshot
  role: Used upstream to infer pseudotime trajectories for each lineage within samples
- name: TSCAN
  role: Default method in Lamian to construct pseudotime via cluster-based minimum
    spanning tree
  repo: https://github.com/Winnie09/Lamian
- name: scRNA-seq
  role: 'Data modality: single-cell RNA-sequencing count matrices from multiple donors/samples'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:0008b5c6e057fe05bfac1809dec84e437b129e4debfd0d78879a0744b18a2c63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Slingshot
  - TSCAN
  - scRNA-seq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-sample-scrna-seq-differential-pseudotime-analysis

## Summary

A statistically-rigorous framework (Lamian) for detecting differences in gene expression (TDE/XDE), cell density (TCD/XCD), and trajectory topology along pseudotime across multiple scRNA-seq samples, accounting for cross-sample variability to control FDR.

## When to use

Use when you have scRNA-seq data from multiple biological samples with covariates (e.g. sex, disease status) and a reconstructed pseudotime trajectory, and you need to test which genes or density patterns change along pseudotime in association with those covariates.

## When NOT to use

- You have only one biological sample per condition and cannot estimate sample-level variance — Lamian's FDR control relies on cross-sample variability (as with condiments' limitation).
- You need topology/differential-trajectory modules compatible with Monocle2, Monocle3, or Slingshot trajectories — Lamian modules 1 and 2 do not support these.
- Your goal is a systematic comparison of sample-harmonization methods, which is explicitly beyond the paper's scope.

## Inputs

- Gene expression count matrix (expr) from multi-sample scRNA-seq data
- Cell-to-sample annotation table (cellanno)
- Per-cell pseudotime values from Slingshot or TSCAN trajectory inference
- Design matrix with sample covariates (e.g. sex, disease status), first column as intercept
- Example Lamian data objects: hca_bm_saver, hca_bm_pca, hca_bm_cellanno, slingshot_data, expdata

## Outputs

- Per-lineage statistics table (Res$statistics) with fdr.overall columns for TDE and XDE genes
- Counts of significant TDE and XDE genes per lineage at FDR < 0.05 (e.g. tde_xde_lineage_results.csv)
- Lists of significant genes for downstream enrichment analysis (e.g. sex-chromosome enrichment)

## How to apply

Preprocess pseudotime by dropping cells with NA pseudotime and excluding samples with fewer than 50 cells to ensure signal-to-noise. For each lineage (e.g. myeloid, erythroid, lymphoid), run lamian_test() with test.type='time' using expr, cellanno, pseudotime, and design (first design column as intercept; permuiter=100) to detect TDE genes. Run lamian_test() with test.type='variable' and testvar=2 (permutation-based likelihood ratio test) to detect XDE genes whose dynamics depend on sample covariates. Call genes significant at fdr.overall < 0.05 from Res$statistics, count them per lineage, and validate against known markers (CD14, HBB, GATA1, CD3D, CD19, CD27) and reported counts (TDE 8475/7454/8953; XDE 43/32/29 in the HCA-BM analysis).

## Related tools

- **Lamian** (Core framework: runs lamian_test() for TDE ('time') and XDE ('variable') tests using permutation-based likelihood ratio tests) — https://github.com/Winnie09/Lamian
- **Slingshot** (Used upstream to infer pseudotime trajectories for each lineage within samples)
- **TSCAN** (Default method in Lamian to construct pseudotime via cluster-based minimum spanning tree) — https://github.com/Winnie09/Lamian
- **scRNA-seq** (Data modality: single-cell RNA-sequencing count matrices from multiple donors/samples)

## Examples

```
library(Lamian); Res <- lamian_test(expr=expr, cellanno=cellanno, pseudotime=pseudotime, design=design, test.type='variable', testvar=2, permuiter=100)
```

## Evaluation signals

- Reproduce reported per-lineage significance counts at FDR < 0.05: TDE 8475/7454/8953 and XDE 43/32/29 for myeloid/erythroid/lymphoid in the HCA-BM dataset.
- Confirm recovery of known lineage markers among significant genes: CD14 (myeloid), HBB/GATA1 (erythroid), CD3D/CD19/CD27 (lymphoid).
- Check that ~33% (N=14) of myeloid XDE genes are significantly enriched on sex chromosomes when testing male-female differences.
- Verify design matrix first column is an intercept and permuiter is set (recommended 100) so permutation-based LRT is reproducible.
- Confirm samples with fewer than 50 cells and cells with NA pseudotime were excluded before testing.

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories; interoperability tailoring is deferred to future work.
- A systematic comparison of harmonization methods is beyond the scope of the study.
- Simultaneously accounting for pseudotime inference uncertainty, expression/abundance uncertainty, and cross-sample variability remains a future research topic.
- Simulations evaluating TDE, TCD, and XDE detection are only in Supplementary Notes.

## Evidence

- [abstract] Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects: "Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects"
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] The raw count matrix of bone marrow scRNA-seq data sequenced in 10x Genomics platform from 8 healthy donors were downloaded: "The raw count matrix of bone marrow scRNA-seq data sequenced in 10x Genomics platform from 8 healthy donors were downloaded from the Human Cell Atlas (HCA) data portal"
- [methods] here we use example data hca_bm_saver, hca_bm_pca, hca_bm_cellanno to demonstrate their data structures.: "Here, we use example data **hca_bm_saver**, **hca_bm_pca**, **hca_bm_cellanno** to demonstrate their data structures."
- [readme] devtools::install_github("Winnie09/Lamian"): [evidence span withheld — anchor status: external_doc]
