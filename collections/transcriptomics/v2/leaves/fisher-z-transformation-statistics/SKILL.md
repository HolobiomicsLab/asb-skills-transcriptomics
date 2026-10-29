---
name: fisher-z-transformation-statistics
description: Use when you have two condition-specific FPKM expression matrices and
  a fixed reference network, and need to test whether each edge's Pearson correlation
  differs significantly between conditions.
when_to_use_negative:
- The cancer type (or condition comparison) has fewer than 15 normal samples — the
  ΔZ distribution's standard-normal approximation was validated only for sample size
  >15
- There is no predefined reference network constraining which gene pairs to test —
  the test is applied only to gene pairs connected in Gene_Net, not genome-wide
- The expression data are raw counts rather than FPKM-normalized, since Pearson correlations
  are computed on FPKM-normalized values
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: WMDS.net
  role: Framework that consumes the differential co-expression edges produced by the
    Fisher Z / ΔZ test to find driver nodes
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation language for the pipeline (Optimization ToolBox intlinprog
    used for the downstream MDS problem)
- name: DawnRank
  role: Source of the Gene_Net reference network whose edges define the gene pairs
    tested for correlation difference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:5b4fc7fcdb94bafba52b80493dce0be2fd990858e4c109d794fe34da8c626631
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fisher-z-transformation-statistics

## Summary

Quantifies differential co-expression edges between two physiological conditions by Fisher Z-transforming Pearson correlation coefficients and applying a ΔZ Z-test per gene pair, as used in WMDS.net on TCGA RNA-seq data.

## When to use

Use when you have two condition-specific FPKM expression matrices and a fixed reference network, and need to test whether each edge's Pearson correlation differs significantly between conditions.

## When NOT to use

- The cancer type (or condition comparison) has fewer than 15 normal samples — the ΔZ distribution's standard-normal approximation was validated only for sample size >15
- There is no predefined reference network constraining which gene pairs to test — the test is applied only to gene pairs connected in Gene_Net, not genome-wide
- The expression data are raw counts rather than FPKM-normalized, since Pearson correlations are computed on FPKM-normalized values

## Inputs

- FPKM-normalized mRNA expression matrix for condition 1 (e.g. TCGA_BLCAtumor; rows = gene symbols, columns = sample ids)
- FPKM-normalized mRNA expression matrix for condition 2 (e.g. TCGA_BLCAnormal; rows = gene symbols, columns = sample ids)
- Dawnrank-derived Gene_Net reference network (11,648 genes and 211,794 edges)

## Outputs

- differential_coexpression_edges.csv with (gene_i, gene_j, ΔZ, P-value) rows for edges significant at P < 0.05

## How to apply

Load the Dawnrank-derived Gene_Net reference network (11,648 genes, 211,794 edges) and retain only its largest connected subgraph, discarding nodes with no edge connection to the central part. Compute Pearson correlations on FPKM-normalized mRNA expression between every gene pair connected in the reference network, separately for the tumor and normal matrices (rows = gene symbols, columns = sample ids). Fisher Z-transform each correlation, z = 0.5*ln((1+r)/(1-r)), where z_ij is asymptotically normal with standard deviation 1/sqrt(n-3). For each edge compute ΔZ = (z_ij1 - z_ij2)/sqrt(1/(n1-3) + 1/(n2-3)) under the null ρ1 = ρ2, yielding a standard normal statistic, and derive the two-sided P-value; retain edges with P < 0.05 as significant differential edges.

## Related tools

- **WMDS.net** (Framework that consumes the differential co-expression edges produced by the Fisher Z / ΔZ test to find driver nodes) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation language for the pipeline (Optimization ToolBox intlinprog used for the downstream MDS problem))
- **DawnRank** (Source of the Gene_Net reference network whose edges define the gene pairs tested for correlation difference)

## Examples

```
z1 = 0.5*log((1+r1)./(1-r1)); z2 = 0.5*log((1+r2)./(1-r2)); dz = (z1-z2)./sqrt(1/(n1-3)+1/(n2-3)); p = 2*(1-normcdf(abs(dz)));
```

## Evaluation signals

- Each edge's ΔZ = (z_ij1 - z_ij2)/sqrt(1/(n1-3) + 1/(n2-3)) should follow a standard normal distribution under H0: ρ1 = ρ2; Kolmogorov-Smirnov testing showed good approximation when sample size >15
- Output file contains rows of (gene_i, gene_j, ΔZ, P-value) and only edges with P < 0.05 are retained as significant differential edges
- Fisher-transformed z values must satisfy z = 0.5*ln((1+r)/(1-r)) with standard deviation 1/sqrt(n-3)
- Input matrices must have gene symbols as rows and sample ids as columns; correlations are computed only for pairs connected in the retained largest connected subgraph of Gene_Net
- Resulting differential network should satisfy approximate scale-free topology with linear-regression R² > 0.80

## Limitations

- The ΔZ statistic is only a good approximation of the standard normal when sample size >15 (per Kolmogorov-Smirnov testing); apply only to cancer types with at least 15 normal samples
- Nodes in Gene_Net with no edge connection to the central part are discarded, so isolated genes cannot yield differential edges
- The correlation-difference test is restricted to gene pairs already connected in the Dawnrank-derived reference network

## Evidence

- [other] fisher_z_delta_test: [evidence span withheld — anchor status: not_found]
