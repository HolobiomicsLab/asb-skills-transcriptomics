---
name: cell-type-clustering-assessment
description: Use when an scRNA-Seq matrix has been corrected for the cell-cycle effect
  and you must verify that cell-type signal was retained while cell-cycle heterogeneity
  was removed, comparing original vs corrected matrices.
when_to_use_negative:
- The dataset has very low cell-cycle activity, as in the Patel et al. human glioblastoma
  data, where the article notes the effect is minimal and only a smaller cutoff CΔ
  = 2 is warranted — assessment of cell-cycle removal is largely moot.
- 'The goal is to remove other confounders (batch, mitochondrial, technical effects):
  ccRemover removes only the cell-cycle effect, so an odds-ratio comparison against
  it will not diagnose other dominant sources of variation.'
- There are no known cell-type or cell-cycle stage labels, so the hypergeometric odds-ratio
  / majority-class validation has no ground truth to enrich against.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: ccRemover
  role: Produces the cell-cycle-corrected matrix 'xhat' whose cell-type clustering
    is assessed against original and scLVM-corrected matrices
  repo: cran/ccRemover
- name: scLVM
  role: Baseline latent-factor correction (Python script, K=1 and scree-plot K) whose
    effect on cell-type separation is benchmarked against ccRemover
- name: Python
  role: Runs the scLVM Python script from the scLVM GitHub page on the same preprocessed
    matrices
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:849203f8785b1e66b4f655dd1594a38f86fb60675ebb8d88dbe49e97f20d1423
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - Python
  - ccRemover
  - org.Mm.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-type-clustering-assessment

## Summary

A benchmarking skill for judging whether a cell-cycle correction (ccRemover vs scLVM) preserves or improves cell-type separation in scRNA-Seq data, using matched clustering plus statistical tests (t-test at FDR 0.01, hypergeometric odds ratio) on original and corrected matrices.

## When to use

Use when an scRNA-Seq matrix has been corrected for the cell-cycle effect and you must verify that cell-type signal was retained while cell-cycle heterogeneity was removed, comparing original vs corrected matrices.

## When NOT to use

- The dataset has very low cell-cycle activity, as in the Patel et al. human glioblastoma data, where the article notes the effect is minimal and only a smaller cutoff CΔ = 2 is warranted — assessment of cell-cycle removal is largely moot.
- The goal is to remove other confounders (batch, mitochondrial, technical effects): ccRemover removes only the cell-cycle effect, so an odds-ratio comparison against it will not diagnose other dominant sources of variation.
- There are no known cell-type or cell-cycle stage labels, so the hypergeometric odds-ratio / majority-class validation has no ground truth to enrich against.

## Inputs

- Original log-transformed, gene-by-gene centered scRNA-Seq expression matrix (e.g. T helper, mESC E-MTAB-2802, glioblastoma MGH26-MGH31, or lung adenocarcinoma data)
- Corrected matrices: ccRemover 'xhat' output and scLVM-corrected matrices (K=1 and scree-plot-derived K)
- A priori cell-type / cell-cycle stage labels (e.g. Hoechst-stained mESC stages; simulated six cell-type-by-cycle-stage groups; glioblastoma tumor class)

## Outputs

- Cluster assignments per cell for each matrix variant (original, ccRemover-corrected, scLVM-corrected)
- Differentially expressed gene sets per cluster (t-test at FDR 0.01)
- Hypergeometric-test odds ratio quantifying cell-type signature enrichment per clustering
- Comparison table (simulation_correction_comparison.csv) and summary figure (mesc_ccremover_sclvm_comparison.png)

## How to apply

Apply the SAME clustering algorithm and statistical tests to the original, ccRemover-corrected, and scLVM-corrected matrices so performance is comparable. For T helper and mESC data, apply 2-means clustering, then use a t-test with false discovery rate 0.01 to identify differentially expressed genes in each cluster, and compute a hypergeometric-test odds ratio for enrichment of known cell-type (e.g. TH2) signature genes among the DEGs — an odds ratio less than 1 indicates the clustering is unlikely to be physiologically meaningful. For glioblastoma (MGH26-MGH31), use hierarchical clustering split into five clusters with each cluster assigned its majority tumor class; for lung adenocarcinoma, use 4-means clustering to test whether the 77 cells of the first cell type split by technical replicates. On the simulated 50-cell/2,000-gene data, success is defined as cells clustering into six cell-type-by-cell-cycle-stage groups originally, then by cell type alone with cell-cycle stages mixed after ccRemover (2nd and 3rd PCs removed), whereas scLVM with K=1 mistakenly removes the cell-type effect and with K=3 removes both effects leaving only noise. Record the comparison outcomes in files such as simulation_correction_comparison.csv and a summary figure (mesc_ccremover_sclvm_comparison.png).

## Related tools

- **ccRemover** (Produces the cell-cycle-corrected matrix 'xhat' whose cell-type clustering is assessed against original and scLVM-corrected matrices) — cran/ccRemover
- **scLVM** (Baseline latent-factor correction (Python script, K=1 and scree-plot K) whose effect on cell-type separation is benchmarked against ccRemover)
- **Python** (Runs the scLVM Python script from the scLVM GitHub page on the same preprocessed matrices)

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Hypergeometric odds ratio for the known cell-type signature (e.g. TH2) exceeds 1 on corrected data; odds ratio < 1 on original data confirms the bias was real.
- On simulated data, ccRemover-corrected cells cluster by cell type with cell-cycle stages mixed within clusters, and the 2nd and 3rd principal components are flagged as cell-cycle effects.
- scLVM outcomes match expectations: K=1 mistakenly removes the cell-type effect; K=3 removes both effects leaving just noise.
- Identical clustering algorithms and statistical tests (2-means, t-test FDR 0.01, odds ratio; or hierarchical/4-means per dataset) were applied to every matrix variant before comparison.
- Required output artifacts exist: simulation_correction_comparison.csv and mesc_ccremover_sclvm_comparison.png showing both methods remove the cell-cycle effect with ccRemover slightly outperforming scLVM.

## Limitations

- ccRemover may miss the cell-cycle effect if it influences too few control genes (below roughly 8% affected), since it will not be captured by the leading principal components.
- Annotation databases of cell-cycle genes are always incomplete and inaccurate, which weakens both the correction and the gene-set-based odds-ratio validation.
- ccRemover removes only the cell-cycle effect; other confounding effects may dominate the clustering after removal and will not be diagnosed by this assessment.

## Evidence

- [results] 2-means clustering with t-test at FDR 0.01 plus odds ratio: "we applied 2-means clustering and use a t-test with false discovery rate 0.01 to identify differentially expressed genes"
- [results] Odds ratio interpretation: "On the original data, the odds ratio is less than 1, indicating that the clustering of cells is unlikely to be physiologically meaningful"
- [results] Identical tests across matrix variants: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"
- [results] Dataset-specific clustering choices: "we consider using 4-means clustering, in order to see whether the 77 cells of the first cell type can be clustered accordingly to the two sets of technical replicates"
- [abstract] Rationale: ccRemover as pre-processing: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
