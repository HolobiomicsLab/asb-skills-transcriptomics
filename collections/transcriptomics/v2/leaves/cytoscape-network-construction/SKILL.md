---
name: cytoscape-network-construction
description: Use when KEGG KGML pathway files loaded via KEGGScape show only the first
  listed gene per reaction node, and the goal is to render all functional homolog
  genes in Cytoscape.
when_to_use_negative:
- Glycan-related KEGG pathways (category 1.7) that use 'line' link nodes instead of
  nodes, which cannot be expanded or coloured in Cytoscape
- Nodes already containing a single gene only — expansion is unnecessary and the KEGGScape
  baseline rendering suffices
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: Cytoscape
  role: Visualization platform into which the original and expanded KGML networks
    are loaded
- name: KEGGScape
  role: KGML-reader app used to load KEGG XML pathway files into Cytoscape and establish
    the baseline rendering
- name: FunHoP
  role: Python scripts that parse KGML XML, expand multi-gene nodes, and emit expanded
    network XML files
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:0bfcf9054317fb8fbeb8bab0ba113f682eb924f37ed3f3fb2a266be6fc83e862
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cytoscape-network-construction

## Summary

Constructs Cytoscape pathway networks from KEGG KGML files in which multi-gene nodes (functional homolog OR-groups) are expanded so every responsible gene is displayed, reusing KEGG's protein-complex group-node solution and original node coordinates.

## When to use

Use when KEGG KGML pathway files loaded via KEGGScape show only the first listed gene per reaction node, and the goal is to render all functional homolog genes in Cytoscape.

## When NOT to use

- Glycan-related KEGG pathways (category 1.7) that use 'line' link nodes instead of nodes, which cannot be expanded or coloured in Cytoscape
- Nodes already containing a single gene only — expansion is unnecessary and the KEGGScape baseline rendering suffices

## Inputs

- KEGG KGML XML pathway files (e.g. hsa00340, hsa00564)
- hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa

## Outputs

- Expanded KGML/network XML files showing all genes per multi-gene node (71 usable non-glycan metabolic pathways)
- Cytoscape-ready networks in which expanded homolog nodes are visually distinguishable (larger nodes, white field/border) from protein complexes

## How to apply

Download the KGML XML files for the human metabolic pathways (e.g. hsa00340 histidine metabolism, hsa00564 glycerophospholipid metabolism) together with the hsa ID-to-gene-name list from http://rest.kegg.jp/list/hsa. Parse each XML with Python's ElementTree API and identify nodes whose name strings contain more than one gene (functional homolog/OR-group nodes). For each such node, create a new child element for every gene not present in the initial child, combine new and old children in a common 'group' node that reuses KEGG's protein-complex (AND-group) solution, keeping the original node coordinates so expanded genes appear in place. Visually distinguish expanded homolog nodes from true protein complexes by rendering the gene nodes larger than the default size with a white field/border. Emit expanded network XML files; KEGGScape then renders all genes when the files are loaded into Cytoscape.

## Related tools

- **Cytoscape** (Visualization platform into which the original and expanded KGML networks are loaded)
- **KEGGScape** (KGML-reader app used to load KEGG XML pathway files into Cytoscape and establish the baseline rendering)
- **FunHoP** (Python scripts that parse KGML XML, expand multi-gene nodes, and emit expanded network XML files) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- The ALDH3A1 node in hsa00340 contains four genes after expansion: ALDH3A1, ALDH1A3, ALDH3B1, ALDH3B2
- The PLA2G4B node in hsa00564 carries all 21 genes, matching the expanded networks in the paper's File S1
- Expanded genes appear at the original node coordinates (position attributes preserved)
- Exactly 71 of the 85 metabolic pathways are usable; the 14 pathways with 'line' link nodes (e.g. glycans) are excluded
- Expanded homolog nodes are distinguishable from protein AND-complexes by larger size and white field/border styling

## Limitations

- Glycan-related pathways (KEGG category 1.7) are excluded because they use 'line' links instead of nodes, which cannot be expanded or coloured in Cytoscape
- FunHoP is not yet a Cytoscape app; XML files must be pre-processed outside Cytoscape (app conversion in development)
- The load_countdata/genenames workflow contains hard-coding (gene names in first row, tissue status in column 7, gene-length correction flag)

## Evidence

- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction"
- [methods] To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded: "To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded"
- [methods] 85 pathways of relevance to human metabolism, from subcategories 1.1 up to and including 1.11, were downloaded with human genes from the KEGG pathway database: "85 pathways of relevance to human metabolism, from subcategories 1.1 up to and including 1.11, were downloaded with human genes from the KEGG pathway database"
- [readme] the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes: "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes"
- [discussion] Converting FunHoP into a Cytoscape app is also in development, which will make it easier for all users to apply this method to their own analyses: "Converting FunHoP into a Cytoscape app is also in development, which will make it easier for all users to apply this method to their own analyses"
