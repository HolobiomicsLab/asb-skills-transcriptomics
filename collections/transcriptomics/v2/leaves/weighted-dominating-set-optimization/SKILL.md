---
name: weighted-dominating-set-optimization
description: Use when you have paired expression matrices (e.g., TCGA tumor vs normal)
  as a differential co-expression network and need to identify key driver nodes of
  transcriptome programs.
when_to_use_negative:
- Input is a single condition without a paired reference state — the method requires
  co-expression differences between two physiological states
- Data are not gene expression matrices with gene symbols as row names and sample
  IDs as column names in the format the Matlab main program expects
- The goal is global (non-personalized) driver detection across a cohort; WMDS.netP's
  per-patient sample-specific network construction is only needed for personalized
  driver discovery
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3524
tools:
- name: WMDS.net
  role: Weighted MDS network model that finds driver nodes of differential gene co-expression
    networks by integrating node degree and co-expression difference significance
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation environment; solves the binary integer-programming problem
    via the intlinprog function in the Optimization ToolBox
- name: DawnRank
  role: Source of the gene interaction network intersected with TCGA genes to construct
    the Gene_Net hub network
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:5e05ce674674e70b6683ad80a1e66f014154f766f636622e5faeb04049565693
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# weighted-dominating-set-optimization

## Summary

Solve the weighted minimum dominating set (WMDS) on a gene co-expression network to identify driver nodes, integrating node degree and the significance of co-expression difference between two physiological states into node controllability via a binary integer program solved with Matlab's intlinprog.

## When to use

Use when you have paired expression matrices (e.g., TCGA tumor vs normal) as a differential co-expression network and need to identify key driver nodes of transcriptome programs.

## When NOT to use

- Input is a single condition without a paired reference state — the method requires co-expression differences between two physiological states
- Data are not gene expression matrices with gene symbols as row names and sample IDs as column names in the format the Matlab main program expects
- The goal is global (non-personalized) driver detection across a cohort; WMDS.netP's per-patient sample-specific network construction is only needed for personalized driver discovery

## Inputs

- Expression matrix for condition 1 (e.g., TCGA_BLCAtumor): rows = gene symbols, columns = sample IDs
- Expression matrix for condition 2 (e.g., TCGA_BLCAnormal): rows = gene symbols, columns = sample IDs
- Gene_Net: hub network from intersection of DawnRank gene interaction network and TCGA genes

## Outputs

- Gene list of key nodes (driver genes) in the final weighted co-expression network
- For WMDS.netP: per-patient personalized driver gene lists with recurrence frequencies classifiable as high (>=0.6), medium (0.3-0.6), low (<0.3)

## How to apply

Build the differential gene co-expression network (Gene_Net, the hub network from the intersection of the DawnRank gene interaction network and TCGA genes), then formulate the weighted minimum dominating set as a binary integer-programming problem solved with the 'intlinprog' function from the Optimization ToolBox of MatLab R2020b. Weight each node using its degree in the network and the significance of the gene co-expression difference between the two physiological states, so that controllability reflects both topology and differential signal. Inputs to the main program are the two expression matrices (rows = genes/symbols, columns = samples), e.g. TCGA_BLCAtumor and TCGA_BLCAnormal. For the personalized (WMDS.netP) variant, build each patient's sample-specific network by adding that patient's tumor sample to the TCGA normal-sample reference network, recalculating Pearson correlations, retaining edges with Delta-r Z-test P < 0.05, and using node weight wi = (sum over adjacent j of -log10(p_ij))^(-gamma) with gamma = 0.01. The output is the gene list of key nodes (driver genes) in the final network.

## Related tools

- **WMDS.net** (Weighted MDS network model that finds driver nodes of differential gene co-expression networks by integrating node degree and co-expression difference significance) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation environment; solves the binary integer-programming problem via the intlinprog function in the Optimization ToolBox)
- **DawnRank** (Source of the gene interaction network intersected with TCGA genes to construct the Gene_Net hub network)

## Examples

```
In MatLab R2020b: [fval,x] = intlinprog(c, [], [], [], Aeq, beq, lb, ub) % c encodes node weights wi from degree and co-expression difference p-values, x = binary driver-node indicators
```

## Evaluation signals

- The intlinprog binary integer program converges to a dominating set: every network node is either selected or adjacent to a selected node
- Selected driver lists show better balance between precision and recall than top-tier comparator tools across TCGA datasets
- Sanity check: CDKN2A appears in the LUAD WMDS.netP personalized driver list while absent from the global WMDS.net driver list
- For personalized runs, the low-frequency class (<0.3) accounts for the largest proportion of driver genes in every one of the 14 cancer types (Figure 5 expectation)
- Output is a gene list of key nodes with gene symbols consistent with the input matrix row names

## Limitations

- Requires Matlab R2020b with the Optimization ToolBox (intlinprog) — not portable to other solvers without reimplementation
- Network is restricted to Gene_Net, the intersection of the DawnRank interaction network and TCGA genes; genes outside this hub cannot be selected
- The paper's discussion section content is absent from the extracted document (only a changelog placeholder), so caveats from discussion are unavailable

## Evidence

- [intro] Weighted MDS formulation: "We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks."
- [readme] Solver and parameters: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] Input format: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
- [intro] Gene_Net definition: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [abstract] Validation via TCGA: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall."
