---
name: tcga-rna-seq-matrix-processing
description: Use when you have FPKM-normalized TCGA tumor and normal expression matrices
  (rows = gene symbols, columns = sample ids) plus the DawnRank-derived Gene_Net reference
  network, and need significant condition-specific co-expression edges.
when_to_use_negative:
- The cancer type has fewer than 15 normal samples — the ΔZ statistic is no longer
  a good approximation of the standard normal, so P-values are unreliable
- Input is a single condition (only one expression matrix) — the ΔZ test fundamentally
  requires two physiological states with separate correlation estimates
- Gene pairs not connected in the Gene_Net reference network — correlations are computed
  only for pairs connected in the reference network, so testing arbitrary pairs is
  out of scope
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: WMDS.net
  role: Framework whose differential co-expression network builder this skill reconstructs;
    downstream node-controllability step consumes the significant differential edges
  repo: https://github.com/chaofen123/WMDS.net
- name: DawnRank
  role: Source of the Gene_Net reference network whose connected edges define which
    gene pairs are tested
- name: Matlab R2020b
  role: Implementation language; correlation and ΔZ computation feed the intlinprog-solved
    binary integer-programming step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:0983bad92bebfe0aa9ac319f478cc54897629e1725627de66b5440267bf69d9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# tcga-rna-seq-matrix-processing

## Summary

Builds a differential co-expression network from two TCGA RNA-seq conditions by Fisher Z-transforming Pearson correlations on the Gene_Net edge set and applying a ΔZ two-sample test. Produces significant differential edges (gene_i, gene_j, ΔZ, P-value) that feed WMDS.net's node-controllability model.

## When to use

Use when you have FPKM-normalized TCGA tumor and normal expression matrices (rows = gene symbols, columns = sample ids) plus the DawnRank-derived Gene_Net reference network, and need significant condition-specific co-expression edges.

## When NOT to use

- The cancer type has fewer than 15 normal samples — the ΔZ statistic is no longer a good approximation of the standard normal, so P-values are unreliable
- Input is a single condition (only one expression matrix) — the ΔZ test fundamentally requires two physiological states with separate correlation estimates
- Gene pairs not connected in the Gene_Net reference network — correlations are computed only for pairs connected in the reference network, so testing arbitrary pairs is out of scope

## Inputs

- FPKM-normalized TCGA tumor expression matrix (rows = gene symbols, columns = sample ids), e.g. TCGA_BLCAtumor
- FPKM-normalized TCGA normal expression matrix (rows = gene symbols, columns = sample ids), e.g. TCGA_BLCAnormal
- Gene_Net reference network: hub network of the intersection of the DawnRank gene interaction network and TCGA genes (11,648 genes, 211,794 edges)

## Outputs

- differential_coexpression_edges.csv with rows of (gene_i, gene_j, ΔZ, P-value) for edges retained at P < 0.05
- largest connected subgraph of Gene_Net used as the tested edge set

## How to apply

Load the DawnRank-derived Gene_Net reference network (11,648 genes, 211,794 edges), keep only the largest connected subgraph, and discard nodes with no edge connection to the central part. For every gene pair connected in the reference network, compute Pearson correlations on FPKM-normalized mRNA expression separately for the TCGA tumor matrix and the TCGA normal matrix, then Fisher Z-transform each correlation as z = 0.5*ln((1+r)/(1-r)), where z_ij is asymptotically normal with standard deviation 1/sqrt(n-3). For each edge compute ΔZ = (z_ij1 - z_ij2)/sqrt(1/(n1-3) + 1/(n2-3)) under the null hypothesis ρ1 = ρ2, treat it as a standard normal statistic, and derive the two-sided P-value. Retain edges with P < 0.05 as significant differential edges and write (gene_i, gene_j, ΔZ, P-value) rows to differential_coexpression_edges.csv. Restrict the method to cancer types with at least 15 normal samples, since Kolmogorov-Smirnov testing showed the ΔZ distribution approximates the standard normal only when sample size is >15.

## Related tools

- **WMDS.net** (Framework whose differential co-expression network builder this skill reconstructs; downstream node-controllability step consumes the significant differential edges) — https://github.com/chaofen123/WMDS.net
- **DawnRank** (Source of the Gene_Net reference network whose connected edges define which gene pairs are tested)
- **Matlab R2020b** (Implementation language; correlation and ΔZ computation feed the intlinprog-solved binary integer-programming step)

## Examples

```
z1 = 0.5*log((1+r1)./(1-r1)); z2 = 0.5*log((1+r2)./(1-r2)); dz = (z1-z2)./sqrt(1/(n1-3)+1/(n2-3)); p = 2*(1-normcdf(abs(dz)));
```

## Evaluation signals

- Every tested gene pair must correspond to an edge of the retained largest connected subgraph of Gene_Net (11,648 genes, 211,794 edges before pruning)
- Each z_ij should be asymptotically normal with standard deviation 1/sqrt(n-3); a Kolmogorov-Smirnov test on ΔZ should confirm approximation to the standard normal when sample size is >15
- Resulting differential network must satisfy approximate scale-free topology with linear-regression R² > 0.80
- Output CSV must contain four columns (gene_i, gene_j, ΔZ, P-value) and only edges with P < 0.05
- Rows and columns of input matrices must be gene symbols and sample ids respectively; FPKM normalization should precede correlation computation

## Limitations

- Requires at least 15 normal samples; below that the ΔZ distribution deviates from the standard normal per Kolmogorov-Smirnov testing
- Only gene pairs connected in the DawnRank-derived Gene_Net reference network are assessed; the reference network's largest connected subgraph is used and unconnected nodes are discarded
- Validated on TCGA RNA-seq data (e.g. BLCA tumor vs normal); no guarantee for other data types or normalization schemes

## Evidence

- [methods] workflow_fisher_z: [evidence span withheld — anchor status: not_found]
- [methods] workflow_retention: [evidence span withheld — anchor status: not_found]
- [readme] readme_input_format: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
- [intro] intro_gene_net: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [abstract] abstract_tcga: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas"
