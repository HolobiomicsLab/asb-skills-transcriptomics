---
name: spatial-apa-pattern-clustering
description: Use when you have an stAPAminer object with DEAPA, LSAPA, and SVAPA gene
  calls and need to classify genes into layer-specific spatial APA usage patterns
  and enrich them for GO terms.
when_to_use_negative:
- Input is not spatially barcoded ST data — APA sites can only be identified from
  spatially barcoded ST data with tools like scAPAtrap or Sierra, not from image-based
  MERFISH/seqFISH data
- You only need genes with spatial gene expression patterns rather than spatial APA
  usage patterns — some genes (e.g., St8sia5, Myef2, Per1) show spatial APA patterns
  without spatial gene expression patterns and require this APA-specific workflow
- You are analyzing intronic APA genes — the 601 intronic APA genes with spatial patterns
  are distinct from the 654 genes with spatial 3' UTR APA patterns and the paper treats
  them separately
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: Provides findContrastAPA, findAllMarkers, findSVAPA, findstAPASet, and findSpatialPattern
    to merge DEAPA/LSAPA/SVAPA genes and group them into five spatial APA usage patterns
  repo: https://github.com/BMILAB/stAPAminer
- name: org.Mm.eg.db
  role: Mouse gene annotation used for Gene Ontology functional enrichment of the
    spatial pattern gene sets
- name: SPARK
  role: Detects spatially variable APA usage to define SVAPA genes (adjusted P < 0.05)
    feeding the combined gene set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:991d59731afb43015416117280db0a09ee3f77ddeeaf446727d2b37334a54fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - org.Mm.eg.db
  - stAPAminer
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spatial-apa-pattern-clustering

## Summary

Groups the combined DEAPA, LSAPA, and SVAPA genes from stAPAminer into five major spatial APA usage patterns corresponding to mouse olfactory bulb (MOB) morphological layers, then characterizes each pattern set by Gene Ontology (GO) enrichment. This reproduces the paper's synthesis of 654 non-redundant 3' UTR APA genes into spatial pattern clusters.

## When to use

Use when you have an stAPAminer object with DEAPA, LSAPA, and SVAPA gene calls and need to classify genes into layer-specific spatial APA usage patterns and enrich them for GO terms.

## When NOT to use

- Input is not spatially barcoded ST data — APA sites can only be identified from spatially barcoded ST data with tools like scAPAtrap or Sierra, not from image-based MERFISH/seqFISH data
- You only need genes with spatial gene expression patterns rather than spatial APA usage patterns — some genes (e.g., St8sia5, Myef2, Per1) show spatial APA patterns without spatial gene expression patterns and require this APA-specific workflow
- You are analyzing intronic APA genes — the 601 intronic APA genes with spatial patterns are distinct from the 654 genes with spatial 3' UTR APA patterns and the paper treats them separately

## Inputs

- stAPAminer object (stObj) built from imputed APA usage ratios (RUD) and spot coordinates of the MOB ST data
- DEAPA, LSAPA, and SVAPA gene lists from findContrastAPA, findAllMarkers, and findSVAPA
- Mouse gene annotation via org.Mm.eg.db for GO enrichment

## Outputs

- Gene-to-spatial-pattern assignments (five clusters: ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL)
- Per-cluster gene counts (table(patterns$cluster))
- GO enrichment terms per pattern set at FDR 5%
- spatial_apa_patterns.csv with gene-to-pattern assignments and enrichment results

## How to apply

First merge the DEAPA genes (adjusted P < 0.05 and log2 fold change > 0.5 from findContrastAPA), LSAPA genes (findAllMarkers with logFC = 0.5), and SVAPA genes (findSVAPA with SPARK adjusted P < 0.05) into a non-redundant set using findstAPASet(stObj). Then run findSpatialPattern(stObj, gene) on the combined set, selecting representative genes by Pearson's correlation > 0.5 and P < 0.05 against the cluster mean APA usage profile. Group the selected genes with k-means into five patterns matching the MOB layers (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL) and summarize per-cluster counts with table(patterns$cluster). Finally, perform GO enrichment with mouse annotation via org.Mm.eg.db and retain terms at a false discovery rate of 5%, yielding terms such as synapse organization, neuron differentiation, and nervous system development.

## Related tools

- **stAPAminer** (Provides findContrastAPA, findAllMarkers, findSVAPA, findstAPASet, and findSpatialPattern to merge DEAPA/LSAPA/SVAPA genes and group them into five spatial APA usage patterns) — https://github.com/BMILAB/stAPAminer
- **org.Mm.eg.db** (Mouse gene annotation used for Gene Ontology functional enrichment of the spatial pattern gene sets)
- **SPARK** (Detects spatially variable APA usage to define SVAPA genes (adjusted P < 0.05) feeding the combined gene set)

## Examples

```
geneSet <- findstAPASet(stObj); patterns <- findSpatialPattern(stObj, geneSet); table(patterns$cluster)
```

## Evaluation signals

- The merged DEAPA + LSAPA + SVAPA set from ST-MOB data should contain 654 non-redundant 3' UTR APA genes
- Representative genes must satisfy Pearson's correlation > 0.5 and P < 0.05 against the cluster mean APA usage profile
- k-means grouping should recover five clusters corresponding to the MOB morphological layers (ONL; GL+MCL; GL+ONL+OPL; GCL; non-ONL)
- GO enrichment should yield 66 terms at FDR 5%, including olfactory bulb development terms such as synapse organization, neuron differentiation, and nervous system development
- Output file spatial_apa_patterns.csv should contain both gene-to-pattern assignments and enrichment results

## Limitations

- The APA ratio matrix is transformed into a count matrix to meet the input requirement of SPARK; such a transformation may not be theoretically reasonable or fully meet the model's assumptions
- Missing APA usage entries remaining after imputation iterations are arbitrarily set to 0, which can affect downstream pattern detection
- No method is proposed to mitigate discrepancies among replicates when identifying reproducible spatial patterns, although spatial APA patterns of several genes were reproducible across two additional MOB replicates
- K-means cluster assignment of patterns depends on the mean APA usage profiles; genes not meeting the correlation > 0.5 and P < 0.05 thresholds are excluded from pattern grouping

## Evidence

- [discussion] selected genes with Pearson's correlation > 0.5 and P < 0.05, as representative genes with spatial APA patterns: [evidence span withheld — anchor status: not_found]
- [discussion] 654 non-redundant genes from combined DEAPA, LSAPA, SVAPA representing five patterns: "A total of 654 non-redundant 3′ UTR APA genes from the combined list of DEAPA, LSAPA, and SVAPA genes were obtained from the ST-MOB data (Figure 5A)"
- [abstract] Compiled comprehensive list of genes with spatial APA dynamics and obtained major spatial expression patterns: "We compiled a comprehensive list of genes with spatial APA dynamics and obtained several major spatial expression patterns"
- [readme] README geneSet construction via findstAPASet: "stObj <- findContrastAPA(stObj)
stObj <- findAllMarkers(stObj,logFC = 0.5)
stObj <- findSVAPA(stObj)
geneSet <- findstAPASet(stObj)"
