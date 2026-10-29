---
name: pluripotency-discrimination-assessment
description: Use when single-cell RNA-Seq profiles (e.g. hESC vs progenitor/differentiated
  populations) must be ranked or classified by differentiation potency without marker
  genes, feature selection, or training labels.
when_to_use_negative:
- The task is pseudo-time ordering or trajectory inference with tools such as Monocle
  or MPath, which the paper notes cannot readily obtain this network-entropy potency
  insight.
- No matched PPI/connectome network is available for the expression matrix, since
  SR's discrimination depends on network-constrained expression patterns (reshuffling
  over the network abolishes it).
- The analysis requires potency estimates derived from prior biological knowledge
  such as marker expression or timepoint, which SCENT is explicitly designed to avoid.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: R package that computes the normalized signalling entropy rate (SR) per cell
    by integrating the scRNA-Seq expression matrix with the PPI connectome; installed
    from GitHub
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment for running SCENT and evaluating discrimination (Spearman correlation,
    ROC AUC, Wilcoxon test) and the network-reshuffling permutations
- name: CCAT
  role: Companion tool paired with SCENT in the paper's tutorial for estimating differentiation
    potency of single cells
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:915725e8dd3d989dff77f26f6cd91b0b6cf49fb39f0d18c6adfd9db9bb344a36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - CCAT
  - R
  - SCENT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pluripotency-discrimination-assessment

## Summary

Computes the normalized signalling entropy rate (SR) per single cell with the SCENT R package by integrating an scRNA-Seq expression matrix with a PPI connectome, then tests whether SR alone discriminates pluripotent from non-pluripotent cells (Chu et al.: Spearman 0.91, ROC AUC 0.96) without feature selection or training.

## When to use

Use when single-cell RNA-Seq profiles (e.g. hESC vs progenitor/differentiated populations) must be ranked or classified by differentiation potency without marker genes, feature selection, or training labels.

## When NOT to use

- The task is pseudo-time ordering or trajectory inference with tools such as Monocle or MPath, which the paper notes cannot readily obtain this network-entropy potency insight.
- No matched PPI/connectome network is available for the expression matrix, since SR's discrimination depends on network-constrained expression patterns (reshuffling over the network abolishes it).
- The analysis requires potency estimates derived from prior biological knowledge such as marker expression or timepoint, which SCENT is explicitly designed to avoid.

## Inputs

- Single-cell RNA-Seq expression matrix (e.g. Chu et al., 1,018 cells spanning pluripotent hESC and non-pluripotent progenitor/differentiated classes)
- Human protein-protein interaction (connectome) network compatible with SCENT
- Per-cell class labels or pluripotency scores for evaluating discrimination (pluripotent vs non-pluripotent)
- Developmental-stage annotations (e.g. Treutlein et al. time-course) when testing the monotonic SR decrease with stage

## Outputs

- Per-cell normalized signalling entropy rate (SR) values with class labels (e.g. chu_scent_sr.csv)
- ROC AUC and Wilcoxon test statistics for pluripotent vs non-pluripotent discrimination
- Spearman correlation between SR and the pluripotency score
- permutation_significance_results.csv with aggregated network-reshuffling permutation distributions

## How to apply

Install the SCENT R package via devtools::install_github('aet21/SCENT') and load the scRNA-Seq expression matrix (e.g. the Chu et al. matrix of 1,018 hESC and progenitor/differentiated cells) together with the human PPI connectome. Run SCENT to compute the normalized signalling entropy rate (SR) for every cell by integrating the expression matrix with the PPI network, deliberately without feature selection or prior biological knowledge such as marker expression or timepoint. Correlate per-cell SR with the pluripotency score (expect Spearman 0.91 on Chu) and compute the ROC AUC for pluripotent versus non-pluripotent discrimination (expect AUC 0.96, Wilcoxon P<1e-300). As a network-dependence ablation, randomly reshuffle gene expression values over the PPI network — 100 permuted datasets for Chu and 1,000 for Treutlein — recompute SR in each permutation, and confirm that pluripotency discrimination and the monotonic decrease of SR with developmental stage are abolished under the null. Write per-cell SR values and class labels to chu_scent_sr.csv and the aggregated permutation distributions to permutation_significance_results.csv.

## Related tools

- **SCENT** (R package that computes the normalized signalling entropy rate (SR) per cell by integrating the scRNA-Seq expression matrix with the PPI connectome; installed from GitHub) — https://github.com/aet21/SCENT
- **R** (Environment for running SCENT and evaluating discrimination (Spearman correlation, ROC AUC, Wilcoxon test) and the network-reshuffling permutations)
- **CCAT** (Companion tool paired with SCENT in the paper's tutorial for estimating differentiation potency of single cells)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- Spearman correlation between per-cell SR and the pluripotency score reaches 0.91 on the Chu et al. dataset.
- ROC AUC for pluripotent hESC versus non-pluripotent progenitor discrimination equals 0.96 (Wilcoxon P<1e-300).
- chu_scent_sr.csv contains one SR value plus class label per cell (1,018 Chu cells), with SR computed without feature selection or prior biological knowledge.
- Network-reshuffling null: after 100 (Chu) and 1,000 (Treutlein) permutations of expression over the PPI network, SR loses pluripotency discrimination power and its monotonic decrease with developmental stage.
- permutation_significance_results.csv summarizes the null permutation distributions for both datasets as specified in the workflow.

## Limitations

- Dependence on the PPI connectome: randomly reshuffling expression values over the network abolishes SR's discrimination power, so network coverage/quality directly conditions performance.
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested, pending high-resolution time-course and genome-wide scRNA-Seq data.
- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established.

## Evidence

- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [abstract] Signalling entropy provides a more accurate and robust potency estimate than other entropy-based measures, driven in part by a subtle positive correlation between the transcriptome and connectome: [evidence span withheld — anchor status: not_found]
- [methods] A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
