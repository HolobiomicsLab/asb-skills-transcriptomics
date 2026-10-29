---
name: egfr-pathway-connectivity-analysis
description: Use when bulk or single-sample RNA-seq data (e.g., TCGA-LUAD tumor and
  adjacent-normal samples) must be converted into per-sample gene-gene association
  networks so that pathway-level connectivity (e.g., EGFR pathway, EGF/EGFR degrees)
  can be compared sample-by-sample.
when_to_use_negative:
- The goal is causal gene regulation — c-CSN explicitly does not construct causal
  networks and cannot give directions of gene associations
- Only a single aggregated cohort-level network is needed rather than per-sample/per-cell
  networks — a standard co-expression or CSN analysis avoids the G-fold computational
  overhead
- The input is already a network/degree matrix and no per-sample network construction
  is required
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: 'Core method: constructs one conditional cell-specific network (CCSN) per
    individual sample from RNA-seq expression data, enabling per-sample EGFR pathway
    degree extraction'
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:56cdaf7a8ac9bab8464e3af8dba71a5065b48dfe87f27338efd783cfa2d11df9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# egfr-pathway-connectivity-analysis

## Summary

Constructs a conditional cell-specific network (CCSN) for each individual TCGA-LUAD sample (tumor and normal) using the c-CSN method, then compares EGFR pathway degree density between tumor and benign tissues to reproduce the finding that EGFR is densely connected only in tumors.

## When to use

Use when bulk or single-sample RNA-seq data (e.g., TCGA-LUAD tumor and adjacent-normal samples) must be converted into per-sample gene-gene association networks so that pathway-level connectivity (e.g., EGFR pathway, EGF/EGFR degrees) can be compared sample-by-sample.

## When NOT to use

- The goal is causal gene regulation — c-CSN explicitly does not construct causal networks and cannot give directions of gene associations
- Only a single aggregated cohort-level network is needed rather than per-sample/per-cell networks — a standard co-expression or CSN analysis avoids the G-fold computational overhead
- The input is already a network/degree matrix and no per-sample network construction is required

## Inputs

- TCGA LUAD RNA-seq expression matrix (tumor and adjacent-normal samples, from GDC portal), normalized
- EGFR pathway gene membership list (including EGF and EGFR)
- Significance level α for the conditional independence hypothesis test
- Number of conditional genes G (top-degree hub genes selected from the CSN)

## Outputs

- One CCSN (direct association network) per individual sample
- Conditional network degree matrix (CNDM) with per-sample gene-gene interaction degrees
- Per-sample EGF/EGFR network degrees
- EGFR pathway subnetwork connectivity density per sample (tumor vs. normal comparison plots analogous to Figures S8B and S8C)

## How to apply

Download and normalize TCGA LUAD RNA-seq expression profiles (tumor and adjacent normal) from the GDC portal. For each sample, run the c-CSN procedure: select conditional genes as the top-degree hub genes from the CSN (choosing the top G largest 'importance' genes, e.g., EGF/EGFR-relevant regulators), estimate the conditional independence statistic ρxy|z via scatter-diagram neighborhoods in 3D expression space, and test each gene pair at significance level α against the Nα quantile of the standard normal distribution to build the CCSN. Rationale: plain CSN overestimates gene-gene associations by including indirect effects, so the conditional test isolates direct associations, yielding one direct association network per cell/sample. Integrate the per-sample CCSNs into a conditional network degree matrix (CNDM) whose entries cijk are the degrees of the gene-gene interaction network of cell k. Extract EGF and EGFR degrees and the EGFR pathway subnetwork degree density from each per-sample CCSN, compare tumor vs. normal distributions, and render representative single-sample EGFR networks mirroring Figures S8B and S8C.

## Related tools

- **c-CSN** (Core method: constructs one conditional cell-specific network (CCSN) per individual sample from RNA-seq expression data, enabling per-sample EGFR pathway degree extraction) — https://github.com/LinLi-0909/c-CSN

## Evaluation signals

- Exactly one direct association network (CCSN) is produced per individual sample/cell
- Per-sample EGF and EGFR CCSN degrees are consistently higher across tumor samples than across normal/benign samples
- EGFR pathway subnetwork degree density is significantly higher in tumor samples and not densely connected in normal tissues, mirroring the published Figure S8B/S8C pattern
- Each accepted gene-gene edge satisfies the hypothesis test at significance level α against the Nα quantile of the standard normal distribution for the conditional statistic ρxy|z
- The integrated CNDM is an m × n matrix (genes × samples) whose entries are network degrees derived from the per-sample CCSNs

## Limitations

- c-CSN is not designed to construct causal gene association networks, and directions of gene associations cannot be obtained
- No parallel computation scheme is provided despite the G-fold computational cost increase from using G conditional genes per sample
- The method is demonstrated on RNA-seq/scRNA-seq expression data (e.g., TCGA LUAD, Wang, Yang datasets); behavior on other data modalities is not established

## Evidence

- [abstract] One direct association network per sample: "1) One direct association network is generated for one cell."
- [intro] c-CSN removes indirect associations that inflate CSN: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [methods] Conditional genes selected as top hub genes: [evidence span withheld — anchor status: not_found]
- [methods] Application to TCGA LUAD RNA-seq: "we applied c-CSN to the TCGA lung adenocarcinoma (LUAD) RNA-seq dataset"
- [methods] Degree matrix integration of CCSNs: [evidence span withheld — anchor status: not_found]
