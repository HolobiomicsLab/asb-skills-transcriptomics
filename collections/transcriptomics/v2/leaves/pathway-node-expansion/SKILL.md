---
name: pathway-node-expansion
description: Use when KEGG KGML pathway nodes carry multiple gene names (functional
  homologs/OR groups) and Cytoscape/KEGGScape shows only the first gene, hiding candidate
  enzymes for a reaction.
when_to_use_negative:
- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' whose reactions
  use 'link' entries instead of nodes — these cannot be expanded or coloured in Cytoscape
  and are excluded from FunHoP
- When the goal is collapsed, single-value node expression rather than per-gene visibility
  — FunHoP's aggregation step collapses all homologs in a node into one RNA-seq read-count
  measurement
- Non-human or non-metabolic KEGG pathways — the current FunHoP version supports only
  human metabolic pathways (with glycan exceptions)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: KEGGScape
  role: Cytoscape app used to load the original and expanded KGML XML files and render
    the baseline vs expanded pathway networks
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Platform for visualizing the expanded networks; expanded homolog nodes are
    distinguished from complexes by larger node size and white field/border
- name: FunHoP
  role: Python code implementing KGML node expansion, gene aggregation and pathway
    emission (71 non-glycan metabolic pathways)
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:6d3deebe5882443cabd1edf4e0abd256ff527be8147363450f67ddb13db8dea2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pathway-node-expansion

## Summary

Expands KEGG KGML pathway XML nodes containing multiple functional homologs so that every responsible gene is rendered in Cytoscape via KEGGScape, instead of only the first listed gene, by rewriting the KGML with ElementTree and reusing KEGG's protein-complex group-node solution.

## When to use

Use when KEGG KGML pathway nodes carry multiple gene names (functional homologs/OR groups) and Cytoscape/KEGGScape shows only the first gene, hiding candidate enzymes for a reaction.

## When NOT to use

- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' whose reactions use 'link' entries instead of nodes — these cannot be expanded or coloured in Cytoscape and are excluded from FunHoP
- When the goal is collapsed, single-value node expression rather than per-gene visibility — FunHoP's aggregation step collapses all homologs in a node into one RNA-seq read-count measurement
- Non-human or non-metabolic KEGG pathways — the current FunHoP version supports only human metabolic pathways (with glycan exceptions)

## Inputs

- KEGG KGML pathway XML files (human metabolic pathways, hsa IDs, e.g. hsa00340, hsa00564)
- hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa (tab-separated)
- Updated gene symbols list to recover more genes from the KEGG pathways

## Outputs

- Expanded KGML-derived network XML files in which every multi-gene node lists all functional homologs as children of a common group node
- Cytoscape networks (loaded via KEGGScape) showing expanded homolog nodes with enlarged node size and white field/border to distinguish them from protein complexes

## How to apply

Download the KEGG KGML XML files for the human metabolic pathways (e.g. hsa00340 histidine metabolism, hsa00564 glycerophospholipid metabolism) together with the hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa. Load the original XMLs into Cytoscape v3.4 via KEGGScape to establish the baseline rendering where only the first gene of each node appears. Parse each XML with the ElementTree XML API and identify nodes whose name strings contain more than one gene; for each such node, create a new child element for every gene not present initially and combine new and old children in a common 'group' node, reusing KEGG's protein-complex (AND-group) solution while keeping the original node coordinates so expanded genes appear in place. Visually distinguish expanded homolog nodes from true protein complexes by rendering the gene nodes larger than the default size with a white field/border on each side. Emit expanded network XML files for all non-glycan metabolic pathways (71 of the 85 downloaded pathways are usable); the FunHoP repo executes this via its expansion functions.

## Related tools

- **KEGGScape** (Cytoscape app used to load the original and expanded KGML XML files and render the baseline vs expanded pathway networks) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Platform for visualizing the expanded networks; expanded homolog nodes are distinguished from complexes by larger node size and white field/border)
- **FunHoP** (Python code implementing KGML node expansion, gene aggregation and pathway emission (71 non-glycan metabolic pathways)) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- The ALDH3A1 node in hsa00340 (histidine metabolism) now contains ALDH3A1, ALDH1A3, ALDH3B1 and ALDH3B2 as group children
- The PLA2G4B node in hsa00564 (glycerophospholipid metabolism) carries all 21 genes, matching the expanded networks in the paper's File S1
- Original node coordinates are preserved so expanded genes appear in place in the Cytoscape rendering
- Expanded homolog nodes are visually distinguishable from true protein complexes (larger than default node size, white field/border on each side)
- 71 of the 85 downloaded metabolic pathways emit usable expanded XMLs; pathways with 'line' link nodes (category 1.7 glycans) are excluded

## Limitations

- KEGG category 1.7 glycan-related pathways cannot be expanded because they use 'links' instead of nodes, which also cannot be coloured in Cytoscape
- FunHoP is not yet available as a Cytoscape app; conversion is in development
- No version numbers were given for FunHoP, Cytoscape, or KEGGScape; Cytoscape v3.4 is referenced in the workflow
- The FunHoP code contains hard-coded assumptions (gene names in first row of expression table, tissue status in metadata column 7, gene-length correction flag in load_countdata)

## Evidence

- [results] Expanding the KGML nodes recovers the hidden functional homologs: the ALDH3A1 node in the histidine metabolism pathway actually contains four genes: [evidence span withheld — anchor status: not_found]
- [methods] To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded: "To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded"
- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction"
- [readme] the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded: "Notice that the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded"
- [readme] This list can be found here: http://rest.kegg.jp/list/hsa: "This list can be found here: http://rest.kegg.jp/list/hsa"
