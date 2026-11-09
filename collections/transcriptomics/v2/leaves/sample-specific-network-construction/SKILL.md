---
name: sample-specific-network-construction
description: Use when per-patient (personalized) co-expression networks are needed
  from TCGA RNA-seq expression matrices, e.g. before running per-patient WMDS.netP
  driver-gene identification.
when_to_use_negative:
- You need a single global differential co-expression network across all tumors (use
  the WMDS.net two-condition differential network mode instead of the per-patient
  SSN mode).
- Input expression matrices are not aligned to TCGA gene symbols or the Gene_Net candidate
  pairs cannot be intersected with your genes.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: WMDS.net
  role: Implements the SSN (WMDS.netP) mode that builds the per-patient perturbed
    co-expression network from Delta-r Z-scores
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Language in which the algorithm and downstream intlinprog-based weighted MDS
    solve are implemented (Optimization ToolBox)
- name: DawnRank
  role: Source of the gene-interaction network intersected with TCGA genes to form
    the Gene_Net candidate-pair restriction
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:bbcf0cf761404d4274e7e116525f82230f94a2e426108ce3d17c08e6ca4a6d13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sample-specific-network-construction

## Summary

Constructs a personalized (sample-specific) differential co-expression network for each tumor patient by appending that patient's tumor sample to the TCGA normal-sample reference set, recomputing Pearson correlations, and retaining edges with significant correlation shift (Z-test P-value < 0.05).

## When to use

Use when per-patient (personalized) co-expression networks are needed from TCGA RNA-seq expression matrices, e.g. before running per-patient WMDS.netP driver-gene identification.

## When NOT to use

- You need a single global differential co-expression network across all tumors (use the WMDS.net two-condition differential network mode instead of the per-patient SSN mode).
- Input expression matrices are not aligned to TCGA gene symbols or the Gene_Net candidate pairs cannot be intersected with your genes.

## Inputs

- Normal-sample expression matrix (rows = gene symbols, columns = sample IDs)
- Tumor-sample expression matrix (rows = gene symbols, columns = sample IDs), e.g. TCGA_BLCAtumor / TCGA_BLCAnormal in the tested data
- Gene_Net hub reference network (intersection of the DawnRank gene-interaction network with TCGA genes)

## Outputs

- Per-patient edge list personalized_ssn_edges.csv with columns patient_id, gene_i, gene_j, delta_r, p_value

## How to apply

Load the normal-sample and tumor-sample expression matrices (rows = gene symbols, columns = sample IDs) and the Gene_Net hub reference network (intersection of the DawnRank gene-interaction network with TCGA genes) to restrict candidate gene pairs. Compute Pearson correlation coefficients r_ij^n across all n normal samples to build the co-expression reference network. For each tumor sample individually, append it to the normal reference samples, recompute the correlation r_ij^(n+1) to form the perturbed network, and take Delta-r_ij = r_ij^(n+1) - r_ij^n for every edge. Convert each Delta-r_ij to a Z-score, obtain a P-value by Z-test, and retain only edges with P-value < 0.05 for that patient. This yields a unique differential network per patient, which downstream WMDS.netP converts into a weighted minimum dominating set problem solved with intlinprog in Matlab R2020b.

## Related tools

- **WMDS.net** (Implements the SSN (WMDS.netP) mode that builds the per-patient perturbed co-expression network from Delta-r Z-scores) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Language in which the algorithm and downstream intlinprog-based weighted MDS solve are implemented (Optimization ToolBox))
- **DawnRank** (Source of the gene-interaction network intersected with TCGA genes to form the Gene_Net candidate-pair restriction)

## Evaluation signals

- personalized_ssn_edges.csv exists with the required columns patient_id, gene_i, gene_j, delta_r, p_value
- Every retained edge has p_value < 0.05 by the Delta-r Z-test
- Edge sets differ across patients, confirming the networks are sample-specific rather than identical per-tumor edge lists
- Candidate gene pairs are restricted to Gene_Net (the DawnRank x TCGA gene intersection)

## Limitations

- Perturbed correlation is computed by appending one tumor sample to the normal reference set, so per-patient Delta-r depends on reference set size n; small reference cohorts make the Z-test P-values less stable.
- Personalized driver genes recovered downstream are mostly low-frequency (< 0.3) across patients, so this per-patient procedure trades reproducibility for capturing intra-cancer-type heterogeneity.

## Evidence

- [intro] Construct per-patient SSN by appending the tumor sample to the normal reference and recomputing correlations: [evidence span withheld — anchor status: not_found]
- [intro] Z-test P < 0.05 edge retention: [evidence span withheld — anchor status: not_found]
- [intro] WMDS.net model description from paper intro: "We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks."
- [readme] README: Matlab intlinprog implementation and input format: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] README: input matrix orientation: "The rows of the expression matrices represent gene expression, and the columns represent samples."
