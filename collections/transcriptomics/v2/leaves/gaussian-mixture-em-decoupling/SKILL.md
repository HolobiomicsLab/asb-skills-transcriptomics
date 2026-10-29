---
name: gaussian-mixture-em-decoupling
description: Use when analyzing scCRISPR-seq cell-by-gene matrices where perturbation
  (Y) and control (Z) groups have differing cluster proportions, so naive fold-change
  estimation is distorted by infected proportion bias and the true cellular response
  must be decoupled.
when_to_use_negative:
- Samples where the Gaussian mixture assumption on gene expression distributions does
  not hold, e.g. intricate systems or disease tissues such as tumors
- Datasets where some rare cell types are completely absent from infection, so cell
  clusters are not shared between NT and perturbation groups
- Cells with extreme heterogeneity — scDecouple's assumption of similar cellular responses
  across clusters suits only moderate heterogeneity
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: R package implementing the Gaussian mixture / EM decoupling (run_scDecouple,
    data_preprocessing, pc_selection, scDecouple, downstream_analysis)
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: R library for EM fitting of mixture models, loaded in the scDecouple demo
    workflow
- name: diptest
  role: Provides the dip statistic used by pc_selection to threshold multimodal PCs
    (top.multimodal=3)
- name: devtools
  role: Used to install the developmental scDecouple package from GitHub
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:5b03f08873454f65aae69f54ade3ab8f29bd1473f9f5d40c38445ed50cd049fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - devtools
  - diptest
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

# gaussian-mixture-em-decoupling

## Summary

Fit a Gaussian mixture model to perturbed-cell expression profiles and iteratively find the maximum likelihood of cell cluster proportions and the cellular response per gRNA via an EM algorithm, decoupling true perturbation response from infected proportion bias in scCRISPR-seq data.

## When to use

Use when analyzing scCRISPR-seq cell-by-gene matrices where perturbation (Y) and control (Z) groups have differing cluster proportions, so naive fold-change estimation is distorted by infected proportion bias and the true cellular response must be decoupled.

## When NOT to use

- Samples where the Gaussian mixture assumption on gene expression distributions does not hold, e.g. intricate systems or disease tissues such as tumors
- Datasets where some rare cell types are completely absent from infection, so cell clusters are not shared between NT and perturbation groups
- Cells with extreme heterogeneity — scDecouple's assumption of similar cellular responses across clusters suits only moderate heterogeneity

## Inputs

- Y: perturbation-group cell-by-gene count matrix (scCRISPR-seq)
- Z: control-group (unperturbed or NT gRNA) cell-by-gene count matrix
- Optional parameters: using_rank, top.sdev=30, top.multimodal=3, th.sdev, th.multimodal, degenes.num=1500, seed_=0

## Outputs

- Selected PCs with high potential infection bias (pc_selection result)
- beta.PC.scDecouple: estimated cellular response per gRNA in PC space
- Decoupled cluster proportions vs cellular response per gRNA
- Gene ranking and pathway (GO) enrichment from downstream_analysis

## How to apply

First preprocess the perturbation group Y and control group Z (cell-by-gene count matrices) with log-normalization and highly variable gene detection via data_preprocessing(Y, Z). Select PCs that may carry high infection bias with pc_selection using rank-based selection (using_rank=TRUE, top.sdev=30, top.multimodal=3 on dip statistics). Then run the EM-based decoupling step scDecouple() on the Z PCA coordinates/rotation and log-normalized Y, which models gene expression distributions as a Gaussian mixture and iteratively maximizes likelihood over cluster proportions and the cellular response for each gRNA (seed_=0 for reproducibility). Finally map beta.PC.scDecouple back to genes via downstream_analysis for gene ranking and GO enrichment (degenes.num=1500), or run everything at once with run_scDecouple(Y, Z). The rationale is that cluster-proportion differences between perturbation and control groups confound fold-change-based effect estimates, and the mixture model separates the response direction from the composition bias.

## Related tools

- **scDecouple** (R package implementing the Gaussian mixture / EM decoupling (run_scDecouple, data_preprocessing, pc_selection, scDecouple, downstream_analysis)) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (R library for EM fitting of mixture models, loaded in the scDecouple demo workflow)
- **diptest** (Provides the dip statistic used by pc_selection to threshold multimodal PCs (top.multimodal=3))
- **devtools** (Used to install the developmental scDecouple package from GitHub)

## Examples

```
library(scDecouple); res <- run_scDecouple(Y, Z, using_rank=TRUE, top.sdev=30, top.multimodal=3, degenes.num=1500, seed_=0)
```

## Evaluation signals

- PC selection flags a small set of multimodal PCs (dip statistic above top.multimodal=3 when using_rank=TRUE) that plausibly carry infection bias
- On synthetic two-cluster benchmarks, estimated cellular response on the response-bearing PC 1 correlates with ground truth (OC stays accurate) even as infected proportion bias (d, r) increases, unlike naive observed change
- Estimation error versus ground-truth simulated responses is lower and more stable than a conventional cluster-annotate-map baseline across covariance angles, cluster distances, and infection efficiencies (100 runs per two-cluster setting, 50 per four-cluster/10-gRNA setting)
- Each EM run is seeded (seed_=0) so repeated runs give reproducible beta.PC.scDecouple estimates

## Limitations

- The Gaussian mixture assumption holds well in certain cell lines and tissues but might not remain in intricate systems or disease tissues, such as tumors
- The same-cell-clusters assumption might fail when some rare cell types are completely absent from infection
- The similar-cellular-response-across-clusters assumption limits scDecouple to cells with moderate heterogeneity

## Evidence

- [intro] scDecouple leverages a Gaussian mixture model to depict the distribution of gene: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation. scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [readme] README usage of run_scDecouple: "res <- run_scDecouple(Y, Z)"
- [readme] Parameter defaults for rank-based PC selection: "`using_rank` (optional): A logical value indicating whether to use rank-based selection of principal components (PCs) or value-based selection. The default is TRUE."
- [methods] Step-wise pipeline in methods: "## step3: decouple cellular response and infection bias"
- [discussion] Limitation on GMM assumption: "The first assumption holds well in certain cell lines and tissues, but it might not remain in intricate systems or disease tissues, such as tumors"
