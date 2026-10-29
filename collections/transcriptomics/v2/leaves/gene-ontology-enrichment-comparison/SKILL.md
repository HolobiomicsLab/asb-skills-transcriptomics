---
name: gene-ontology-enrichment-comparison
description: Use when scCRISPR-seq analysis needs to judge whether decoupled perturbation
  responses (vs the naive OC estimate) yield better pathway enrichment and perturbation-related
  gene rankings, e.g. for TCR-related genes like CD69 and RELA on Jurkat CROP-seq
  data.
when_to_use_negative:
- The sample is an intricate system or disease tissue such as a tumor, where the Gaussian
  mixture model assumption underlying scDecouple's decoupling may not hold
- Rare cell types are completely absent from infection, violating the assumption that
  NT and perturbation groups share the same cell clusters
- Cell populations show extreme rather than moderate heterogeneity, breaching the
  assumption of similar cellular responses across cell clusters
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: R package whose downstream_analysis function computes gene rankings and GO
    enrichment from the decoupled responses
  repo: https://github.com/MengQiuchen/scDecouple
- name: enrichR
  role: R library used for Gene Ontology enrichment calculation in the downstream
    step
- name: R
  role: Environment in which the scDecouple package and downstream enrichment analysis
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:27dff7d6820f63e53202996d5ee8d37b62dd9d3d5d645198c08a12a5b191ee0a
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gene-ontology-enrichment-comparison

## Summary

Performs Gene Ontology pathway enrichment on the gene rankings produced by scDecouple's downstream_analysis step (top degenes.num=1500 DE genes via enrichR) and statistically compares enrichment against the naive observed-change (OC) estimate using Fisher's exact test. This validates that decoupling cellular response from infected proportion bias enhances identification of biologically perturbation-related genes.

## When to use

Use when scCRISPR-seq analysis needs to judge whether decoupled perturbation responses (vs the naive OC estimate) yield better pathway enrichment and perturbation-related gene rankings, e.g. for TCR-related genes like CD69 and RELA on Jurkat CROP-seq data.

## When NOT to use

- The sample is an intricate system or disease tissue such as a tumor, where the Gaussian mixture model assumption underlying scDecouple's decoupling may not hold
- Rare cell types are completely absent from infection, violating the assumption that NT and perturbation groups share the same cell clusters
- Cell populations show extreme rather than moderate heterogeneity, breaching the assumption of similar cellular responses across cell clusters

## Inputs

- res.step1 objects from data_preprocessing: Z.pca.results$rotation, Z.norm.log.sub, Y.norm.log.sub, genes.variable
- res.step3 object from scDecouple decoupling step: beta.PC.scDecouple
- Perturbation matrix Y and control matrix Z (cell-by-gene), e.g. Jurkat CROP-seq data for 22 target genes restricted to >60 infected cells
- degenes.num parameter (default 1500 top DE genes for GO enrichment)

## Outputs

- Gene rankings per gRNA for the decoupled cellular response (with positions of target genes such as CD69 and RELA extractable)
- Gene Ontology enrichment results for the decoupled responses (via enrichR)
- Fisher's exact test comparison of scDecouple vs OC pathway enrichment
- Average gene-ranking gain metrics versus OC (reported: 280 and 125 on Jurkat)

## How to apply

Run the scDecouple pipeline to step 3, then call downstream_analysis passing Z.pca.rotation, beta.PC.scDecouple, Z.norm.log.sub, Y.norm.log.sub and genes.variable from the previous steps, with degenes.num=1500 to select the top 1500 DE genes for GO enrichment computation (via enrichR). Extract the gene rankings and GO enrichment results for the decoupled responses. Compare scDecouple pathway enrichment against the OC baseline using Fisher's exact test, and additionally read the positions of known perturbation-related genes (CD69, RELA for the Jurkat TCR pathway case) from the rankings. Success is judged by average ranking gains over OC (reported: 280 and 125 across the Jurkat gRNAs) and enrichment statistics favoring the decoupled response.

## Related tools

- **scDecouple** (R package whose downstream_analysis function computes gene rankings and GO enrichment from the decoupled responses) — https://github.com/MengQiuchen/scDecouple
- **enrichR** (R library used for Gene Ontology enrichment calculation in the downstream step)
- **R** (Environment in which the scDecouple package and downstream enrichment analysis are executed)

## Examples

```
res.step4 <- downstream_analysis(Z.pca.rotation=res.step1$Z.pca.results$rotation, beta.PC.scDecouple=res.step3$beta.PC.scDecouple, Z.norm.log.sub=res.step1$Z.norm.log.sub, Y.norm.log.sub=res.step1$Y.norm.log.sub, genes.variable=res.step1$genes.variable)
```

## Evaluation signals

- The results include a Fisher's exact test comparing scDecouple GO enrichment against the OC baseline
- The results table contains extractable CD69 and RELA positions from the gene rankings
- Average gene-ranking gains over OC are consistent with the reported increases of 280 and 125 on the Jurkat dataset
- GO enrichment was computed on the top degenes.num=1500 DE genes (default), matching the README's documented parameter
- Gene rankings are derived from beta.PC.scDecouple rotated back through Z.pca.rotation and restricted to genes.variable

## Limitations

- GMM assumption may not hold in intricate systems or disease tissues such as tumors
- Assumption of same cell clusters between NT and perturbation groups may fail when rare cell types are completely absent from infection
- Assumption of similar cellular responses across cell clusters limits scDecouple to cells with moderate heterogeneity

## Evidence

- [methods] GO enrichment from downstream_analysis call: "## step4: downstream analysis: gene ranking and pathway enrichment
res.step4 <- downstream_analysis(Z.pca.rotation = res.step1$Z.pca.results$rotation, beta.PC.scDecouple ="
- [abstract] biologically perturbation-related genes: "By applying scDecouple to real scCRISPR-seq data, we found that scDecouple enhances the identification of biologically perturbation-related genes."
- [discussion] GO enrichment plots among package outputs: "including PC plots, PC variance and multimodality visualizations, cellular response heatmaps and Gene Ontology enrichment plots"
- [readme] degenes.num parameter for GO enrichment: "degenes.num` (optional): The number of differentially expressed genes (DE genes) for GO enrichment calculation. The default is 1500."
