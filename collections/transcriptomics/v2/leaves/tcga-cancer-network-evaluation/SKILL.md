---
name: tcga-cancer-network-evaluation
description: Use when you have constructed gene co-expression or differential co-expression
  networks (e.g. from TCGA tumor/normal RNA-seq) and need to verify they approximately
  satisfy a scale-free topology before downstream driver-node analysis.
when_to_use_negative:
- The network was not built from expression data and a scale-free degree distribution
  is not an expected property (e.g. a fixed hub/gene-interaction network like the
  Gene_Net hub network).
- The analysis goal is driver-node identification rather than network topology validation
  — use WMDS.net controllability ranking instead once the scale-free check has passed.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: Matlab R2020b
  role: Implementation language for the degree-distribution computation and log2(log2)
    linear regression fit; intlinprog from the Optimization ToolBox is used in the
    surrounding WMDS.net pipeline.
- name: WMDS.net
  role: Upstream framework whose differential gene co-expression networks (built from
    TCGA tumor/normal RNA-seq) are the objects evaluated by this scale-free topology
    check.
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:ec4075eb2e20cecc1c9d056f4ede8a0b1bb5d3e9d4064decae7a4ffdf1559dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# tcga-cancer-network-evaluation

## Summary

Assessment of differential co-expression networks built from TCGA cancer RNA-seq (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) by fitting log2(degree frequency) vs log2(degree) with linear regression and checking the R-square against the scale-free topology criterion R2 > 0.80.

## When to use

Use when you have constructed gene co-expression or differential co-expression networks (e.g. from TCGA tumor/normal RNA-seq) and need to verify they approximately satisfy a scale-free topology before downstream driver-node analysis.

## When NOT to use

- The network was not built from expression data and a scale-free degree distribution is not an expected property (e.g. a fixed hub/gene-interaction network like the Gene_Net hub network).
- The analysis goal is driver-node identification rather than network topology validation — use WMDS.net controllability ranking instead once the scale-free check has passed.

## Inputs

- 14 TCGA cancer-type differential co-expression networks (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC)
- TCGA RNA-seq expression matrices with rows as gene symbols and columns as sample ids (e.g. TCGA_BLCAtumor and TCGA_BLCAnormal test data)

## Outputs

- scale_free_r2.csv with per-cancer-type R-square values (columns: cancer type, R2)
- log-log degree distribution plots (log2 degree frequency vs log2 degree) with linear regression fits

## How to apply

Load each of the 14 TCGA differential co-expression networks produced by the upstream network-construction step. For each network, compute the node degree distribution (frequency of each degree value), then plot and fit log2(numbers of degree) versus log2(degree) with a linear regression model, following the paper's scale-free topology check as in Figure 2C. Record the R-square of each fit; apply the Zhang and Horvath acceptance threshold R2 > 0.80, with the paper reporting R2 > 0.9 on most of the 14 TCGA datasets. Implement in Matlab R2020b and write per-cancer-type R2 values to scale_free_r2.csv with columns for cancer type and R2.

## Related tools

- **Matlab R2020b** (Implementation language for the degree-distribution computation and log2(log2) linear regression fit; intlinprog from the Optimization ToolBox is used in the surrounding WMDS.net pipeline.)
- **WMDS.net** (Upstream framework whose differential gene co-expression networks (built from TCGA tumor/normal RNA-seq) are the objects evaluated by this scale-free topology check.) — https://github.com/chaofen123/WMDS.net

## Examples

```
p = fitlm(log2(degrees), log2(degFreqs)); R2 = p.Rsquared.Ordinary; assert(R2 > 0.80, 'fails scale-free topology check')
```

## Evaluation signals

- Every reported R-square exceeds the paper's acceptance threshold R2 > 0.80 (Zhang and Horvath criterion); most should exceed 0.9 as in Figure 2C.
- scale_free_r2.csv contains exactly 14 rows, one per TCGA cancer type, with columns for cancer type and R2.
- The fitted regression is on log2-transformed axes: log2(numbers of degree) vs log2(degree), matching the paper's scale-free check.
- Degree distributions are recomputed from the network adjacency/graph structure, not copied from raw expression matrices.

## Limitations

- The check is an approximate criterion (R2 > 0.80), not a strict test of scale-freeness; passing it does not guarantee an ideal power-law degree distribution.
- Reported performance (R2 > 0.9 on most datasets) is specific to the paper's 14 TCGA differential co-expression networks and may not transfer to other network constructions.

## Evidence

- [abstract] we developed a weighted MDS network model (WMDS.net) to find the driver nodes of differential gene co-expression networks: "we developed a weighted MDS network model (WMDS.net) to find the driver nodes of differential gene co-expression networks"
- [intro] solve the binary integer-programming problem using function "intlinprog" which is available in the Optimization ToolBox of MatLab version R2020b: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas."
- [readme] The rows of the expression matrices represent gene expression, and the columns represent samples.: "The rows of the expression matrices represent gene expression, and the columns represent samples."
