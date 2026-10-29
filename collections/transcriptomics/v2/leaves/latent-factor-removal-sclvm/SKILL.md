---
name: latent-factor-removal-sclvm
description: Reference for removing dominant latent factors (e.g. cell cycle) from
  log-transformed, gene-centered scRNA-Seq matrices via scLVM's Python script, choosing
  K = 1 or the scree-plot value, and comparing corrected vs. original data.
when_to_use_negative:
- When the leading factor's identity must be statistically verified before removal
  — scLVM 'does not inspect whether a leading factor represents the cell cycle' and
  risks removing biological components of interest such as cell-type signal (K = 1
  on simulated data removed the cell-type effect).
- When the goal is to remove only the cell-cycle effect while preserving cell-type
  separation and the data contain strong competing latent structure — ccRemover is
  the article's purpose-built alternative with a formal bootstrap t-test on the loading
  difference Δj.
- When a versioned, packaged R workflow is required — the article applies scLVM via
  a Python script from its GitHub page with no version number or explicit repository
  URL recorded.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scLVM
  role: Latent-variable model used via its Python script to remove the top K leading
    factors from the gene-centered scRNA-Seq matrix
- name: ccRemover
  role: Comparison method from the same article that formally tests each principal
    component as a cell-cycle effect before removal (cutoff = 3 or 2, max_it = 4,
    nboot = 100–200, ntop = 10)
  repo: cran/ccRemover
- name: Python
  role: Runtime environment in which the scLVM script is executed
- name: org.Mm.eg.db
  role: Mouse cell-cycle gene annotations used to build the if_cc indicator for the
    comparable ccRemover workflow
- name: Biomart
  role: Source of cell-cycle gene lists used to annotate cell-cycle genes for the
    parallel correction comparison
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:0f6e7c8ec36a5b7e3d8576384135aadd24af2e546460fa41ed9e2f005392d799
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - Python
  - ccRemover
  - org.Hs.eg.db
  - org.Mm.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# latent-factor-removal-sclvm

## Summary

Apply scLVM to remove latent factors (primarily the cell-cycle effect) from single-cell RNA-Seq data by fitting a latent variable model with a user-chosen number K of leading factors, then assess downstream clustering/enrichment on the corrected matrix. In the source article scLVM is used as the comparison baseline against ccRemover on simulated data, T-helper cells, mESC, and glioblastoma datasets.

## When to use

Reference for removing dominant latent factors (e.g. cell cycle) from log-transformed, gene-centered scRNA-Seq matrices via scLVM's Python script, choosing K = 1 or the scree-plot value, and comparing corrected vs. original data.

## When NOT to use

- When the leading factor's identity must be statistically verified before removal — scLVM 'does not inspect whether a leading factor represents the cell cycle' and risks removing biological components of interest such as cell-type signal (K = 1 on simulated data removed the cell-type effect).
- When the goal is to remove only the cell-cycle effect while preserving cell-type separation and the data contain strong competing latent structure — ccRemover is the article's purpose-built alternative with a formal bootstrap t-test on the loading difference Δj.
- When a versioned, packaged R workflow is required — the article applies scLVM via a Python script from its GitHub page with no version number or explicit repository URL recorded.

## Inputs

- scRNA-Seq expression matrix (genes × cells) from e.g. the Buettner mESC dataset (E-MTAB-2802), Mahata T-helper cell dataset (t.cell_data, 81 cells × 14,147 genes), or Patel glioblastoma dataset (GSE57872)
- Number of latent factors K (default 1 or a scree-plot-derived value)
- scLVM Python script from the scLVM GitHub page

## Outputs

- scLVM-corrected expression matrix with the top K latent factors removed
- Scree plot / K selection diagnostics
- Downstream comparison statistics (clustering accuracies, TH2 signature odds ratios, e.g. scLVM OR = 2.382 vs. ccRemover 3.439; glioblastoma accuracy 90.00% for scLVM vs. 87.44% original)

## How to apply

Preprocess the scRNA-Seq matrix by log-transforming and centering gene-by-gene, then run scLVM using the Python script from the scLVM GitHub page. Choose K (the number of leading factors to remove) by trying both the default K = 1 and the value indicated by the scree plot, since the article shows the choice is consequential: on simulated data with a cell-type effect and a cell-cycle effect, K = 1 mistakenly removed the cell-type signal while K = 3 removed both effects, rendering the data noise. Apply the same clustering algorithms and statistical tests (e.g. 2-means clustering with a t-test at FDR 0.01, hypergeometric enrichment of signature genes, or hierarchical clustering into tumor-of-origin groups) to the original, scLVM-corrected, and — in the article's comparisons — ccRemover-corrected matrices to judge whether the intended effect was removed and biological signal preserved. The article's rationale is that scLVM does not inspect whether a leading factor represents the cell cycle, so validation of the corrected output against known structure (cell type, tumor of origin, or signature-gene enrichment) is essential.

## Related tools

- **scLVM** (Latent-variable model used via its Python script to remove the top K leading factors from the gene-centered scRNA-Seq matrix)
- **ccRemover** (Comparison method from the same article that formally tests each principal component as a cell-cycle effect before removal (cutoff = 3 or 2, max_it = 4, nboot = 100–200, ntop = 10)) — cran/ccRemover
- **Python** (Runtime environment in which the scLVM script is executed)
- **org.Mm.eg.db** (Mouse cell-cycle gene annotations used to build the if_cc indicator for the comparable ccRemover workflow)
- **Biomart** (Source of cell-cycle gene lists used to annotate cell-cycle genes for the parallel correction comparison)

## Examples

```
python scLVM.py --input t_cell_data_cen.csv --K 1  # also try scree-plot K; then 2-means + t-test FDR 0.01 on corrected matrix
```

## Evaluation signals

- On the simulated dataset (50 cells, 2,000 genes, 400 cell-cycle genes): scLVM with K = 1 should cluster cells by cell-cycle stage (i.e. it mistakenly removes the cell-type effect), and with K = 3 it should remove both effects leaving only noise — record outcomes in simulation_correction_comparison.csv.
- On the T-helper data: TH2 signature-gene hypergeometric enrichment odds ratio for scLVM-corrected data should be ≈ 2.382 (vs. 3.439 for ccRemover; original data OR < 1).
- On the mESC dataset (E-MTAB-2802): both scLVM and ccRemover should remove the cell-cycle effect by Buettner et al.'s two criteria (2-means clustering, t-test at FDR 0.01, hypergeometric odds ratio), with ccRemover slightly better; the comparison figure mesc_ccremover_sclvm_comparison.png must exist.
- On the Patel glioblastoma data: hierarchical clustering accuracy on scLVM-corrected data should be ≈ 90.00% (±2 pp), vs. 87.44% original and 92.32% ccRemover.
- K selection must be justified by trying both the default K = 1 and the scree-plot-indicated number of leading factors, with the scree plot retained as evidence.

## Limitations

- scLVM does not formally test whether a leading factor represents the cell cycle, so it has a high risk of removing other biological components of interest — on simulated data K = 1 removed the cell-type effect and K = 3 removed both effects, leaving 'just noise'.
- No version numbers for scLVM, ccRemover, or R annotation packages are reported, and no explicit GitHub URL or changelog is given for the scLVM script in the article text.
- scLVM's Python-script usage depends on the practitioner choosing K (default 1 vs. scree plot), and a wrong K can compromise downstream analyses.
- Unlike ccRemover, scLVM is not designed to verify annotation completeness; the article notes annotation databases are 'always incomplete and inaccurate', a shared caveat for defining the cell-cycle gene set.

## Evidence

- [results] scLVM is applied using the python script available from the scLVM GitHub page: "scLVM is applied using the python script available from the scLVM GitHub page"
- [results] For the choice of K ... we try both the default value K = 1 and the value given by the scree plot: "For the choice of K (the number of leading factors to be removed) in scLVM, we try both the default value K = 1 and the value given by the scree plot"
- [discussion] The current method developed for removing this effect, scLVM, does not inspect whether a leading factor represents the cell cycle: "The current method developed for removing this effect, scLVM, does not inspect whether a leading factor represents the cell cycle"
- [results] scLVM has failed completely here by mistakenly removing the cell-type effect instead of the cell-cycle effect: "scLVM has failed completely here by mistakenly removing the cell-type effect instead of the cell-cycle effect"
- [methods] We recommend transforming the data to a log scale and centering it on a gene-by-gene basis.: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [results] the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"
