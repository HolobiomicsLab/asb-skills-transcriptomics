---
name: multi-platform-st-dataset-benchmarking
description: Use when evaluating reference-free cell-type deconvolution on multi-cellular
  pixel-resolution spatial transcriptomics data from multiple platforms (Visium, DBiT-seq,
  Slide-seq) and checking that known marker genes are recovered.
when_to_use_negative:
- Data are sub-cellular pixel-resolution ST, where the authors state new methods specifically
  suited for that resolution will be needed
- Tissue slivers or thin structures covering only a few pixels, which the discussion
  flags as challenging for deconvolution by STdeconvolve
- Gene expression is too sparse with high stochastic drop-out rates or cell-type proportions
  are homogeneous across pixels, where LDA may fail to identify distinct groups of
  co-expressed genes
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution of multi-cellular pixel-resolution
    ST data; performs overdispersed gene feature selection, LDA fitting at chosen
    K, and extraction of theta/beta
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs the STdeconvolve package from Bioconductor
- name: remotes
  role: Installs the development version of STdeconvolve from GitHub
- name: R
  role: Runtime environment in which STdeconvolve is executed (R >= 4.2 for Bioconductor
    release 3.15)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:adab958e14279a5a8d8027d10abc5c112a5357f901d0d6784fbf7c0ad8ee169c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-platform-st-dataset-benchmarking

## Summary

Benchmark STdeconvolve, a reference-free LDA-based cell-type deconvolution, across real high-resolution multi-cellular ST datasets from diverse technologies (10X Visium, DBiT-seq, Slide-seq) at pre-specified numbers of cell types K, verifying that deconvolved transcriptional profiles recover expected marker genes.

## When to use

Use when evaluating reference-free cell-type deconvolution on multi-cellular pixel-resolution spatial transcriptomics data from multiple platforms (Visium, DBiT-seq, Slide-seq) and checking that known marker genes are recovered.

## When NOT to use

- Data are sub-cellular pixel-resolution ST, where the authors state new methods specifically suited for that resolution will be needed
- Tissue slivers or thin structures covering only a few pixels, which the discussion flags as challenging for deconvolution by STdeconvolve
- Gene expression is too sparse with high stochastic drop-out rates or cell-type proportions are homogeneous across pixels, where LDA may fail to identify distinct groups of co-expressed genes

## Inputs

- 10X Visium spatial transcriptomics counts (K=20)
- DBiT-seq 25 µm² E11 mouse embryo pixel counts (K=13)
- Slide-seq cerebellum pixel-by-gene counts (K=14)
- Pre-specified number of cell types K per dataset

## Outputs

- Per-pixel cell-type proportion matrix (theta/deconProp)
- Per-topic cell-type transcriptional profiles (beta/deconGexp)
- Combined table of identified cell-type counts and marker-gene assignments per dataset (celltype_counts_markers.csv)

## How to apply

Install STdeconvolve from Bioconductor via BiocManager (R >= 4.2 for release 3.15) or from GitHub with remotes::install_github('JEFworks-Lab/STdeconvolve'). For each dataset, preprocess with cleanCounts (e.g. min.lib.size = 100) to drop low-content pixels, then feature-select overdispersed genes with restrictCorpus (genes with higher-than-expected variance across pixels). Fit the LDA model with fitLDA at the pre-specified K (20 for Visium, 13 for DBiT-seq E11 embryo, 14 for Slide-seq cerebellum), confirming the choice with STdeconvolve's data-driven metrics and optimalModel. Extract per-pixel cell-type proportions (theta) and per-topic transcriptional profiles (beta) with getBetaTheta (e.g. perc.filt = 0.05, betaScale = 1000), then annotate each topic by matching recovered top genes to expected markers (Myh6 atrium, Myh7 ventricle, Apoa2 liver, Hba.a2 blood vessels for DBiT-seq; Purkinje and Bergmann glia signatures for Slide-seq). Record identified cell-type counts and marker-gene assignments per dataset in a combined table (celltype_counts_markers.csv).

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution of multi-cellular pixel-resolution ST data; performs overdispersed gene feature selection, LDA fitting at chosen K, and extraction of theta/beta) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs the STdeconvolve package from Bioconductor)
- **remotes** (Installs the development version of STdeconvolve from GitHub)
- **R** (Runtime environment in which STdeconvolve is executed (R >= 4.2 for Bioconductor release 3.15))

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 20, by = 1)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- Each dataset's deconvolution yields exactly the specified number of topics (K=20 for Visium, K=13 for DBiT-seq, K=14 for Slide-seq)
- Expected marker genes appear as top genes of the corresponding annotated topics (e.g. Myh6 atrium, Myh7 ventricle, Apoa2 liver, Hba.a2 blood vessels)
- Slide-seq cerebellum topics show Purkinje and Bergmann glia signatures
- LDA selection is confirmed with STdeconvolve's data-driven metrics before fixing K
- Output celltype_counts_markers.csv contains cell-type counts and marker-gene assignments for all three datasets

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes, which may limit deconvolution when assumptions are not satisfied
- Sparse ST data with high stochastic drop-out rates may prevent LDA from identifying distinct groups of co-expressed genes
- Homogeneous or uniform cell-type proportions across pixels leave genes without variability, and STdeconvolve may fail to deconvolve transcriptionally distinct cell types

## Evidence

- [abstract] STdeconvolve recovers expected marker genes on DBiT-seq: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available, as well as potentially superior performance when suitable"
- [intro] Overdispersed gene feature selection rationale: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [intro] Data-driven K estimation: "STdeconvolve provides several data-driven metrics to guide the estimation of an appropriate K"
- [intro] LDA infers cell-type profiles and proportions: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [readme] GitHub installation command: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [readme] R version requirement via Bioconductor: [evidence span withheld — anchor status: external_doc]
- [discussion] Homogeneous proportions failure mode: "if genes do not exhibit variability across pixels due to a homogeneous or uniform proportional representation of cell types across pixels (Supplementary Fig. S20), STdeconvolve may fail to deconvolve"
- [discussion] Sparse data limitation: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
