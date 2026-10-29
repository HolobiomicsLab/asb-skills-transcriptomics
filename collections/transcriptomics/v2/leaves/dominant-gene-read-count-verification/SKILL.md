---
name: dominant-gene-read-count-verification
description: Use when a pathway node contains multiple functionally homologous genes
  and you must determine which enzyme dominates the reaction and whether the aggregated
  node-level regulation is trustworthy, e.g., after collapsing RNA-seq read counts
  in FunHoP networks.
when_to_use_negative:
- The pathway is from KEGG category 1.7 'glycan biosynthesis and metabolism' where
  links rather than nodes are used and nodes cannot be expanded or colored in Cytoscape
- Only single-gene nodes are present, so there is no homolog-collapse or dominant-gene
  question to verify
- KEGG-supplied enzyme kinetics or isoform-level activity data is required, since
  FunHoP limits analysis to the gene level
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0121
tools:
- name: FunHoP
  role: Pipeline (funhop.py) that expands multi-gene KEGG nodes, aggregates per-gene
    read counts into node-level totals, and produces the values to verify
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization of expanded and aggregated networks; read counts styled on a
    0 to >=50,000 white-to-dark-blue scale to inspect dominant genes
- name: KEGGScape
  role: KGML-reader app used to load the FunHoP network files into Cytoscape via KEGGScape
- name: Voom
  role: Differential expression at gene and node level (P < 0.05) used to confirm
    node regulation after aggregation
- name: ElementTree XML API
  role: Expands multi-gene KGML nodes (e.g., the 21-gene PLA2G4B node) into per-gene
    child nodes before aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:0b8733793620e6888b71f33230700121b0dba7d621a2b5164ba8b8938d041025
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - ElementTree XML API
  - FunHoP
  - KEGGScape
  - Voom
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dominant-gene-read-count-verification

## Summary

Verifies that FunHoP correctly identifies the dominant (highest read-count) gene within a multi-gene KEGG node (e.g., the 21-gene PLA2G4B node in hsa00564) and that the aggregated node-level read counts and regulation match the published case-study values.

## When to use

Use when a pathway node contains multiple functionally homologous genes and you must determine which enzyme dominates the reaction and whether the aggregated node-level regulation is trustworthy, e.g., after collapsing RNA-seq read counts in FunHoP networks.

## When NOT to use

- The pathway is from KEGG category 1.7 'glycan biosynthesis and metabolism' where links rather than nodes are used and nodes cannot be expanded or colored in Cytoscape
- Only single-gene nodes are present, so there is no homolog-collapse or dominant-gene question to verify
- KEGG-supplied enzyme kinetics or isoform-level activity data is required, since FunHoP limits analysis to the gene level

## Inputs

- TCGA prostate cancer RNA-seq raw read count matrix (tab-separated)
- Gene expression table with gene names and IDs (tab-separated)
- Sample metadata with cancer/control tissue status
- KEGG hsa-ID to gene-name conversion list (http://rest.kegg.jp/list/hsa)
- Updated gene symbols list
- KEGG KGML file for glycerophospholipid metabolism (hsa00564)
- Gene length table (sum of exons per gene)

## Outputs

- Expanded KEGG network XML with one child node per gene and white-bordered functional-homolog (OR) nodes
- Aggregated network XML with node-level total read counts and regulation values
- Summary of expression counts for defining Cytoscape color schemes
- Verified dominant-enzyme calls per node (e.g., PLA2G2A 71,482 reads) and node-level voom differential expression results

## How to apply

After running funhop.py on the TCGA PCa expression table, metadata, raw read counts, hsa-ID/gene-name list, and updated gene symbols, inspect the aggregated network output for multi-gene nodes expanded with ElementTree. Adjust read counts for gene length (sum of exons divided by the average gene length over all genes), sum per-gene counts within each node to obtain node-level totals, and rank genes by read count to identify the dominant enzyme. Cross-check the resulting values against the reported Figure 4 numbers: PLA2G2A dominant with 71,482 reads, PLA2G12A 4,502 reads, and PLA2G16 3,246 reads in the PLA2G4B node of glycerophospholipid metabolism (hsa00564). Confirm the node-level differential expression (voom, DEGs at P < 0.05, Value = log2(P) x (-10) x regulation) shows the PLA2G4B node non-significant in the original network but up-regulated in the aggregated network, supporting net flux from lecithin toward GPC.

## Related tools

- **FunHoP** (Pipeline (funhop.py) that expands multi-gene KEGG nodes, aggregates per-gene read counts into node-level totals, and produces the values to verify) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization of expanded and aggregated networks; read counts styled on a 0 to >=50,000 white-to-dark-blue scale to inspect dominant genes)
- **KEGGScape** (KGML-reader app used to load the FunHoP network files into Cytoscape via KEGGScape)
- **Voom** (Differential expression at gene and node level (P < 0.05) used to confirm node regulation after aggregation)
- **ElementTree XML API** (Expands multi-gene KGML nodes (e.g., the 21-gene PLA2G4B node) into per-gene child nodes before aggregation)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Dominant-gene read counts reproduce Figure 4: PLA2G2A 71,482 reads, PLA2G12A 4,502 reads, PLA2G16 3,246 reads in the PLA2G4B node
- The PLA2G4B node is non-significant in the original KEGGScape network but up-regulated in the aggregated FunHoP network
- Node-level total read counts equal the sum of length-adjusted per-gene counts (sum of exons divided by average gene length over all genes)
- All DEGs meet P < 0.05 and are encoded as Value = log2(P) x (-10) x regulation, with differential expression gradient in the -1200 to 600 range
- funhop.py completes and prints a summary of expression counts suitable for defining Cytoscape count color schemes

## Limitations

- Assumes enzymatic reaction activity depends mainly on the gene with the highest number of RNA-seq reads; enzyme kinetics/efficiency data are usually not available so equal enzyme rate is a simplification
- KEGG does not currently provide curated information on enzyme activity of isoforms, so analysis is limited to the gene level
- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation
- funhop.py contains hard-coded assumptions: gene names in the first row of the expression table, tissue status in column 7 of metadata, and a 'correct' flag in load_countdata for gene-length correction

## Evidence

- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons)"
- [readme] If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function: "If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function"
- [discussion] However, KEGG does not currently provide curated information on enzyme activity of isoforms. We have thus limited analysis to the gene level.: "KEGG does not currently provide curated information on enzyme activity of isoforms. We have thus limited analysis to the gene level."
