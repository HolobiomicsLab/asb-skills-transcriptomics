---
name: perturbation-gene-ranking-evaluation
description: Use when comparing perturbation-effect estimators (scDecouple vs OC)
  on scCRISPR-seq data by measuring gene-ranking gains for TCR/pathway-related genes
  and enrichment significance.
when_to_use_negative:
- Data are from intricate systems or disease tissues such as tumors, where the Gaussian
  mixture model assumption on cluster proportions may not hold
- Rare cell types are completely absent from infection, so the assumption of same
  cell clusters between NT and perturbation groups fails
- Samples have extreme heterogeneity; scDecouple is limited to cells with moderate
  heterogeneity by its similar-response-across-clusters assumption
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDecouple
  role: R package providing data_preprocessing, pc_selection, scDecouple (K=2 GMM
    + EM decoupling), and downstream_analysis for gene ranking and GO enrichment
  repo: https://github.com/MengQiuchen/scDecouple
- name: R
  role: Runtime environment for the scDecouple pipeline and downstream comparison
- name: tidyverse
  role: Loaded for data manipulation in the demo workflow
- name: enrichR
  role: GO/pathway enrichment in downstream_analysis
- name: mixtools
  role: Gaussian mixture model fitting for decoupling
- name: diptest
  role: Dip-test multimodality used in PC selection (top.multimodal=3)
- name: Seurat
  role: Loaded in the demo workflow for single-cell handling/PCA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:2f2c2c1e60ca674cb675288f08b385a840030572bc5090531ecf34d6cca9b2c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - diptest
  - enrichR
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# perturbation-gene-ranking-evaluation

## Summary

Evaluate whether scDecouple's decoupled cellular-response estimate improves the ranking of perturbation-related genes versus the naive observed-change (OC) estimate on real scCRISPR-seq data, using GO pathway enrichment and position of target genes (e.g., CD69, RELA) in gene rankings.

## When to use

Use when comparing perturbation-effect estimators (scDecouple vs OC) on scCRISPR-seq data by measuring gene-ranking gains for TCR/pathway-related genes and enrichment significance.

## When NOT to use

- Data are from intricate systems or disease tissues such as tumors, where the Gaussian mixture model assumption on cluster proportions may not hold
- Rare cell types are completely absent from infection, so the assumption of same cell clusters between NT and perturbation groups fails
- Samples have extreme heterogeneity; scDecouple is limited to cells with moderate heterogeneity by its similar-response-across-clusters assumption

## Inputs

- Perturbation group cell-by-gene count matrix Y (e.g., Jurkat CROP-seq data, 22 target genes restricted to >60 infected cells)
- Control group cell-by-gene count matrix Z (NT gRNA / unperturbed cells)
- OC (observed-change) baseline estimate per gRNA for comparison

## Outputs

- scDecouple gene rankings per gRNA and GO pathway enrichment (degenes.num=1500)
- Fisher's exact test comparison of scDecouple vs OC pathway enrichment
- CD69 and RELA positions in gene rankings per gRNA
- Average ranking gain across gRNAs (reported: 280 for enrichment, 125 for gene ranking)

## How to apply

Run the scDecouple pipeline step by step: data_preprocessing (log-normalization, top 700 highly variable genes), pc_selection with using_rank=TRUE (top.sdev=30, top.multimodal=3) to find infection-biased PCs, scDecouple with a K=2 Gaussian mixture model fit by EM (seed_=0), then downstream_analysis with degenes.num=1500 to obtain gene rankings and GO enrichment. As a baseline, compute the naive observed-change (OC) estimate from fold change between perturbation and control groups. Compare pathway enrichment between scDecouple and OC with Fisher's exact test, and extract the ranks of target genes (CD69 and RELA) from both rankings; on the Jurkat CROP-seq data the reported average ranking improvements are 280 for enrichment-ranked genes and 125 for gene ranking across all gRNAs.

## Related tools

- **scDecouple** (R package providing data_preprocessing, pc_selection, scDecouple (K=2 GMM + EM decoupling), and downstream_analysis for gene ranking and GO enrichment) — https://github.com/MengQiuchen/scDecouple
- **R** (Runtime environment for the scDecouple pipeline and downstream comparison)
- **tidyverse** (Loaded for data manipulation in the demo workflow)
- **enrichR** (GO/pathway enrichment in downstream_analysis)
- **mixtools** (Gaussian mixture model fitting for decoupling)
- **diptest** (Dip-test multimodality used in PC selection (top.multimodal=3))
- **Seurat** (Loaded in the demo workflow for single-cell handling/PCA)

## Examples

```
res <- run_scDecouple(Y, Z); # or stepwise: res.step4 <- downstream_analysis(Z.pca.rotation=res.step1$Z.pca.results$rotation, beta.PC.scDecouple=res.step3$beta.PC.scDecouple, Z.norm.log.sub=res.step1$Z.norm.log.sub, Y.norm.log.sub=res.step1$Y.norm.log.sub, genes.variable=res.step1$genes.variable)
```

## Evaluation signals

- Results table contains the Fisher's-exact-test enrichment comparison of scDecouple vs OC and the CD69/RELA rankings per gRNA
- Average gene-ranking gain across gRNAs is consistent with the reported increase of 125
- Average enrichment-related ranking improvement consistent with the reported 280 (e.g., CD69/RELA position gains)
- pc_selection with using_rank=TRUE (top.sdev=30, top.multimodal=3) yields the expected infection-biased PCs on the control-group PCA
- downstream_analysis returns rankings built on degenes.num=1500 DE genes for GO enrichment

## Limitations

- GMM assumption may not hold in intricate systems or disease tissues such as tumors
- Same cell clusters between NT and perturbation groups assumption may fail when rare cell types are completely absent from infection
- Similar cellular responses across cell clusters assumption limits scDecouple to cells with moderate heterogeneity

## Evidence

- [results] Application improves gene ranking of TCR-related genes: "By applying scDecouple to real scCRISPR-seq data, we found that scDecouple enhances the identification of biologically perturbation-related genes."
- [abstract] GMM + EM decoupling method: "scDecouple first models the distribution of gene expression profiles in perturbed cells and then iteratively finds the maximum likelihood of cell cluster proportions as well as the cellular response"
- [readme] Step-by-step pipeline in README: "res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2)"
- [readme] PC selection parameters: "top.multimodal` (optional): When using_rank=TRUE, this parameter sets the threshold of dip statistics. The default is 3."
- [readme] GO enrichment DE gene count: "degenes.num` (optional): The number of differentially expressed genes (DE genes) for GO enrichment calculation. The default is 1500."
- [intro] Infected proportion bias motivation: "the data analysis may suffer from gRNA ‘infected proportion bias’. The infection proportion bias refers to the significant differences in the proportions of cell clusters between the perturbation"
