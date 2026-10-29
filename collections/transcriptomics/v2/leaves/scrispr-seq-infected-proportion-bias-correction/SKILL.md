---
name: scrispr-seq-infected-proportion-bias-correction
description: Use when analyzing scCRISPR-seq data (e.g. CROP-seq with perturbation
  matrix Y and control matrix Z) where cluster proportions differ between gRNA-perturbed
  and control groups, distorting fold-change estimates of perturbation response.
when_to_use_negative:
- When cell populations are intricate or diseased tissues such as tumors, where the
  GMM assumption on expression distributions may not hold
- When some rare cell types are completely absent from infection, violating the same-cluster
  assumption between NT and perturbation groups
- When cells have extreme (not moderate) heterogeneity, since scDecouple assumes similar
  cellular responses across cell clusters
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: Core R package performing preprocessing, PC selection, EM-based decoupling,
    and downstream gene ranking/enrichment
  repo: https://github.com/MengQiuchen/scDecouple
- name: Seurat
  role: Loaded in the demo workflow for single-cell data handling
- name: mixtools
  role: Provides the Gaussian mixture model fitting used in the decoupling step
- name: diptest
  role: Dip-test multimodality statistics used by pc_selection to flag infection-biased
    PCs
- name: enrichR
  role: Gene Ontology / pathway enrichment in downstream_analysis
- name: devtools
  role: Installs scDecouple from GitHub with build_vignettes=TRUE
  repo: https://github.com/MengQiuchen/scDecouple
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:1183dcf0402cad5304c96f8f2641af799cec41a09312ebda2cb636adcf00c20e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - devtools
  - diptest
  - enrichR
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# scrispr-seq-infected-proportion-bias-correction

## Summary

Applies the scDecouple R package to decouple true cellular response from infected proportion bias in scCRISPR-seq data, using a Gaussian mixture model with EM maximum-likelihood estimation over infection-biased PCs.

## When to use

Use when analyzing scCRISPR-seq data (e.g. CROP-seq with perturbation matrix Y and control matrix Z) where cluster proportions differ between gRNA-perturbed and control groups, distorting fold-change estimates of perturbation response.

## When NOT to use

- When cell populations are intricate or diseased tissues such as tumors, where the GMM assumption on expression distributions may not hold
- When some rare cell types are completely absent from infection, violating the same-cluster assumption between NT and perturbation groups
- When cells have extreme (not moderate) heterogeneity, since scDecouple assumes similar cellular responses across cell clusters

## Inputs

- Perturbation group cell-by-gene count matrix Y
- Control group (unperturbed or NT gRNA cells) cell-by-gene matrix Z
- Optional parameters: using_rank (default TRUE), top.sdev (default 30), top.multimodal (default 3), th.sdev (default 1), th.multimodal (default 0.0125), degenes.num (default 1500), seed_ (default 0)

## Outputs

- Decoupled cellular response estimates (beta.PC.scDecouple) per gRNA
- Cellular response heatmaps and PC variance/multimodality visualizations
- Gene rankings and Gene Ontology enrichment plots from downstream_analysis
- Infected proportion bias quantification

## How to apply

Install scDecouple from GitHub via devtools and provide a cell-by-gene perturbation matrix Y and control matrix Z. Run data_preprocessing for log-normalization and highly variable gene detection, then pc_selection on the control PCA with using_rank=TRUE (top.sdev=30, top.multimodal=3) to identify PCs with high infection bias via standard-deviation rank and dip-test multimodality. Run scDecouple on the selected PCs with a K=2 Gaussian mixture model and EM algorithm (seed_=0) to estimate cell-cluster proportions and cellular response per gRNA, then run downstream_analysis (degenes.num=1500) for gene rankings and GO enrichment. The one-stop run_scDecouple(Y, Z) wraps all four steps with defaults (using_rank=TRUE, degenes.num=1500, seed_=0).

## Related tools

- **scDecouple** (Core R package performing preprocessing, PC selection, EM-based decoupling, and downstream gene ranking/enrichment) — https://github.com/MengQiuchen/scDecouple
- **Seurat** (Loaded in the demo workflow for single-cell data handling)
- **mixtools** (Provides the Gaussian mixture model fitting used in the decoupling step)
- **diptest** (Dip-test multimodality statistics used by pc_selection to flag infection-biased PCs)
- **enrichR** (Gene Ontology / pathway enrichment in downstream_analysis)
- **devtools** (Installs scDecouple from GitHub with build_vignettes=TRUE) — https://github.com/MengQiuchen/scDecouple

## Examples

```
res <- run_scDecouple(Y, Z); # or stepwise: res.step1 <- data_preprocessing(Y, Z); res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)
```

## Evaluation signals

- Results table must contain a Fisher's exact test comparison of scDecouple GO enrichment versus the naive observed-change (OC) estimate
- On the Jurkat CROP-seq dataset (22 target genes with >60 infected cells, top 700 HVGs), scDecouple should improve TCR-related gene ranking for a majority of gRNAs with an average gain of 125 over OC
- pc_selection with using_rank=TRUE should yield three infection-biased PCs on the Jurkat control PCA
- CD69 and RELA positions extracted from gene rankings should be consistent with the reported average ranking improvements of 280 and 125
- Stepwise results (res.step1–res.step4) should match the fields returned by run_scDecouple(Y, Z): selected_pcs, beta.PC.scDecouple, gene rankings, GO enrichment

## Limitations

- GMM assumption may not hold in intricate systems or disease tissues such as tumors
- Same cell clusters between NT and perturbation groups assumption may fail when rare cell types are completely absent from infection
- Similar cellular responses across cell clusters assumption limits scDecouple to cells with moderate heterogeneity
- Only a developmental version from GitHub is available; no version number is stated

## Evidence

- [abstract] We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias.: "We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias."
- [readme] README installation: "devtools::install_github("MengQiuchen/scDecouple", build_vignettes = TRUE)"
- [readme] README PC selection step: "res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)"
- [readme] README downstream analysis step: "res.step4 <- downstream_analysis(Z.pca.rotation = res.step1$Z.pca.results$rotation, beta.PC.scDecouple = res.step3$beta.PC.scDecouple, Z.norm.log.sub = res.step1$Z.norm.log.sub, Y.norm.log.sub ="
- [intro] GMM/EM method description: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation. scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [abstract] Real-data benefit: "By applying scDecouple to real scCRISPR-seq data, we found that scDecouple enhances the identification of biologically perturbation-related genes."
