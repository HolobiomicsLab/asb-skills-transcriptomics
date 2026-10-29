---
name: pearson-correlation-computation
description: Use when you have two condition-specific expression matrices (rows =
  gene symbols, columns = sample IDs) and need per-patient differential co-expression
  edges for downstream WMDS.net driver-node ranking.
when_to_use_negative:
- Input is a single expression matrix with no normal/tumor contrast — the Δr_ij computation
  requires a pooled normal reference set against which each tumor sample is compared.
- The goal is group-level differential co-expression rather than sample-specific (personalized)
  networks — appending one tumor sample at a time is specific to the SSN mode.
- The gene space has no overlap with the Gene_Net hub network (DawnRank interactions
  ∩ TCGA genes), since candidate pairs are restricted to that reference.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: WMDS.net
  role: Framework consuming the Pearson-correlation-derived differential co-expression
    network to find driver nodes; implementation whose SSN mode defines the Δr_ij
    → Z-score → P-value workflow
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Language in which the WMDS.net algorithm is implemented; its Optimization
    ToolBox intlinprog function solves the downstream binary integer-programming problem
- name: DawnRank
  role: Source of the gene-interaction network used to build the Gene_Net hub reference
    that restricts candidate gene pairs for correlation computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:662b9ef49777d4a5a7085e7c3d7bf2ddaf1816079bca5fcb59945661bbc7061a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pearson-correlation-computation

## Summary

Computes Pearson correlation co-expression edges across normal samples, and recomputes them with each tumor sample appended, to obtain sample-specific edge differences (Δr_ij) that feed WMDS.net's personalized differential co-expression (SSN) analysis.

## When to use

Use when you have two condition-specific expression matrices (rows = gene symbols, columns = sample IDs) and need per-patient differential co-expression edges for downstream WMDS.net driver-node ranking.

## When NOT to use

- Input is a single expression matrix with no normal/tumor contrast — the Δr_ij computation requires a pooled normal reference set against which each tumor sample is compared.
- The goal is group-level differential co-expression rather than sample-specific (personalized) networks — appending one tumor sample at a time is specific to the SSN mode.
- The gene space has no overlap with the Gene_Net hub network (DawnRank interactions ∩ TCGA genes), since candidate pairs are restricted to that reference.

## Inputs

- Normal-sample expression matrix (rows = gene symbols, columns = sample IDs), e.g. TCGA_BLCAnormal
- Tumor-sample expression matrix (rows = gene symbols, columns = sample IDs), e.g. TCGA_BLCAtumor
- Gene_Net hub reference network (intersection of the DawnRank gene-interaction network with TCGA genes) used to restrict candidate gene pairs

## Outputs

- Per-patient edge list personalized_ssn_edges.csv with columns patient_id, gene_i, gene_j, delta_r, p_value
- Per-patient personalized differential co-expression network (edges with p_value < 0.05)

## How to apply

Load the normal-sample and tumor-sample expression matrices (rows = gene symbols, columns = sample IDs) and restrict candidate gene pairs to the Gene_Net hub reference network (intersection of the DawnRank gene-interaction network with TCGA genes). Compute Pearson correlations r_ij^n across all n normal samples to build the co-expression reference network. For each tumor sample individually, append it to the normal reference samples, recompute the correlation r_ij^(n+1), and take Δr_ij = r_ij^(n+1) − r_ij^n for every candidate edge. Convert each Δr_ij to a Z-score, derive a P-value by Z-test, and retain edges with P-value < 0.05 as that patient's personalized differential co-expression network. The rationale for the append-and-recompute strategy is that it isolates the single tumor sample's perturbation of each co-expression edge relative to the pooled normal reference.

## Related tools

- **WMDS.net** (Framework consuming the Pearson-correlation-derived differential co-expression network to find driver nodes; implementation whose SSN mode defines the Δr_ij → Z-score → P-value workflow) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Language in which the WMDS.net algorithm is implemented; its Optimization ToolBox intlinprog function solves the downstream binary integer-programming problem)
- **DawnRank** (Source of the gene-interaction network used to build the Gene_Net hub reference that restricts candidate gene pairs for correlation computation)

## Examples

```
Rscript -e 'N <- as.matrix(read.csv("TCGA_BLCAnormal.csv", row.names=1)); t_i <- as.matrix(read.csv("TCGA_BLCAtumor.csv", row.names=1))[,i]; M <- cbind(N, t_i); delta_r <- cor(t(M)) - cor(t(N))'
```

## Evaluation signals

- personalized_ssn_edges.csv exists and contains the columns patient_id, gene_i, gene_j, delta_r, p_value
- Every retained edge in each patient's output has p_value < 0.05 (Z-test on the Δr_ij Z-score)
- Edge sets differ across patients, as expected for sample-specific networks built by appending one tumor sample at a time
- For each edge, delta_r equals r_ij^(n+1) − r_ij^n, where r_ij^(n+1) is the correlation recomputed on the normal samples plus the appended tumor sample
- All candidate gene pairs fall within the Gene_Net hub network (intersection of the DawnRank interaction network with TCGA genes)

## Limitations

- Candidate gene pairs are restricted to the Gene_Net hub network (DawnRank gene-interaction network ∩ TCGA genes), so correlations are not computed over the full gene space
- Only edges with P-value < 0.05 are retained; no additional multiple-testing correction is described in the workflow
- The reference network depends on the specific normal-sample cohort (e.g. TCGA_BLCAnormal); changing the normal reference changes every patient's Δr_ij values

## Evidence

- [readme] README matrix format: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
- [readme] Gene_Net restriction: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [abstract] WMDS.net purpose: "we developed a weighted MDS network model (WMDS.net) to find the driver nodes of differential gene co-expression networks"
- [intro] Implementation environment: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
