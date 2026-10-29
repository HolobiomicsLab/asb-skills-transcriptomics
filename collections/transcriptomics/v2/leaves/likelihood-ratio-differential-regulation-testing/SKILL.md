---
name: likelihood-ratio-differential-regulation-testing
description: Use when comparing gene regulatory networks between two conditions from
  paired single-cell data (one scRNA-seq + one scATAC-seq per condition) and you need
  subpopulation-specific differential regulation testing with p-values and FDR control.
when_to_use_negative:
- Conditions comprise multiple donors per condition — the method does not yet accommodate
  multiple donors under each condition
- Only scRNA-seq data are available without paired scATAC-seq — use the scRNA-only
  baseline instead (its AUC is lower, 0.7561 vs 0.9157, but sc-compReg requires accessibility
  data)
- The goal is de novo network inference from expression alone rather than testing
  differential regulation — alternative pipelines (GENIE3/PIDC crossed with diffK/diffRank)
  may be more appropriate
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scCompReg
  role: R package providing the sc_compreg function that computes TFRP, fits the linear
    models, and returns the likelihood-ratio differential-regulation output with p-values
    and FDR control
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment in which the scCompReg package is installed from source
    and executed
- name: Bedtools
  role: Linux dependency used in preprocessing (peak-name intersection, peak-gene
    prior preparation) prior to calling sc_compreg
- name: Homer
  role: Linux dependency for motif-based preprocessing steps supporting the TFRP computation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:47bc9ad71e741e1c06ecffa1a551940c4572444b5496024caef0685a43b483f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - GENIE3
  - Homer
  - PIDC
  - R (>= 3.6.0)
  - R (>=)
  - sc-compReg
  - scCompReg
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# likelihood-ratio-differential-regulation-testing

## Summary

sc-compReg's statistical test for differential TF-TG regulatory relations between two linked subpopulations: it computes cell-specific transcription factor regulatory potential (TFRP), fits a linear regression of TFRP on target gene expression per condition, and uses a likelihood-ratio statistic with p-value computation and FDR control to detect edges whose regulation changed between conditions.

## When to use

Use when comparing gene regulatory networks between two conditions from paired single-cell data (one scRNA-seq + one scATAC-seq per condition) and you need subpopulation-specific differential regulation testing with p-values and FDR control.

## When NOT to use

- Conditions comprise multiple donors per condition — the method does not yet accommodate multiple donors under each condition
- Only scRNA-seq data are available without paired scATAC-seq — use the scRNA-only baseline instead (its AUC is lower, 0.7561 vs 0.9157, but sc-compReg requires accessibility data)
- The goal is de novo network inference from expression alone rather than testing differential regulation — alternative pipelines (GENIE3/PIDC crossed with diffK/diffRank) may be more appropriate

## Inputs

- Paired single-cell samples per condition: scRNA-seq expression (O1/O2) and scATAC-seq accessibility (E1/E2) matrices with cell-to-cluster indices (O1.idx/E1.idx) and gene symbols/peak names (symbol1, peak.name1), e.g. sample1.rds and sample2.rds
- Species motif prior: prior_data/motif_human.rds or motif_mouse.rds with motif.name, motif.weight, match2 fields
- Motif target file loaded via mfbs_load (motif_file.rds)
- Intersected peak names file (PeakName_intersect.txt) and peak-gene prior (peak_gene_prior_intersect.bed) generated for genome version hg19/hg38/mm9/mm10

## Outputs

- Per-subpopulation hub TF tables: compreg.output$hub.tf written to tf_<i>.txt for i = 1..n.pops (tab-delimited, no quotes/row names)
- Per-subpopulation differential regulatory network tables: compreg.output$diff.net written to diff_net_<i>.txt containing TF-target edges with differential-regulation scores
- Likelihood-ratio statistics with p-values and FDR-controlled differential regulation calls

## How to apply

First identify linked subpopulations (e.g. via cnmf clustering) in each condition and prepare the intersected peak-name and peak-gene prior files (PeakName_intersect.txt, peak_gene_prior_intersect.bed) using sc_compreg_process_data_.sh against prior_data for the correct genome version (hg19, hg38, mm9, or mm10). In R (>= 3.6.0), load paired samples (O1/E1/O1.idx/E1.idx/symbol1/peak.name1 and O2 equivalents), the species motif prior (motif_human.rds or motif_mouse.rds), and the motif target file loaded via mfbs_load. Call sc_compreg, which computes per-TF/per-TG TFRP scores by combining TF motif activity with the peak-gene coupling matrix and chromatin accessibility, then fits the linear model P(TFRP|TG) per condition and computes a likelihood-ratio statistic testing the null that the regression model is identical across the two conditions, with p-value computation and FDR control. Extract compreg.output$hub.tf per subpopulation to tf_<i>.txt and compreg.output$diff.net to diff_net_<i>.txt. The method is validated on simulated two-condition data against a scRNA-only baseline (AUC 0.9157 vs 0.7561 in the combined differential-regulation scenario), so benchmark outputs against known ground truth when available.

## Related tools

- **scCompReg** (R package providing the sc_compreg function that computes TFRP, fits the linear models, and returns the likelihood-ratio differential-regulation output with p-values and FDR control) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment in which the scCompReg package is installed from source and executed)
- **Bedtools** (Linux dependency used in preprocessing (peak-name intersection, peak-gene prior preparation) prior to calling sc_compreg)
- **Homer** (Linux dependency for motif-based preprocessing steps supporting the TFRP computation)

## Examples

```
compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, sample2$E2, sample2$O2.idx, sample2$E2.idx, sample2$symbol2, sample2$peak.name2, motif$motif.name, motif$motif.weight, motif$match2, motif.file, sep.char=' ')
```

## Evaluation signals

- On simulated two-condition scRNA-seq/scATAC-seq data with ground-truth differential regulation, the sc-compReg LR statistic should achieve AUCs of approximately 0.9802, 0.9972, 0.8124, and 0.9157 across the four differential-regulation scenarios, versus 0.7561 for the scRNA-only baseline in the combined scenario
- One tf_<i>.txt and one diff_net_<i>.txt file exist for every subpopulation i = 1..compreg.output$n.pops, each readable as a tab-delimited table
- diff_net tables contain TF-target regulatory edges with differential-regulation scores (non-empty, correct columns)
- Alternatives benchmark (GENIE3/PIDC crossed with diffK/diffRank, 8 pipelines) should not exceed the sc-compReg LR AUC under the combined differential-regulation scenario if replicating the validation

## Limitations

- Differential regulations discovered on a single patient/control pair will need to be replicated by additional patients and controls
- The method does not yet accommodate multiple donors per condition; extension awaits data availability
- sc-compReg does not yet support analysis incorporating bulk HiChIP data (planned extension)
- Subpopulation linking (clustering, embedding, matching across conditions) must be performed first; differential testing depends on consistent cluster assignments

## Evidence

- [intro] LR statistic rationale: "we present a new statistic for testing differential regulatory relations across two linked subpopulations"
- [intro] p-value and FDR: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [results] simulation validation result: [evidence span withheld — anchor status: not_found]
- [readme] installation: "devtools::install_github("SUwonglab/sc-compReg", ref="master", subdir="R_package")"
- [readme] data acquisition: "git lfs clone https://github.com/SUwonglab/sc-compReg.git"
