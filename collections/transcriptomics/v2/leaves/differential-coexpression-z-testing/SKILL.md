---
name: differential-coexpression-z-testing
description: Use when you have FPKM-normalized expression matrices for two physiological
  conditions (e.g. TCGA_BLCAtumor vs TCGA_BLCAnormal, rows=gene symbols, columns=sample
  ids) and need to call significant differential co-expression edges on a fixed reference
  network.
when_to_use_negative:
- The cancer type has fewer than 15 normal samples — the ΔZ standard-normal approximation
  was shown (by Kolmogorov-Smirnov testing) to be reliable only for sample size >
  15.
- You need de novo co-expression edges rather than differential edge calls on a fixed
  reference network — this skill restricts testing to gene pairs already connected
  in the Gene_Net reference network.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: WMDS.net
  role: Framework whose differential co-expression network construction (bulk ΔZ test
    and SSN per-patient Δr Z-test) this skill implements
  repo: https://github.com/chaofen123/WMDS.net
- name: DawnRank
  role: Source of the Gene_Net gene-interaction network that restricts which gene
    pairs are tested
- name: Matlab R2020b
  role: Implementation language; intlinprog from the Optimization ToolBox is used
    downstream for the WMDS binary integer-programming step
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:7b172b2074fe10b2222e74fd916a7fa45b75dc8a9eee814e0b48621b8e36b91d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# differential-coexpression-z-testing

## Summary

Statistical testing of differential gene co-expression edges as implemented in WMDS.net: Fisher Z-transform Pearson correlations computed per condition and apply a Z-test on the correlation difference (ΔZ) per reference-network edge, retaining edges with P < 0.05. Also covers the SSN variant that Z-tests the per-patient edge difference Δr_ij between a normal reference network and a tumor-perturbed network.

## When to use

Use when you have FPKM-normalized expression matrices for two physiological conditions (e.g. TCGA_BLCAtumor vs TCGA_BLCAnormal, rows=gene symbols, columns=sample ids) and need to call significant differential co-expression edges on a fixed reference network.

## When NOT to use

- The cancer type has fewer than 15 normal samples — the ΔZ standard-normal approximation was shown (by Kolmogorov-Smirnov testing) to be reliable only for sample size > 15.
- You need de novo co-expression edges rather than differential edge calls on a fixed reference network — this skill restricts testing to gene pairs already connected in the Gene_Net reference network.

## Inputs

- FPKM-normalized mRNA expression matrix for condition 1 (rows = gene symbols, columns = sample ids), e.g. TCGA_BLCAtumor
- FPKM-normalized mRNA expression matrix for condition 2 (rows = gene symbols, columns = sample ids), e.g. TCGA_BLCAnormal
- Gene_Net hub reference network (intersection of the DawnRank gene-interaction network with TCGA genes; 11,648 genes, 211,794 edges)

## Outputs

- differential_coexpression_edges.csv with columns (gene_i, gene_j, ΔZ, P-value), restricted to P < 0.05 edges
- personalized_ssn_edges.csv (one per patient) with columns (patient_id, gene_i, gene_j, delta_r, p_value), restricted to P < 0.05 edges

## How to apply

Load the Dawnrank-derived Gene_Net reference network (11,648 genes, 211,794 edges) and keep only its largest connected subgraph. Compute Pearson correlation coefficients on FPKM-normalized mRNA expression for every gene pair connected in the reference network, separately for the tumor and normal matrices. Fisher Z-transform each correlation, z = 0.5*ln((1+r)/(1-r)), which is asymptotically normal with standard deviation 1/sqrt(n-3). For each edge compute ΔZ = (z_ij1 - z_ij2)/sqrt(1/(n1-3) + 1/(n2-3)) under H0: ρ1 = ρ2 (standard normal statistic), derive the two-sided P-value, and retain edges with P < 0.05. For the personalized SSN mode, instead recompute r_ij^(n+1) after appending each tumor sample to the normal reference samples, take Δr_ij = r_ij^(n+1) − r_ij^n, convert to a Z-score with a Z-test, and retain per-patient edges with P < 0.05. Rationale: the KS-validated Z approximation and the fixed reference network restrict testing to pairs plausibly co-regulated, feeding WMDS.net's node controllability scoring.

## Related tools

- **WMDS.net** (Framework whose differential co-expression network construction (bulk ΔZ test and SSN per-patient Δr Z-test) this skill implements) — https://github.com/chaofen123/WMDS.net
- **DawnRank** (Source of the Gene_Net gene-interaction network that restricts which gene pairs are tested)
- **Matlab R2020b** (Implementation language; intlinprog from the Optimization ToolBox is used downstream for the WMDS binary integer-programming step)

## Examples

```
z = 0.5*np.log((1+r)/(1-r)); dz = (z1-z2)/np.sqrt(1/(n1-3)+1/(n2-3)); p = 2*(1-stat.norm.cdf(abs(dz)))
```

## Evaluation signals

- Every row written to differential_coexpression_edges.csv / personalized_ssn_edges.csv has p_value < 0.05.
- The testing cancer type has at least 15 normal samples, so the ΔZ statistic is a valid standard-normal approximation.
- The resulting differential network satisfies approximate scale-free topology with linear-regression R² > 0.80.
- Personalized SSN edge sets differ across patients, as expected for sample-specific networks.
- Expression matrix orientation is validated: rows = gene symbols, columns = sample ids.

## Limitations

- The ΔZ standard-normal approximation holds only when sample size is > 15 (KS-test validated); smaller normal cohorts invalidate the P-values.
- Only gene pairs connected in the Gene_Net reference network are tested; nodes with no edge connection to the central part of Gene_Net are discarded, so unmodeled edges are never assessed.
- Significance threshold is fixed at P < 0.05 with no stated multiple-testing correction for the 211,794 candidate edges.

## Evidence

- [other] Differential co-expression edges between two conditions are assessed by Fisher Z-transforming the Pearson correlation coefficients and applying a Z-test to the correlation difference of each gene pair.: [evidence span withheld — anchor status: not_found]
- [other] Retain edges with P < 0.05 as significant differential edges: [evidence span withheld — anchor status: not_found]
- [other] Fisher Z-transform each correlation, z = 0.5*ln((1+r)/(1-r)), where zij is asymptotically normal with standard deviation 1/sqrt(n-3): [evidence span withheld — anchor status: not_found]
- [readme] The rows of the expression matrices represent gene expression, and the columns represent samples.: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
- [readme] Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA.: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
