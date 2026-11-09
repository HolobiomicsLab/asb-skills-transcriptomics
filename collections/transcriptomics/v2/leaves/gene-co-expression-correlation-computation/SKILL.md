---
name: gene-co-expression-correlation-computation
description: Use when two FPKM-normalized expression matrices (rows=gene symbols,
  columns=sample ids) for two physiological conditions are available and differential
  co-expression edges between gene pairs must be quantified and significance-tested.
when_to_use_negative:
- The condition of interest has fewer than 15 normal samples — the ΔZ standard-normal
  approximation breaks down (KS testing showed good approximation only for n > 15)
- Correlations are needed for all possible gene pairs rather than pairs connected
  in the Gene_Net reference network
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: WMDS.net
  role: Framework whose differential co-expression network builder executes the Pearson/Fisher-Z/ΔZ
    edge test
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation language; correlation and testing computations run in Matlab
    before the intlinprog driver-selection step
- name: DawnRank
  role: Source of the Gene_Net reference network that restricts which gene pairs get
    correlation tests
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:dba16fdf4bb64dc350f1b16a25eecd06e6c194d1114d07220e4d2569b29da6da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-co-expression-correlation-computation

## Summary

Computes pairwise Pearson correlation coefficients between genes from FPKM-normalized RNA-seq expression matrices and Fisher Z-transforms them so that correlation differences between two conditions can be tested statistically. This is the edge-quantification step underlying WMDS.net's differential co-expression network construction.

## When to use

Use when two FPKM-normalized expression matrices (rows=gene symbols, columns=sample ids) for two physiological conditions are available and differential co-expression edges between gene pairs must be quantified and significance-tested.

## When NOT to use

- The condition of interest has fewer than 15 normal samples — the ΔZ standard-normal approximation breaks down (KS testing showed good approximation only for n > 15)
- Correlations are needed for all possible gene pairs rather than pairs connected in the Gene_Net reference network

## Inputs

- FPKM-normalized mRNA expression matrix for condition 1 (rows = gene symbols, columns = sample ids, e.g. TCGA_BLCAtumor)
- FPKM-normalized mRNA expression matrix for condition 2 (rows = gene symbols, columns = sample ids, e.g. TCGA_BLCAnormal)
- Dawnrank-derived Gene_Net reference network (11,648 genes and 211,794 edges) defining which gene pairs are tested

## Outputs

- Per-edge differential co-expression statistics written as (gene_i, gene_j, ΔZ, P-value) rows to differential_coexpression_edges.csv
- Significant differential edge set restricted to P < 0.05 edges from the largest connected subgraph of Gene_Net

## How to apply

Load the Dawnrank-derived Gene_Net reference network (11,648 genes, 211,794 edges) and keep only its largest connected subgraph, discarding disconnected nodes. Compute Pearson correlations on FPKM-normalized mRNA expression for every gene pair connected in the reference network, separately for each condition matrix (e.g. TCGA_BLCAtumor and TCGA_BLCAnormal). Fisher Z-transform each correlation, z = 0.5*ln((1+r)/(1-r)); z_ij is asymptotically normal with standard deviation 1/sqrt(n-3). For each edge compute ΔZ = (z_ij1 - z_ij2)/sqrt(1/(n1-3) + 1/(n2-3)) under H0: ρ1 = ρ2, giving a standard-normal statistic and a two-sided P-value; retain edges with P < 0.05. Apply only to cancers with at least 15 normal samples, since Kolmogorov-Smirnov testing showed the ΔZ distribution approximates the standard normal only above that threshold.

## Related tools

- **WMDS.net** (Framework whose differential co-expression network builder executes the Pearson/Fisher-Z/ΔZ edge test) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation language; correlation and testing computations run in Matlab before the intlinprog driver-selection step)
- **DawnRank** (Source of the Gene_Net reference network that restricts which gene pairs get correlation tests)

## Examples

```
z = 0.5*log((1+r)/(1-r)); dz = (z_ij1 - z_ij2)./sqrt(1./(n1-3) + 1./(n2-3)); p = 2*(1-normcdf(abs(dz)));
```

## Evaluation signals

- Every output row of differential_coexpression_edges.csv has the four-field schema (gene_i, gene_j, ΔZ, P-value) and P-value is a valid two-sided standard-normal P-value
- No tested gene pair is absent from the reference network: nodes with no edge connection to the central part of Gene_Net were discarded before correlation computation
- Significant edge set contains only edges with P < 0.05
- Resulting differential network satisfies approximate scale-free topology with linear-regression R² > 0.80
- Input matrices validated as rows = gene symbols and columns = sample ids before correlation computation

## Limitations

- Requires at least 15 normal samples per condition; below that the asymptotic standard-normal assumption for ΔZ (z_ij SD = 1/sqrt(n-3)) is not supported by the article's KS testing
- Only gene pairs connected in the Gene_Net hub network (intersection of DawnRank interaction network and TCGA genes) are assessed; unconnected genes are excluded
- Matrices must be FPKM-normalized; the Pearson correlation step assumes rows are genes and columns are samples

## Evidence

- [readme] The rows of the expression matrices represent gene expression, and the columns represent samples.: "The rows of the expression matrices represent gene expression, and the columns represent samples."
- [intro] WMDS.net integrates node degree and significance of co-expression difference: [evidence span withheld — anchor status: not_found]
- [abstract] TCGA application validating the workflow: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas"
