---
name: anova-differential-expression-testing-across-time-points
description: Use when scRNA-seq data span ordered time points of a differentiation
  process (e.g., primary human myoblast GSE52529 at 0/24/48/72 h) and a gene set is
  needed to drive multi-branching trajectory reconstruction.
when_to_use_negative:
- The data do not contain replicate cells or samples grouped across multiple time-point
  conditions, so an ANOVA design cannot be fit
- The analysis goal is trajectory inference that does not rely on a time-course DEG
  feature set (Mpath's other contrasts use only end-stage comparisons)
- A DEG table with adjusted P values has already been produced — re-running ANOVA
  would duplicate the step
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment for running ANOVA with post-hoc Tukey's test and Benjamini-Hochberg
    adjustment on the time-course expression data, per the Mpath supplementary workflow
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: Mpath
  role: Downstream consumer of the ANOVA-derived DEGs for neighborhood-based cell
    state transition mapping and multi-branching trajectory reconstruction
  repo: https://github.com/JinmiaoChenLab/Mpath
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:49e760488817dbe6a200ec6639fea88ead8ab3bdbfb37b1b3b311f001e5af27b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# ANOVA differential expression testing across time points

## Summary

Identifies differentially expressed genes across ordered differentiation time points using ANOVA followed by post-hoc Tukey's test with Benjamini-Hochberg multiple-testing correction, in the Mpath trajectory-reconstruction workflow. The resulting DEG set (e.g., from primary human myoblast scRNA-seq across 0/24/48/72 h) serves as the gene feature input for Mpath's neighborhood-based cell state transition mapping.

## When to use

Use when scRNA-seq data span ordered time points of a differentiation process (e.g., primary human myoblast GSE52529 at 0/24/48/72 h) and a gene set is needed to drive multi-branching trajectory reconstruction.

## When NOT to use

- The data do not contain replicate cells or samples grouped across multiple time-point conditions, so an ANOVA design cannot be fit
- The analysis goal is trajectory inference that does not rely on a time-course DEG feature set (Mpath's other contrasts use only end-stage comparisons)
- A DEG table with adjusted P values has already been produced — re-running ANOVA would duplicate the step

## Inputs

- Single-cell RNA-seq expression matrix spanning ordered differentiation time points (e.g., primary human myoblast scRNA-seq, GEO accession GSE52529, 0/24/48/72 h)
- Per-cell time-point metadata defining the ANOVA grouping factor

## Outputs

- List of ANOVA-derived differentially expressed genes with post-hoc Tukey's pairwise contrasts
- Benjamini-Hochberg-adjusted P values for each gene
- DEG expression matrix used as input to Mpath neighborhood-based trajectory mapping

## How to apply

Load the time-course expression matrix in R (e.g., GEO accession GSE52529 primary human myoblast scRNA-seq) and fit an ANOVA model per gene across the time-point groups (0, 24, 48, 72 h). For genes with significant ANOVA, run post-hoc Tukey's test to resolve which pairwise time-point contrasts drive the difference. Adjust P values for multiple testing with Benjamini-Hochberg and retain significant DEGs; optionally cluster the DEGs to summarize expression patterns before downstream use. Feed the ANOVA-derived DEG matrix into Mpath, which 'utilizes genes only from end-stage contrasts' and requires no a priori temporal ordering of developmental subsets to construct the multi-branching trajectory.

## Related tools

- **R** (Environment for running ANOVA with post-hoc Tukey's test and Benjamini-Hochberg adjustment on the time-course expression data, per the Mpath supplementary workflow) — https://github.com/JinmiaoChenLab/Mpath
- **Mpath** (Downstream consumer of the ANOVA-derived DEGs for neighborhood-based cell state transition mapping and multi-branching trajectory reconstruction) — https://github.com/JinmiaoChenLab/Mpath

## Examples

```
fit <- aov(t(exprs) ~ time_point, data=pd); tk <- TukeyHSD(fit); padj <- p.adjust(fit$`Pr(>F)`, method="BH")
```

## Evaluation signals

- Every reported DEG carries a Benjamini-Hochberg-adjusted P value and, where ANOVA is significant, post-hoc Tukey's pairwise time-point contrasts
- Downstream Mpath run on the DEG matrix reproduces the reported 8 designated landmark states, the two-branch split between muscle (MYOG, MYOD1, MEF2C) and non-muscle (PDGFRA) paths, and 7 ordered temporal waves of gene regulation
- Cluster analysis of the DEGs yields distinct expression clusters consistent with the published supplementary figures
- The DEG set, when supplied to Mpath, recapitulates the 0/24/48/72 h differentiation time course rather than a random ordering of cells

## Limitations

- The ANOVA-derived DEG approach was demonstrated only on two datasets: mouse bone marrow-derived DC progenitors and primary human myoblasts
- Genes are derived from end-stage contrasts without a priori temporal ordering knowledge, so informative early transitions may depend on the chosen time points
- The downstream Mpath algorithm depends on signature genes of differentiated cell types and is not yet extended to highly heterogeneous single-cell datasets where specific subset information is unknown

## Evidence

- [supplementary] The DEGs were identified using ANOVA followed by post-hoc Tukey's test.: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test."
- [supplementary] P values were adjusted for multi-test correction using Benjamini-Hochberg.: "P values were adjusted for multi-test correction using Benjamini-Hochberg."
- [discussion] without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts.: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
- [abstract] Applied to human myoblasts, Mpath recapitulates the time course of myoblast differentiation and isolates a branch of non-muscle cells involved in the differentiation.: "Applied to human myoblasts, Mpath recapitulates the time course of myoblast differentiation and isolates a branch of non-muscle cells involved in the differentiation."
