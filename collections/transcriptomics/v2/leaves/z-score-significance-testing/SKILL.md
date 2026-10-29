---
name: z-score-significance-testing
description: 'Use when building a sample-specific (personalized) differential co-expression
  network: you have normal and tumor expression matrices and need to test, per patient,
  whether each gene-gene edge''s Δr_ij is statistically significant (P < 0.05).'
when_to_use_negative:
- The goal is a group-level (population) differential co-expression network rather
  than a sample-specific one — WMDS.net's standard differential mode handles two physiological
  states directly without appending a single tumor sample
- Only one condition's expression matrix is available, so no Δr_ij between normal
  reference and tumor-perturbed networks can be computed
- Candidate gene pairs should not be restricted to the Gene_Net/DawnRank-T CGA hub
  network (e.g. genome-wide co-expression analysis)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: WMDS.net
  role: Provides the SSN (netP) mode in which per-patient Δr_ij edge differences are
    Z-score transformed and Z-tested to build the personalized differential co-expression
    network
  repo: github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation language; WMDS.net's algorithm (including the intlinprog-based
    optimization step) runs on Matlab R2020b with the Optimization ToolBox
- name: DawnRank
  role: Source of the gene-interaction network whose intersection with TCGA genes
    forms the Gene_Net hub network that restricts candidate gene pairs before Z-testing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:9873552495af3345f24a1c184da949307afdb3ab2c9e349a03ec6713dab7582a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# z-score-significance-testing

## Summary

In WMDS.net's sample-specific network (SSN) mode, per-patient correlation edge differences (Δr_ij between a normal-sample co-expression reference network and a tumor-perturbed network) are converted to Z-scores and assessed by Z-test; edges with P < 0.05 define a patient's personalized differential co-expression network.

## When to use

Use when building a sample-specific (personalized) differential co-expression network: you have normal and tumor expression matrices and need to test, per patient, whether each gene-gene edge's Δr_ij is statistically significant (P < 0.05).

## When NOT to use

- The goal is a group-level (population) differential co-expression network rather than a sample-specific one — WMDS.net's standard differential mode handles two physiological states directly without appending a single tumor sample
- Only one condition's expression matrix is available, so no Δr_ij between normal reference and tumor-perturbed networks can be computed
- Candidate gene pairs should not be restricted to the Gene_Net/DawnRank-T CGA hub network (e.g. genome-wide co-expression analysis)

## Inputs

- Normal-sample expression matrix (rows = gene symbols, columns = sample IDs, e.g. TCGA_BLCAnormal)
- Tumor-sample expression matrix (rows = gene symbols, columns = sample IDs, e.g. TCGA_BLCAtumor)
- Gene_Net hub reference network (intersection of DawnRank gene-interaction network with TCGA genes) to restrict candidate gene pairs

## Outputs

- personalized_ssn_edges.csv with columns patient_id, gene_i, gene_j, delta_r, p_value — one per-patient edge list containing only edges with P-value < 0.05

## How to apply

Load the normal-sample and tumor-sample expression matrices (rows = gene symbols, columns = sample IDs) and restrict candidate gene pairs to the Gene_Net hub reference network (intersection of the DawnRank gene-interaction network with TCGA genes). Compute Pearson correlations r_ij^n across all n normal samples to build the co-expression reference network. For each tumor sample individually, append it to the normal reference samples, recompute the correlation r_ij^(n+1) to form the perturbed network, and take Δr_ij = r_ij^(n+1) − r_ij^n for every edge. Convert each Δr_ij to a Z-score, derive a P-value by Z-test, and retain edges with P-value < 0.05 for that patient, writing one per-patient edge list (patient_id, gene_i, gene_j, delta_r, p_value) to personalized_ssn_edges.csv. The Z-test on Δr is what turns a population-level differential co-expression framework into a single-patient network in WMDS.net's netP mode.

## Related tools

- **WMDS.net** (Provides the SSN (netP) mode in which per-patient Δr_ij edge differences are Z-score transformed and Z-tested to build the personalized differential co-expression network) — github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation language; WMDS.net's algorithm (including the intlinprog-based optimization step) runs on Matlab R2020b with the Optimization ToolBox)
- **DawnRank** (Source of the gene-interaction network whose intersection with TCGA genes forms the Gene_Net hub network that restricts candidate gene pairs before Z-testing)

## Evaluation signals

- personalized_ssn_edges.csv exists with exactly the columns patient_id, gene_i, gene_j, delta_r, p_value
- Every retained edge has p_value < 0.05 (the Z-test retention threshold)
- Edge sets differ across patients, as expected for sample-specific networks (each patient's tumor sample perturbs the reference correlations differently)
- Each retained edge carries a plausible Δr_ij = r_ij^(n+1) − r_ij^n computed from the appended-sample Pearson recorrelation
- Candidate gene pairs fall within the Gene_Net hub reference network

## Limitations

- Requires paired normal and tumor expression matrices with gene symbols and sample IDs; a single tumor sample must be appended to the normal reference set to compute the perturbed correlation
- Candidate edges are restricted to the Gene_Net hub network (DawnRank interactions ∩ TCGA genes), so off-network gene pairs are never Z-tested
- The 0.05 Z-test P-value cutoff is the fixed retention threshold; edges with marginal Δr significance are discarded
- The WMDS.net pipeline is Matlab-specific (R2020b, Optimization ToolBox), so the Z-testing step inherits that runtime dependency

## Evidence

- [methods] SSN Z-test on Δr: [evidence span withheld — anchor status: not_found]
- [methods] Perturbed-network construction: [evidence span withheld — anchor status: not_found]
- [readme] README: algorithm implementation: "we implement the algorithm using the Matlab program language and solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab"
- [readme] README: input format: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
- [intro] Article intro: network restriction: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
