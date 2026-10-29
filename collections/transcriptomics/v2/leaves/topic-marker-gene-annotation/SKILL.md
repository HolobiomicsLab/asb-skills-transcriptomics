---
name: topic-marker-gene-annotation
description: Use when STdeconvolve has produced per-topic transcriptional profiles
  (beta) and cell-type proportions (theta) from multi-cellular pixel-resolution ST
  data and the recovered topics must be biologically labeled via marker genes.
when_to_use_negative:
- When a suitable single-cell RNA-seq reference exists and supervised methods like
  SPOTlight, RCTD, or spatialDWLS provide cell-type identities directly
- When topics cannot be matched to any known marker genes because the tissue or cell
  types are poorly characterized
- When the ST data cover sub-cellular pixel resolution or thin structures spanning
  only a few pixels, where deconvolution itself is unreliable
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: STdeconvolve
  role: Produces the LDA beta (per-topic transcriptional profiles) and theta (per-pixel
    proportions) whose top genes are annotated to cell types
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Runtime environment for the STdeconvolve R package used to extract and inspect
    topic-gene matrices
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor prior to annotation workflows
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Alternative installer for the development version of STdeconvolve from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:b63e40dd70ee90212c7ca065b7a462ed7cf6116bdd3ff6215f49c4360758e8c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# topic-marker-gene-annotation

## Summary

Annotate deconvolved cell-type transcriptional profiles (LDA beta topics) from STdeconvolve by inspecting top genes per topic and matching them to known marker genes, e.g. Myh6 for atrium, Myh7 for ventricle, Apoa2 for liver, Hba.a2 for blood vessels, and Purkinje/Bergmann glia signatures.

## When to use

Use when STdeconvolve has produced per-topic transcriptional profiles (beta) and cell-type proportions (theta) from multi-cellular pixel-resolution ST data and the recovered topics must be biologically labeled via marker genes.

## When NOT to use

- When a suitable single-cell RNA-seq reference exists and supervised methods like SPOTlight, RCTD, or spatialDWLS provide cell-type identities directly
- When topics cannot be matched to any known marker genes because the tissue or cell types are poorly characterized
- When the ST data cover sub-cellular pixel resolution or thin structures spanning only a few pixels, where deconvolution itself is unreliable

## Inputs

- Deconvolved per-topic transcriptional profile matrix (beta) from STdeconvolve getBetaTheta
- Per-pixel cell-type proportion matrix (theta) from STdeconvolve getBetaTheta
- Pixel spatial coordinates and tissue annotations (e.g. mOB$pos, mOB$annot)
- Known marker gene lists (e.g. Myh6, Myh7, Apoa2, Hba.a2)

## Outputs

- Annotated topics mapping LDA cell types to biological labels based on marker genes
- Combined table of identified cell-type counts and marker-gene assignments per dataset (e.g. celltype_counts_markers.csv)

## How to apply

After fitting the LDA model, extract the per-topic transcriptional profiles (beta) with getBetaTheta and rank genes by their beta weight within each topic. For each dataset's pre-specified K (e.g. K=20 Visium, K=13 DBiT-seq E11, K=14 Slide-seq cerebellum), inspect the top genes of each topic and match them against known cell-type or anatomical marker genes (Myh6 atrium, Myh7 ventricle, Apoa2 liver, Hba.a2 blood vessels for DBiT-seq E11 embryo; Purkinje and Bergmann glia signatures for Slide-seq cerebellum). Cross-reference topic spatial proportions (theta) with tissue annotations to confirm that annotated topics co-localize with the expected anatomical features. Finally, tabulate identified cell-type counts and marker-gene assignments per dataset (e.g. writing a combined table to celltype_counts_markers.csv). Correct application is confirmed when each expected marker gene appears among the top genes of its corresponding annotated topic.

## Related tools

- **STdeconvolve** (Produces the LDA beta (per-topic transcriptional profiles) and theta (per-pixel proportions) whose top genes are annotated to cell types) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Runtime environment for the STdeconvolve R package used to extract and inspect topic-gene matrices) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor prior to annotation workflows) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Alternative installer for the development version of STdeconvolve from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); deconGexp <- results$beta; top.genes <- apply(deconGexp, 1, function(x) names(head(sort(x, decreasing=TRUE), 10)))
```

## Evaluation signals

- Each dataset's deconvolution yields exactly the pre-specified number of topics (K=20 for Visium, K=13 for DBiT-seq E11, K=14 for Slide-seq cerebellum)
- Expected marker genes (Myh6 atrium, Myh7 ventricle, Apoa2 liver, Hba.a2 blood vessels) appear as top genes of the corresponding annotated topics
- Purkinje and Bergmann glia signatures are recovered as top genes in the Slide-seq cerebellum topics
- Annotated topic spatial proportions (theta) co-localize with matching anatomical features in the tissue
- Combined cell-type counts and marker-gene assignments are written for all three datasets (celltype_counts_markers.csv)

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes; deconvolution accuracy underlying topic annotation degrades when assumptions are not satisfied
- If genes lack variability across pixels due to homogeneous cell-type proportions, STdeconvolve may fail to deconvolve transcriptionally distinct cell types, making topic annotation meaningless
- Sparse ST data with high stochastic drop-out rates may cause LDA to struggle to identify distinct groups of co-expressed genes
- ST profiling of tissue slivers or thin structures covering only a few pixels presents challenges for deconvolution and subsequent annotation

## Evidence

- [full_text] On real 25 µm² DBiT-seq data of the E11 mouse embryo, STdeconvolve's deconvolved cell-type transcriptional profiles contained the expected marker genes of matching anatomical features, such as Myh6 for the atrium and Myh7 for the ventricle.: [evidence span withheld — anchor status: not_found]
- [intro] STdeconvolve applies latent Dirichlet allocation (LDA) to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel.: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [readme] Extract deconvolved cell-type proportions (theta) and transcriptional profiles (beta).: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)"
- [discussion] If the gene expression in the ST data is too sparse with high rates of stochastic drop-outs, then the LDA model may struggle to identify distinct groups of co-expressed genes.: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
