---
name: kegg-pathway-node-expansion
description: Use when KEGG pathway visualization in Cytoscape/KEGGScape hides homologous
  genes (only the first listed gene per reaction node is shown) and RNA-seq read counts
  are available to prioritize the dominant enzyme per node.
when_to_use_negative:
- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded and are excluded from FunHoP
- Non-human pathways — the current FunHoP version supports only human metabolic pathways
  from KEGG
- Data lacking raw RNA-seq read counts — read-count-based prioritization of homologs
  cannot be performed
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: FunHoP
  role: Pipeline that expands KGML multi-gene nodes with ElementTree, aggregates read
    counts per node, and computes node-level differential expression
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization of expanded and aggregated pathway networks with differential-expression
    and read-count color styling
- name: KEGGScape
  role: KGML-reader app used to import KEGG pathway files into Cytoscape
- name: Voom
  role: Differential expression analysis at gene and node level (DEGs at P < 0.05)
- name: ElementTree
  role: Python XML API used to expand multi-gene KGML nodes, creating a new child
    per gene while keeping original coordinates
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:d7232cd3877393947a6db539d6a68f20aa7d4768d43dd1f6e34e0a7b1e9509b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - ElementTree
  - ElementTree XML API
  - FunHoP
  - KEGGScape
  - Voom
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# kegg-pathway-node-expansion

## Summary

FunHoP-style expansion and aggregation of KEGG KGML pathway nodes so that every functionally homologous gene in a multi-gene node is shown, and node-level expression is collapsed from RNA-seq read counts weighted on gene length and ratio.

## When to use

Use when KEGG pathway visualization in Cytoscape/KEGGScape hides homologous genes (only the first listed gene per reaction node is shown) and RNA-seq read counts are available to prioritize the dominant enzyme per node.

## When NOT to use

- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded and are excluded from FunHoP
- Non-human pathways — the current FunHoP version supports only human metabolic pathways from KEGG
- Data lacking raw RNA-seq read counts — read-count-based prioritization of homologs cannot be performed

## Inputs

- KEGG KGML pathway XML file (e.g. hsa00340, hsa00564)
- hsa-ID to gene-name conversion list (from http://rest.kegg.jp/list/hsa)
- Gene expression table with gene names, gene IDs, and probe values (tab-separated)
- Sample metadata indicating cancer vs control tissue status
- Raw RNA-seq read count matrix (e.g. from TCGA, featureCounts-assigned)
- Updated gene symbols list

## Outputs

- Expanded pathway network XML with a child node per gene in multi-gene nodes
- Aggregated node-level network with summed read counts per node (nodes named 'gene1-Bx')
- Node-level differential expression values (log2-transformed P values scaled -1200 to 600)
- Node-level total read count summary table for Cytoscape color-scheme mapping
- Verification table (e.g. histidine_pathway_funhop_verification.tsv)

## How to apply

Parse the KEGG KGML XML (e.g. hsa00340 histidine metabolism or hsa00564 glycerophospholipid metabolism) with ElementTree, creating a new child node for every gene in multi-gene nodes while keeping original coordinates and marking functional-homolog (OR) nodes with a white border; name expanded nodes 'gene1-Bx'. Run funhop.py with the hsa-ID/gene-name list, expression table, metadata, read counts, and updated gene symbols; correct counts for gene length by the factor (gene length as sum of exons) / average gene length over all genes. Compute per-gene average read counts as the mean of cancer-group and normal-group averages, then aggregate: sum read counts of all genes within each node to obtain node-level totals, assuming reaction activity depends mainly on the gene with the highest number of reads. Run Voom differential expression (PCa vs normal, DEGs at P < 0.05) at both gene and node level, log2-transform P values as Value = log2(P) x (-10) x regulation (regulation = 1 up, -1 down), then load into Cytoscape via KEGGScape and style differential expression on the -1200-to-600 gradient and read counts on the 0 to >=50,000 white-to-dark-blue scale.

## Related tools

- **FunHoP** (Pipeline that expands KGML multi-gene nodes with ElementTree, aggregates read counts per node, and computes node-level differential expression) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization of expanded and aggregated pathway networks with differential-expression and read-count color styling)
- **KEGGScape** (KGML-reader app used to import KEGG pathway files into Cytoscape)
- **Voom** (Differential expression analysis at gene and node level (DEGs at P < 0.05))
- **ElementTree** (Python XML API used to expand multi-gene KGML nodes, creating a new child per gene while keeping original coordinates)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Expanded network contains a distinct child node for every gene in each multi-gene node (e.g. the four-gene ALDH3A1 node or 21-gene PLA2G4B node), with OR nodes marked by white border
- Aggregated node-level table reproduces published read-count values: ALDH3A1 node dominated by ALDH1A3 with 46,595 reads; PLA2G2A 71,482 reads, PLA2G12A 4,502, PLA2G16 3,246
- Node-level significance changes after aggregation: the PLA2G4B node is non-significant in the original network but up-regulated in the aggregated network
- Histidine-pathway validation: CNDP2 up-regulated with ~11,000 reads while down-regulated CNDP1 has near-zero reads; HNMT (2,519 reads) and AOC1 (3,636 reads) down-regulated; CARNS1 < 100 reads
- Differential-expression styling falls within the -1200-to-600 gradient and read counts within the 0 to >=50,000 white-to-dark-blue scale

## Limitations

- KEGGScape only presents the first listed gene in a network node for a given reaction, which can lead to incorrect interpretations if not expanded
- Glycan-related KEGG pathways (category 1.7) use 'links' instead of nodes and cannot be expanded or coloured in Cytoscape; they are excluded from FunHoP
- Enzyme kinetics/efficiency data are usually unavailable, so the equal enzyme rate and highest-read-count assumptions are simplifications
- KEGG does not provide curated isoform-level enzyme activity data, limiting analysis to gene level; genes with atypical dominant isoforms may yield aberrant expression estimates
- Calculate_counts.py contains hard-coded assumptions: gene names in the first row, tissue status in column 7, and gene-length correction set via the 'correct' flag in load_countdata

## Evidence

- [abstract] KGML-reader KEGGScape only presents the first listed gene: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways."
- [abstract] FunHoP collapses genes using read counts: "FunHoP collapses all genes in a node into one measurement using read counts from RNA-seq."
- [methods] Voom DE and log2-transformed P values: "Voom [26] was further used for differential expression analysis"
- [methods] gene-length adjustment factor: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [readme] README: pathway XML download and glycan exclusion: "Download the wanted XMLs from KEGG. All pathways related to metabolism can be used."
- [readme] README: funhop.py invocation: "./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>"
