---
name: allen-brain-atlas-ish-cross-validation
description: Use when a reference-free or reference-based deconvolution yields an
  unlabeled cell type whose spatial localization is plausible but whose identity needs
  independent validation via in situ hybridization gene expression patterns.
when_to_use_negative:
- The cell type's top genes have no in situ hybridization data available in Allen
  Brain Atlas (e.g. non-brain tissue or poorly characterized genes)
- The deconvolution produced no spatially coherent cell type, so there is no spatial
  pattern to compare against ISH anatomy
- A curated single-cell reference with confident cell-type annotations already exists,
  making ISH cross-validation redundant for annotation
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: STdeconvolve
  role: Reference-free LDA deconvolution that produces the cell-type transcriptional
    profiles (beta) whose top genes are cross-checked against Allen Brain Atlas ISH
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Language in which STdeconvolve is implemented and the X7 gene list is extracted
    and saved
- name: BiocManager
  role: Installs the STdeconvolve package from Bioconductor in the validation workflow
- name: remotes
  role: Alternative installer for STdeconvolve development version from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:6202716500f40d721963330ef1942ec69c77e674b7b0888eee47ac3e3a122292
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# allen-brain-atlas-ish-cross-validation

## Summary

Validate a deconvolved cell type by extracting its top upregulated genes (e.g. Sox11, Nrep, Dcx for STdeconvolve MOB cluster X7) and cross-checking their spatial expression against Allen Brain Atlas in situ hybridization images.

## When to use

Use when a reference-free or reference-based deconvolution yields an unlabeled cell type whose spatial localization is plausible but whose identity needs independent validation via in situ hybridization gene expression patterns.

## When NOT to use

- The cell type's top genes have no in situ hybridization data available in Allen Brain Atlas (e.g. non-brain tissue or poorly characterized genes)
- The deconvolution produced no spatially coherent cell type, so there is no spatial pattern to compare against ISH anatomy
- A curated single-cell reference with confident cell-type annotations already exists, making ISH cross-validation redundant for annotation

## Inputs

- Deconvolved cell-type transcriptional profiles (beta matrix) from STdeconvolve fitLDA/getBetaTheta
- Per-pixel cell-type proportion matrix (theta) with pixel spatial coordinates (mOB$pos)
- Candidate cell type of interest (e.g. cluster X7 from a K=12 MOB fit)
- Allen Brain Atlas in situ hybridization expression images for candidate marker genes

## Outputs

- Ranked list of top upregulated genes for the cell type (e.g. Sox11, Nrep, Dcx for X7)
- Cross-validation assessment of gene ISH pattern vs. deconvolved spatial proportions
- Fitted model, proportion matrix, and validated gene list saved (e.g. mob_k12_deconvolution.rds)

## How to apply

After fitting the deconvolution model (e.g. STdeconvolve LDA with a chosen K, such as K=12 on the built-in mOB mouse main olfactory bulb dataset), identify the cell type of interest by its spatial proportion map — for X7, proportion overlapping the granule cell layer at the expected rostral migratory stream (RMS) location. Extract the top upregulated genes from that cell type's transcriptional profile (beta matrix), here Sox11, Nrep, and Dcx, using a scaled beta profile (betaScale = 1000) to rank genes. Query the Allen Brain Atlas in situ hybridization resource for each gene and compare the anatomical ISH pattern to the cell type's spatial localization in the tissue section. Agreement between the deconvolved spatial map and independent ISH anatomy (e.g. RMS-restricted neuroblast marker expression such as Dcx) supports the assignment of an anatomical/neurodevelopmental identity to the unsupervised cluster. Record the validated gene list alongside the fitted model outputs for reproducibility.

## Related tools

- **STdeconvolve** (Reference-free LDA deconvolution that produces the cell-type transcriptional profiles (beta) whose top genes are cross-checked against Allen Brain Atlas ISH) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Language in which STdeconvolve is implemented and the X7 gene list is extracted and saved)
- **BiocManager** (Installs the STdeconvolve package from Bioconductor in the validation workflow)
- **remotes** (Alternative installer for STdeconvolve development version from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); top_genes <- names(sort(results$beta['X7', ], decreasing = TRUE))[1:10]
```

## Evaluation signals

- Top genes of the cell type (X7: Sox11, Nrep, Dcx) show ISH expression patterns in Allen Brain Atlas that overlap the same anatomical region where the cell type's theta proportions are elevated
- The cell type's spatial proportion map localizes over the expected anatomical layer (granule cell layer / RMS in the mOB dataset)
- The saved RDS (mob_k12_deconvolution.rds) contains the fitted LDA model, theta proportion matrix, and the X7 gene list
- Gene rankings are computed from the scaled beta matrix (betaScale = 1000) rather than raw counts, ensuring per-cell-type comparability
- The number of deconvolved cell types matches the pre-specified K (12 for the MOB analysis)

## Limitations

- LDA deconvolution performance depends on dataset size in terms of number of pixels and number of genes, and may struggle with sparse ST data with high stochastic drop-out rates
- If cell-type proportions are homogeneous across pixels, genes lack variability and STdeconvolve may fail to deconvolve distinct cell types, leaving nothing meaningful to validate against ISH
- ST profiling of thin structures covering only a few pixels (such as the RMS itself) presents challenges for deconvolution

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [readme] results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000): "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)"
- [discussion] the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes: "the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes"
- [intro] STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
