---
name: metabolic-flux-interpretation-from-gene-expression
description: Use when a KEGG pathway node contains multiple functionally homologous
  genes (e.g. the 21-gene PLA2G4B node) and you need to identify the dominant enzyme,
  node-level regulation, and net metabolic flux from RNA-seq data.
when_to_use_negative:
- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored in Cytoscape and are excluded
  from FunHoP
- Non-human pathways — the current FunHoP version supports only human metabolic pathways
  found in KEGG (excluding glycan-related pathways)
- Analyses requiring isoform- or enzyme-kinetics-level activity, since KEGG provides
  no curated isoform enzyme-activity data and kinetics data are usually unavailable
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: FunHoP
  role: Python pipeline (funhop.py) that expands multi-gene KEGG nodes and collapses
    homolog read counts into node-level measurements
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization platform for the expanded/aggregated metabolic networks
- name: KEGGScape
  role: Cytoscape app used to load KEGG KGML network files; only presents the first
    listed gene per node, which FunHoP corrects
- name: Voom
  role: Differential expression analysis on gene- and node-level counts (DEGs at P
    < 0.05)
- name: ElementTree XML API
  role: Expands KGML XML nodes into one child per gene, preserving coordinates and
    marking OR nodes with a white border
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:101ced234b154a282386da4e3334c6ba4f5de8067209c140cc1f44986cd3904b
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
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolic-flux-interpretation-from-gene-expression

## Summary

Interprets overall metabolic node regulation and inferred substrate flux direction in KEGG metabolic pathways by collapsing multi-gene functionally homologous nodes into single read-count-based measurements, assuming reaction activity depends mainly on the gene with the highest number of RNA-seq reads.

## When to use

Use when a KEGG pathway node contains multiple functionally homologous genes (e.g. the 21-gene PLA2G4B node) and you need to identify the dominant enzyme, node-level regulation, and net metabolic flux from RNA-seq data.

## When NOT to use

- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored in Cytoscape and are excluded from FunHoP
- Non-human pathways — the current FunHoP version supports only human metabolic pathways found in KEGG (excluding glycan-related pathways)
- Analyses requiring isoform- or enzyme-kinetics-level activity, since KEGG provides no curated isoform enzyme-activity data and kinetics data are usually unavailable

## Inputs

- KEGG KGML pathway XML file (e.g. hsa00564 glycerophospholipid metabolism)
- hsa ID to gene-name conversion list (http://rest.kegg.jp/list/hsa)
- Gene expression table (tab-separated; gene names, gene IDs, probe values)
- Sample metadata table (cancer vs control tissue status, e.g. column 6)
- Raw RNA-seq read count matrix (tab-separated, genes x samples)
- Updated gene symbols list

## Outputs

- Expanded and aggregated KEGG network files with all homologs shown per node and node-level aggregated read counts
- Node-level read-count summary used to build Cytoscape color schemes
- Gene- and node-level differential expression values (log2(P) x (-10) x regulation)
- Cytoscape/KEGGScape visualization with DE gradient (-1200 to 600) and read-count scale (0 to >=50,000)
- Interpreted node regulation and inferred net metabolite flux (e.g. lecithin toward GPC)

## How to apply

Run funhop.py on the TCGA expression table, metadata, raw read counts, hsa ID/gene-name list, and updated gene symbols with the KEGG KGML file for the pathway of interest (e.g. hsa00564 glycerophospholipid metabolism) to generate expanded and aggregated networks. Expand multi-gene nodes with the ElementTree XML API so each gene becomes a child node keeping original coordinates, marking functional-homolog (OR) nodes with a white border. Adjust read counts for gene length (sum of exons divided by average gene length) and sum per-gene counts within each node to get node-level totals; apply voom differential expression with DEGs at P < 0.05, log2-transformed as Value = log2(P) x (-10) x regulation. Load the networks into Cytoscape via KEGGScape, styling differential expression on a -1200 to 600 gradient and read counts on a 0 to >=50,000 white-to-dark-blue scale, then read dominant enzymes (highest read counts) and up/down-regulated genes to infer net flux, e.g. lecithin toward GPC when PLA2G2A dominates and LYPLA1/LYPLA2 are up-regulated while LCAT down-regulation is minor.

## Related tools

- **FunHoP** (Python pipeline (funhop.py) that expands multi-gene KEGG nodes and collapses homolog read counts into node-level measurements) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization platform for the expanded/aggregated metabolic networks)
- **KEGGScape** (Cytoscape app used to load KEGG KGML network files; only presents the first listed gene per node, which FunHoP corrects)
- **Voom** (Differential expression analysis on gene- and node-level counts (DEGs at P < 0.05))
- **ElementTree XML API** (Expands KGML XML nodes into one child per gene, preserving coordinates and marking OR nodes with a white border)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Output table reproduces reported Figure 4 values: PLA2G2A dominant with 71,482 reads, PLA2G12A 4,502 reads, PLA2G16 3,246 reads
- The PLA2G4B node is non-significant in the original network but up-regulated in the aggregated network
- Node-level read counts equal the gene-length-adjusted sum of per-gene counts within each node
- DEGs match voom results at P < 0.05 and values follow Value = log2(P) x (-10) x regulation
- funhop.py prints completed pathway steps and ends with a summary of expression counts usable for Cytoscape color schemes

## Limitations

- Assumes equal enzyme rates since enzyme kinetics data are usually unavailable — a simplification
- KEGG does not provide curated isoform-level enzyme activity data, so analysis is limited to gene level
- Genes with particularly short or long dominant isoforms vs the canonical isoform model may yield aberrant expression estimates
- calculate_counts.py contains hard-coded expectations: gene names in first row, tissue status in column 7, and a 'correct' flag for gene-length adjustment
- FunHoP is not yet available as a Cytoscape app; glycan-related pathways (category 1.7) using 'links' are excluded

## Evidence

- [results] FunHoP addresses multi-gene nodes by collapsing all genes in a node into one measurement, assuming the enzymatic reaction activity depends mainly on the gene with the highest number of RNA-seq reads.: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads"
- [methods] Read counts are adjusted for gene length before aggregation.: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] P values are log2 transformed with regulation sign for visualization.: "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [abstract] KEGGScape's default behavior can mislead interpretation.: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways."
- [readme] Glycan pathways cannot be handled.: "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded even if they contained multiple genes"
