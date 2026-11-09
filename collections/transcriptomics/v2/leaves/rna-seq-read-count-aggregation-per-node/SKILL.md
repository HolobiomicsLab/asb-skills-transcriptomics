---
name: rna-seq-read-count-aggregation-per-node
description: Use when a KEGG pathway node contains multiple functionally homologous
  genes (e.g., the 4-gene ALDH3A1 node in hsa00340 or the 21-gene PLA2G4B node in
  hsa00564) and you need one node-level read count and regulation value instead of
  per-gene ambiguity.
when_to_use_negative:
- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored in Cytoscape and are excluded
  from FunHoP
- Analysis is restricted to human metabolic KEGG pathways at the gene level; KEGG
  provides no curated isoform-level enzyme activity, so isoform-resolution questions
  are out of scope
- When you only need per-gene (not node-level) regulation — the standard expanded
  network already shows every homolog individually
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: FunHoP
  role: Runs the node-expansion and per-node read-count aggregation pipeline on KGML
    files and expression counts
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization platform where aggregated networks are colored by node-level
    DE and read counts
- name: KEGGScape
  role: Cytoscape app used to import the KGML-derived aggregated networks; note it
    alone only presents the first listed gene per node
- name: Voom
  role: Differential expression analysis at both gene and node level (DEGs at P <
    0.05)
- name: ElementTree
  role: Python XML API used to expand multi-gene nodes before aggregation
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:47de943208b4d989f55e1b94b72e08cd6d7fb95662c11ef71abbb4ae27654b8f
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rna-seq-read-count-aggregation-per-node

## Summary

FunHoP-style aggregation that collapses all genes within a multi-gene KEGG pathway node into a single node-level measurement by summing RNA-seq read counts (adjusted for gene length), assuming reaction activity depends mainly on the highest-read homolog. This makes node-level differential expression and network coloring biologically interpretable.

## When to use

Use when a KEGG pathway node contains multiple functionally homologous genes (e.g., the 4-gene ALDH3A1 node in hsa00340 or the 21-gene PLA2G4B node in hsa00564) and you need one node-level read count and regulation value instead of per-gene ambiguity.

## When NOT to use

- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored in Cytoscape and are excluded from FunHoP
- Analysis is restricted to human metabolic KEGG pathways at the gene level; KEGG provides no curated isoform-level enzyme activity, so isoform-resolution questions are out of scope
- When you only need per-gene (not node-level) regulation — the standard expanded network already shows every homolog individually

## Inputs

- Raw RNA-seq read-count matrix (tab-separated, all samples, same gene ordering as the expression table)
- Expanded KEGG KGML pathway XML files (per-gene child nodes created by FunHoP's ElementTree expansion)
- hsa-ID-to-gene-name conversion list (e.g. from http://rest.kegg.jp/list/hsa)
- Gene expression table with gene names/IDs (gene names in column 0)
- Sample metadata with tissue status (cancer vs control, expected in column 7 by define_groups)
- Updated gene symbols list
- Gene lengths (sum of exons) for read-count length adjustment

## Outputs

- Aggregated pathway network files with one collapsed node per multi-gene reaction (node-level total read counts)
- Node-level differential expression values on the log2(P) × (−10) × regulation scale
- Summary table of expression/read counts per node for building Cytoscape color schemes
- Cytoscape-rendered aggregated network (e.g. Figure 3D/4 style) colored by DE gradient and read-count scale

## How to apply

First expand the KGML pathway XML with ElementTree so each multi-gene node gains a child per gene, then load raw RNA-seq read counts and adjust them for gene length by the factor (gene length as sum of exons) / average gene length over all genes. Sum the adjusted per-gene read counts within each aggregated node (nodes named 'gene1-Bx' style) to obtain a node-level total read count. Run voom differential expression (PCa vs normal, DEGs at P < 0.05) at the node level and log2-transform P values as Value = log2(P) × (−10) × regulation (regulation = 1 up, −1 down). The rationale is that enzymatic activity mainly depends on the gene with the highest number of reads, weighted on gene length and ratio, so node aggregation preserves all information while yielding one interpretable regulation value. In Cytoscape via KEGGScape, render the aggregated network on the −1200-to-600 gradient for DE and the 0-to-≥50,000 white-to-dark-blue scale for counts.

## Related tools

- **FunHoP** (Runs the node-expansion and per-node read-count aggregation pipeline on KGML files and expression counts) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization platform where aggregated networks are colored by node-level DE and read counts)
- **KEGGScape** (Cytoscape app used to import the KGML-derived aggregated networks; note it alone only presents the first listed gene per node)
- **Voom** (Differential expression analysis at both gene and node level (DEGs at P < 0.05))
- **ElementTree** (Python XML API used to expand multi-gene nodes before aggregation)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Histidine case (hsa00340): ALDH1A3 dominates the ALDH3A1 node with 46,595 reads; CNDP2 up-regulated with ~11,000 reads while down-regulated CNDP1 has near-zero reads; CARNS1 < 100 reads; HAL up-regulated with near-zero reads
- GPC case (hsa00564): PLA2G2A dominant with 71,482 reads, PLA2G12A 4,502, PLA2G16 3,246 — the aggregated table must reproduce these reported Figure 4 values
- PLA2G4B node must be non-significant in the original (expanded) network but up-regulated in the aggregated network — aggregation should change node-level significance where single dominant genes mask node flux
- All read counts must be length-adjusted by (gene length as sum of exons) / average gene length over all genes before summing within nodes
- Node-level DE values must fall on the −1200-to-600 gradient scale after Value = log2(P) × (−10) × regulation transformation

## Limitations

- Enzyme kinetics/efficiency data are usually not available, so assuming equal enzyme rates and prioritizing by read count is a simplification
- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation
- Glycan-related KEGG pathways (category 1.7) are excluded because some use 'links' instead of nodes and cannot be expanded or colored
- calculate_counts.py contains hard-coded assumptions: gene names in the first row, tissue status in column 7 of metadata, and the 'correct' flag for gene-length correction must be set manually in load_countdata
- FunHoP is not yet available as a Cytoscape app; conversion is in development

## Evidence

- [abstract] FunHoP collapses all genes in a node into one measurement using read counts from RNA-seq.: "FunHoP collapses all genes in a node into one measurement using read counts from RNA-seq."
- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] To make more interpretable networks yet containing all the information, all genes within a node were aggregated into one: "To make more interpretable networks yet containing all the information, all genes within a node were aggregated into one"
- [readme] The file "Calculate_counts.py" still contains three cases of hard coding:: "The file "Calculate_counts.py" still contains three cases of hard coding:"
- [discussion] genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation: "genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation"
