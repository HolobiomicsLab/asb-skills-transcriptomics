---
name: fisher-exact-test-differential-gene-enrichment
description: Use when you have imputed surface protein abundances (cTP-net output)
  that split one cell type into two subgroups, and you need to test whether the genes
  differential between those subgroups are enriched, e.g. CD56-high/CD16-low vs CD56-low/CD16-high
  NK cells.
when_to_use_negative:
- Input is raw RNA abundance alone rather than imputed protein space — the CD56/CD16
  gradient is not found in the corresponding RNA abundances (F-test p = 0.9377), so
  enrichment testing there is uninformative for this skill's purpose
- Expression input is not UMI-based counts — cTP-net can only apply to count-based
  expression input (UMI-based), so TPM/RPKM-style inputs cannot produce the imputed
  abundances this enrichment test depends on
- Cells are outside the hematopoietic lineage — the article reports that imputation
  of these immune-related markers was not tried on non-hematopoietic cells, so subgroup
  definitions and enrichment results are unvalidated there
edam_operation: http://edamontology.org/operation_2239
tools:
- name: R
  role: Environment for running the Fisher exact test on differential gene sets and
    supporting Spearman/F-test/bootstrap statistics
- name: cTP-net
  role: Produces the imputed 24 surface protein abundances (CD56, CD16, CD57) whose
    imputed groups define the contingency table for the Fisher exact test
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Pre-imputation denoising of scRNA-seq counts; required to avoid a 0.02 correlation
    retraction in imputation quality
- name: Seurat
  role: LogNormalize transformation of denoised counts and preprocessing/postprocessing
    alignment via preprocess_seurat3/postprocess_seurat3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:649b176f7eeb1d1fe0698ee57634a44e0dab0d90a18772dd91b1c00cd0738a13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fisher-exact-test-differential-gene-enrichment

## Summary

Applies the Fisher exact test on genes differential between imputed-protein-defined cell subpopulations (e.g., NK cells split by the CD56/CD16 gradient from cTP-net imputation) to test whether the protein-imputed substructure corresponds to distinct transcriptomic programs.

## When to use

Use when you have imputed surface protein abundances (cTP-net output) that split one cell type into two subgroups, and you need to test whether the genes differential between those subgroups are enriched, e.g. CD56-high/CD16-low vs CD56-low/CD16-high NK cells.

## When NOT to use

- Input is raw RNA abundance alone rather than imputed protein space — the CD56/CD16 gradient is not found in the corresponding RNA abundances (F-test p = 0.9377), so enrichment testing there is uninformative for this skill's purpose
- Expression input is not UMI-based counts — cTP-net can only apply to count-based expression input (UMI-based), so TPM/RPKM-style inputs cannot produce the imputed abundances this enrichment test depends on
- Cells are outside the hematopoietic lineage — the article reports that imputation of these immune-related markers was not tried on non-hematopoietic cells, so subgroup definitions and enrichment results are unvalidated there

## Inputs

- Imputed 24-marker surface protein relative abundances per cell (postprocess_seurat3 output from cTP-net, e.g. CD56, CD16, CD57 levels)
- Cell type labels identifying NK cells in HCA CBMC/BMMC (MantonCB1, MantonBM1) data
- List of genes differential between the CD56-high/CD16-low and CD56-low/CD16-high imputed subgroups
- SAVER-X denoised, Seurat LogNormalize-normalized UMI count matrices

## Outputs

- Fisher exact test p-value for enrichment of differential genes between imputed-protein-defined NK subgroups (target: 1.07e-04)
- Contingency table of differential gene membership by subgroup
- Supporting statistics: Spearman rho of imputed CD56 vs CD16 in NK cells (-0.47) and F-test p-values for protein vs RNA space (1.667e-15 vs 0.9377)

## How to apply

First obtain the imputed 24-protein relative abundances: denoise the HCA MantonCB1/MantonBM1 scRNA-seq counts with SAVER-X (default parameters; skipping denoising costs ~0.02 in correlation), LogNormalize with Seurat, run preprocess_seurat3 with the 24-protein panel (gene_names_protein24), call cTPnet(data, data_type, model_file_path, dprotein), then postprocess_seurat3 to recover y_pred. Within NK cells, compute the Spearman correlation between imputed CD56 and CD16 (expected rho = -0.47) to confirm the opposing gradient before defining groups. Partition NK cells into CD56-high/CD16-low and CD56-low/CD16-high sets, identify genes differential between the groups, and run the Fisher exact test on the resulting gene sets in R (set options(stringsAsFactors = FALSE) first to avoid the demo dimension bug). Accept the run only if the Fisher test p-value matches the reported 1.07e-04, consistent with Fig. 3f-g and Supplementary Figs. 8-10; the paired F-test on the CD56/CD16 gradient should also separate protein space (p = 1.667e-15) from RNA space (p = 0.9377), demonstrating the effect is visible only after imputation.

## Related tools

- **R** (Environment for running the Fisher exact test on differential gene sets and supporting Spearman/F-test/bootstrap statistics)
- **cTP-net** (Produces the imputed 24 surface protein abundances (CD56, CD16, CD57) whose imputed groups define the contingency table for the Fisher exact test) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Pre-imputation denoising of scRNA-seq counts; required to avoid a 0.02 correlation retraction in imputation quality)
- **Seurat** (LogNormalize transformation of denoised counts and preprocessing/postprocessing alignment via preprocess_seurat3/postprocess_seurat3)

## Examples

```
options(stringsAsFactors=FALSE); y_pred <- cTPnet(data, data_type, model_file_path, dprotein); fisher.test(table(diff_genes, grp))  # grp = CD56high/CD16low vs CD56low/CD16high NK cells
```

## Evaluation signals

- Fisher exact test p-value on CD56-high/CD16-low vs CD56-low/CD16-high differential genes equals the reported 1.07e-04
- Spearman correlation between imputed CD56 and CD16 in NK cells equals -0.47 before groups are defined
- F-test comparing the CD56/CD16 gradient in protein vs RNA space yields p = 1.667e-15 (protein) and p = 0.9377 (RNA)
- Bootstrap test comparing imputed CD57 between CBMC and BMMC gives p < 1e-06
- Contingency table margins must sum to the filtered gene universe (genes <10 counts across cells removed; cells <200 genes detected removed)

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); prediction accuracy for TPM/RPKM-style CITE-seq data is unknown
- Missing genes present in the training data but not in the testing data are filled with zeros, which constrains which genes can enter the differential/enrichment analysis
- Generalization to cell types outside the hematopoietic lineage was not tested, so enrichment of imputed-protein subgroups outside immune cells is unvalidated
- On Windows, cTPnet may fail with ModuleNotFoundError: No module named 'ctpnet'; requires numpy 1.19.3, and Windows is not recommended for running cTP-net

## Evidence

- [discussion] low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [discussion] Genes that are in the training data but not in the testing data are filled with zeros.: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] cTP-net can only apply to count-based expression input (UMI-based).: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm.: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
