---
name: spatial-apa-gene-set-integration
description: Use when you have an stAPAminer object with DEAPA, LSAPA and SVAPA calls
  (e.g., ST-MOB Replicate 11) and need a combined non-redundant gene set, its k-means
  spatial pattern clusters, and GO terms.
when_to_use_negative:
- 'Input is image-based ST data (MERFISH/seqFISH): APA sites can only be identified
  from spatially barcoded ST data with tools like scAPAtrap or Sierra'
- You only need spatially variable genes from the conventional gene-spot expression
  matrix, not APA usage patterns (SVAPA and SV gene sets overlap only partially —
  16 shared genes in ST-MOB Replicate 11)
- The ratio-type APA usage matrix has not been count-transformed for SPARK; the article
  notes this transformation may not fully meet SPARK's model assumptions and should
  be treated as a workaround
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: Provides findContrastAPA, findAllMarkers, findSVAPA, findstAPASet, and findSpatialPattern
    for identifying and integrating DEAPA/LSAPA/SVAPA genes and their spatial patterns
  repo: https://github.com/BMILAB/stAPAminer
- name: SPARK
  role: Applied by findSVAPA to the count-transformed imputed RUD matrix to detect
    SVAPA genes at adjusted P < 0.05
- name: org.Mm.eg.db
  role: Mouse gene annotation used for GO functional enrichment of the spatial pattern
    gene sets at 5% FDR
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:b890e7a13362356e06a4f4bd3c82fee808cd5feb077e9ce283cc8b619ca6dd11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - org.Mm.eg.db
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spatial-apa-gene-set-integration

## Summary

Integrates the three stAPAminer gene classes (DEAPA, LSAPA, SVAPA) detected from spatially barcoded ST data into a single non-redundant spatial APA gene set, resolves its major spatial APA usage patterns, and annotates them with GO enrichment.

## When to use

Use when you have an stAPAminer object with DEAPA, LSAPA and SVAPA calls (e.g., ST-MOB Replicate 11) and need a combined non-redundant gene set, its k-means spatial pattern clusters, and GO terms.

## When NOT to use

- Input is image-based ST data (MERFISH/seqFISH): APA sites can only be identified from spatially barcoded ST data with tools like scAPAtrap or Sierra
- You only need spatially variable genes from the conventional gene-spot expression matrix, not APA usage patterns (SVAPA and SV gene sets overlap only partially — 16 shared genes in ST-MOB Replicate 11)
- The ratio-type APA usage matrix has not been count-transformed for SPARK; the article notes this transformation may not fully meet SPARK's model assumptions and should be treated as a workaround

## Inputs

- stAPAminer object (stObj) with layer annotations and imputed RUD (APA usage) matrix
- Gene-spot count matrix (count.csv) and spot coordinates (position.txt) used for KNN imputation (k=10) and SPARK
- Mouse GO annotation package org.Mm.eg.db

## Outputs

- Non-redundant combined spatial APA gene set (654 genes expected from DEAPA + LSAPA + SVAPA)
- Gene-to-spatial-pattern assignments across five k-means clusters (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL)
- GO enrichment results for the spatial pattern gene sets at 5% FDR
- Tabular output such as spatial_apa_patterns.csv and spatial_apa_gene_sets.csv

## How to apply

Run findContrastAPA on the layer-annotated stAPAminer object to get DEAPA genes (adjusted P < 0.05, log2 fold change > 0.5; 905 total, 403 non-redundant in ST-MOB), findAllMarkers with logFC = 0.5 to get LSAPA genes per MOB layer (784 genes), and findSVAPA — which applies SPARK to the count-transformed imputed RUD matrix — to get SVAPA genes (adjusted P < 0.05; 133 genes). Merge the three lists into a non-redundant set with findstAPASet(stObj), expecting 654 combined 3' UTR APA genes. Select representative genes with findSpatialPattern using Pearson's correlation > 0.5 and P < 0.05 against the cluster mean APA usage profile, then group them with k-means into five spatial patterns corresponding to the MOB layers (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL). Perform GO enrichment on the pattern gene sets using mouse annotation (org.Mm.eg.db) and retain terms at 5% FDR (66 GO terms expected for the 654-gene set, including synapse organization, neuron differentiation, and nervous system development).

## Related tools

- **stAPAminer** (Provides findContrastAPA, findAllMarkers, findSVAPA, findstAPASet, and findSpatialPattern for identifying and integrating DEAPA/LSAPA/SVAPA genes and their spatial patterns) — https://github.com/BMILAB/stAPAminer
- **SPARK** (Applied by findSVAPA to the count-transformed imputed RUD matrix to detect SVAPA genes at adjusted P < 0.05)
- **org.Mm.eg.db** (Mouse gene annotation used for GO functional enrichment of the spatial pattern gene sets at 5% FDR)

## Examples

```
stObj <- findContrastAPA(stObj); stObj <- findAllMarkers(stObj, logFC = 0.5); stObj <- findSVAPA(stObj); geneSet <- findstAPASet(stObj)
```

## Evaluation signals

- Combined non-redundant set from findstAPASet should contain 654 3' UTR APA genes for ST-MOB
- Component class sizes should match expectations: 905 DEAPA calls (403 non-redundant), 784 LSAPA genes, 133 SVAPA genes
- SVAPA–SV overlap from intersecting with SPARK on the gene expression matrix should be 16 genes
- findSpatialPattern representatives must pass Pearson's correlation > 0.5 and P < 0.05 against the cluster mean profile
- GO enrichment at 5% FDR should recover 66 terms including olfactory-bulb-development-related terms (synapse organization, neuron differentiation, nervous system development)

## Limitations

- The APA ratio matrix must be transformed into a count matrix to meet SPARK's input requirement; the article notes this transformation may not be theoretically reasonable or fully meet the model's assumptions
- There is no dedicated tool for identifying global spatial patterns from ratio-type data, so count-based SV tools are used as a workaround
- Discrepancies among MOB replicates exist and no method is proposed to mitigate them for more reproducible spatial patterns
- Missing APA usage entries remaining after imputation iterations are arbitrarily set to 0

## Evidence

- [discussion] A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data: "A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data"
- [discussion] Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes: "Genes with an adjusted P value < 0.05 and log2 fold change > 0.5 were considered DEAPA genes"
- [discussion] selected genes with Pearson's correlation > 0.5 and P < 0.05, as representative genes with spatial APA patterns: [evidence span withheld — anchor status: not_found]
- [readme] stObj <- findContrastAPA(stObj); stObj <- findAllMarkers(stObj,logFC = 0.5); stObj <- findSVAPA(stObj); geneSet <- findstAPASet(stObj): "stObj <- findContrastAPA(stObj)
stObj <- findAllMarkers(stObj,logFC = 0.5)
stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj)"
