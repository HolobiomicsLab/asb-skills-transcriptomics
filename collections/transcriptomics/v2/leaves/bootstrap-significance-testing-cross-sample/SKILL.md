---
name: bootstrap-significance-testing-cross-sample
description: Use when comparing the distribution of cTP-net-imputed surface protein
  abundances between two independent scRNA-seq samples (e.g., cord blood vs bone marrow
  mononuclear cells) to test whether imputed protein levels differ biologically across
  samples.
when_to_use_negative:
- Input is raw RNA abundance instead of cTP-net-imputed protein levels — the paper
  shows RNA fails to recover these cross-sample/subpopulation signals (F-test p =
  0.9377 in RNA space)
- Expression data are not UMI-based counts — the article states cTP-net can only apply
  to count-based (UMI) expression input, so imputed protein levels would be unavailable
- The two groups are cell types within one sample rather than independent samples
  — cell-type-level subpopulation contrasts (e.g., CD56bright vs CD56dim) use Spearman
  correlation and Fisher exact tests, not the cross-sample bootstrap
tools:
- name: cTP-net
  role: Imputes the 24 surface protein relative abundances (including CD57) whose
    cross-sample differences the bootstrap test evaluates
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq counts with default parameters before imputation; skipping
    it retracts prediction correlation by 0.02
- name: Seurat
  role: LogNormalize transformation of denoised counts and seurat3-format pre/post-processing
    (preprocess_seurat3 / postprocess_seurat3) around the cTPnet call
- name: R
  role: Implementation environment for the cTPnet R package and the bootstrap significance
    testing
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:1c1dab968f842c795a3039502e778d596d9a55a84b323852faee1faa40e9ae09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bootstrap-significance-testing-cross-sample

## Summary

Bootstrap-based significance testing to compare cTP-net-imputed surface protein levels (e.g., CD57) between independent scRNA-seq samples (HCA-CBMC vs HCA-BMMC), establishing that imputed protein abundances reveal cross-sample biological differences beyond transcriptome measurements.

## When to use

Use when comparing the distribution of cTP-net-imputed surface protein abundances between two independent scRNA-seq samples (e.g., cord blood vs bone marrow mononuclear cells) to test whether imputed protein levels differ biologically across samples.

## When NOT to use

- Input is raw RNA abundance instead of cTP-net-imputed protein levels — the paper shows RNA fails to recover these cross-sample/subpopulation signals (F-test p = 0.9377 in RNA space)
- Expression data are not UMI-based counts — the article states cTP-net can only apply to count-based (UMI) expression input, so imputed protein levels would be unavailable
- The two groups are cell types within one sample rather than independent samples — cell-type-level subpopulation contrasts (e.g., CD56bright vs CD56dim) use Spearman correlation and Fisher exact tests, not the cross-sample bootstrap

## Inputs

- cTP-net-imputed 24-protein relative abundance matrix (postprocess_seurat3 output) for HCA-CBMC (MantonCB1) cells
- cTP-net-imputed 24-protein relative abundance matrix for HCA-BMMC (MantonBM1) cells
- Cell type / subpopulation annotations (e.g., NK cells for CD56/CD16 analyses; all cells for CD57)

## Outputs

- Bootstrap p-value comparing imputed CD57 levels between CBMC and BMMC (expected p < 1e-06)
- Per-sample bootstrap distribution of the mean/median imputed protein level
- Comparison table of protein-space vs RNA-space significance statistics (F-test p = 1.667e-15 vs 0.9377; Fisher p = 1.07e-04)

## How to apply

After denoising HCA MantonCB1 and MantonBM1 count matrices with SAVER-X, LogNormalize with Seurat, and running the cTP-net 24-protein model (preprocess_seurat3, cTPnet, postprocess_seurat3), extract the imputed relative abundance of the target protein (CD57) for every cell in each sample. Resample cells with replacement within each sample to build a bootstrap distribution of the CD57 level difference between CBMC and BMMC, and derive a bootstrap p-value for the observed difference. The expected outcome for imputed CD57 between CBMC and BMMC is p < 1e-06, supporting that imputation recovers sample-specific protein biology (Fig. 3f-g, Supplementary Figs. 8-10). Run this alongside the Spearman CD56/CD16 gradient test (rho = -0.47), the F-test (p = 1.667e-15 protein vs 0.9377 RNA), and the Fisher exact test (p = 1.07e-04) as a battery of imputation-validation statistics; set options(stringsAsFactors = FALSE) in R before running.

## Related tools

- **cTP-net** (Imputes the 24 surface protein relative abundances (including CD57) whose cross-sample differences the bootstrap test evaluates) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq counts with default parameters before imputation; skipping it retracts prediction correlation by 0.02)
- **Seurat** (LogNormalize transformation of denoised counts and seurat3-format pre/post-processing (preprocess_seurat3 / postprocess_seurat3) around the cTPnet call)
- **R** (Implementation environment for the cTPnet R package and the bootstrap significance testing) — https://github.com/zhouzilu/cTPnet

## Examples

```
options(stringsAsFactors=FALSE); y_pred <- postprocess_seurat3(seurat_data, cTPnet(preprocess_seurat3(seurat_data, dprotein=gene_names_protein24), data_type, model_file_path, dprotein=gene_names_protein24))
```

## Evaluation signals

- Bootstrap p-value for imputed CD57 (CBMC vs BMMC) is < 1e-06, consistent with Fig. 3f-g and Supplementary Figs. 8-10
- Companion F-test comparing CD56/CD16 gradient in protein vs RNA space yields p = 1.667e-15 (protein) vs 0.9377 (RNA)
- Companion Spearman rho = -0.47 for imputed CD56 vs CD16 within NK cells, a pattern absent in RNA abundances
- Companion Fisher exact test on CD56-high/CD16-low vs CD56-low/CD16-high differential genes gives p = 1.07e-04
- Pipeline invariants hold: low-quality genes (<10 counts across cells) and cells (<200 genes detected) removed; options(stringsAsFactors = FALSE) set to avoid the demo dimension error

## Limitations

- cTP-net only applies to UMI-based count expression input, so the bootstrap must operate on imputed values from UMI scRNA-seq data
- Generalization to cell types outside the hematopoietic lineage was not tested, bounding the biological scope of cross-sample conclusions
- SAVER-X denoising omission retracts prediction correlation by 0.02 — more than any other parameter tweak — degrading the imputed levels the bootstrap compares

## Evidence

- [methods] run the bootstrap test comparing imputed CD57 levels between CBMC and BMMC: [evidence span withheld — anchor status: not_found]
- [results] bootstrap p < 1e-06 for imputed CD57 (CBMC vs BMMC): [evidence span withheld — anchor status: not_found]
- [discussion] without SAVER-X denoising performance retracts: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] low-quality gene/cell filtering thresholds: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [discussion] UMI-only input restriction: "cTP-net can only apply to count-based expression input (UMI-based)."
