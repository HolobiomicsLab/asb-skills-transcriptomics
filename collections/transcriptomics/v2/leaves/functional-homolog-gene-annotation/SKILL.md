---
name: functional-homolog-gene-annotation
description: Use when KEGG KGML pathway files loaded into Cytoscape (KEGGScape) render
  only the first gene per multi-gene node, and the analysis goal is to surface all
  functionally homologous genes responsible for enzymatic reactions.
when_to_use_negative:
- The pathway is a glycan-related KEGG pathway (category 1.7 'Glycan biosynthesis
  and metabolism') whose 'line' link nodes cannot be expanded or colored in Cytoscape,
  and are hence excluded from FunHoP
- The pathway already displays all genes per node (e.g. data from tools other than
  KEGGScape that do not collapse OR groups), so expansion would duplicate nodes
- The goal is isoform-level enzyme activity analysis, since KEGG does not provide
  curated enzyme-activity information for isoforms and FunHoP limits analysis to the
  gene level
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0128
tools:
- name: Cytoscape
  role: Visualization platform into which the original and expanded KGML networks
    are loaded for pathway rendering
- name: KEGGScape
  role: Cytoscape app used to import KEGG KGML files; it displays only the first gene
    per node, which this skill corrects by expanding the XML
- name: FunHoP
  role: Python code implementing the node expansion and homolog aggregation; run via
    funhop.py on pathways, hsa list, expression table, metadata, expression counts,
    and updated gene symbols
  repo: https://github.com/kjerstirise/FunHoP
- name: ElementTree XML API
  role: Python XML parser used to identify multi-gene nodes and add child elements
    for each hidden homolog
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:ce7854eaf659f7988bb6592e9e57ebc4003985f3d0b4cd9ae8dfe1e93c23af6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - ElementTree XML API
  - FunHoP
  - KEGGScape
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# functional-homolog-gene-annotation

## Summary

A KGML-expansion method that rewrites KEGG pathway XML nodes so that every functional homolog gene behind a reaction is displayed in Cytoscape (via KEGGScape/FunHoP), instead of only the first listed gene, recovering hidden enzymes such as the 21 genes behind the PLA2G4B node in hsa00564.

## When to use

Use when KEGG KGML pathway files loaded into Cytoscape (KEGGScape) render only the first gene per multi-gene node, and the analysis goal is to surface all functionally homologous genes responsible for enzymatic reactions.

## When NOT to use

- The pathway is a glycan-related KEGG pathway (category 1.7 'Glycan biosynthesis and metabolism') whose 'line' link nodes cannot be expanded or colored in Cytoscape, and are hence excluded from FunHoP
- The pathway already displays all genes per node (e.g. data from tools other than KEGGScape that do not collapse OR groups), so expansion would duplicate nodes
- The goal is isoform-level enzyme activity analysis, since KEGG does not provide curated enzyme-activity information for isoforms and FunHoP limits analysis to the gene level

## Inputs

- KEGG KGML XML pathway files for human metabolic pathways (e.g. hsa00340, hsa00564)
- hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa
- Optional: RNA-seq read count matrix and expression table for downstream node-level aggregation
- Optional: updated gene symbols list to improve gene matching in KEGG pathways

## Outputs

- Expanded pathway XML files in which every multi-gene node lists all functional homolog genes (e.g. ALDH3A1 node carrying ALDH1A3, ALDH3B1, ALDH3B2; PLA2G4B node carrying 21 genes)
- Networks loadable in Cytoscape via KEGGScape with expanded homolog nodes visually distinguished from protein complexes
- A FunHoP summary of aggregated expression counts per node for creating count-based color schemes in Cytoscape

## How to apply

Download the KEGG KGML XML files for the human metabolic pathways of interest (e.g. hsa00340 histidine metabolism, hsa00564 glycerophospholipid metabolism) together with the hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa. Parse each XML with the ElementTree XML API and identify nodes whose name strings contain more than one gene (functional homolog nodes / OR groups). For each multi-gene node, create a new child element for every gene not present in the initial child and combine all children in a common 'group' node, reusing KEGG's protein-complex (AND-group) representation while keeping the original node coordinates so the expanded genes appear in place. Visually distinguish expanded homolog nodes from true protein complexes by rendering the gene nodes larger than default with a white field/border on each side. Emit expanded network XML files for all non-glycan metabolic pathways (71 of 85 pathways usable), then load them into Cytoscape v3.4 via KEGGScape or via the FunHoP scripts, which further collapse all genes in a node into one measurement using RNA-seq read counts weighted on gene length and ratio.

## Related tools

- **Cytoscape** (Visualization platform into which the original and expanded KGML networks are loaded for pathway rendering)
- **KEGGScape** (Cytoscape app used to import KEGG KGML files; it displays only the first gene per node, which this skill corrects by expanding the XML)
- **FunHoP** (Python code implementing the node expansion and homolog aggregation; run via funhop.py on pathways, hsa list, expression table, metadata, expression counts, and updated gene symbols) — https://github.com/kjerstirise/FunHoP
- **ElementTree XML API** (Python XML parser used to identify multi-gene nodes and add child elements for each hidden homolog)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- The ALDH3A1 node in hsa00340 now contains ALDH1A3, ALDH3B1, and ALDH3B2 alongside ALDH3A1 after expansion
- The PLA2G4B node in hsa00564 carries all 21 genes, matching the expanded networks reported in the paper (File S1)
- Expanded genes keep the original node coordinates so they appear in place in Cytoscape
- Expanded homolog nodes are visually distinguishable from protein complexes (larger gene nodes with white field/border on each side)
- All 71 non-glycan human metabolic pathways produce valid expanded XML; glycan pathways with 'line' link nodes are excluded

## Limitations

- Glycan-related KEGG pathways (category 1.7) are excluded because some paths use 'links' instead of nodes, which cannot be expanded or colored in Cytoscape
- KEGG does not currently provide curated information on enzyme activity of isoforms, so analysis is limited to the gene level
- The current version of FunHoP supports only the human metabolic pathways found in KEGG
- Calculate_counts.py contains hard coding: gene names expected in the first row, tissue types expected in column 7 of the metadata, and a 'correct' flag for gene-length correction
- FunHoP is not yet available as a Cytoscape app; conversion is in development

## Evidence

- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways."
- [methods] To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded: "To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded"
- [other] the ALDH3A1 node in the histidine metabolism pathway actually contains four genes: [evidence span withheld — anchor status: not_found]
- [readme] the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes: "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded even if they contained multiple genes"
- [methods] 85 pathways of relevance to human metabolism, from subcategories 1.1 up to and including 1.11, were downloaded: "85 pathways of relevance to human metabolism, from subcategories 1.1 up to and including 1.11, were downloaded with human genes from the KEGG pathway database"
