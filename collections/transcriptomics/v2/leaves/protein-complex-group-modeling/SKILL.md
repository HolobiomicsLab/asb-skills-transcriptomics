---
name: protein-complex-group-modeling
description: Use when KEGG KGML pathway files are loaded in Cytoscape via KEGGScape
  and nodes whose name strings contain more than one gene (functional homolog or OR-group
  nodes) display only the first gene, hiding the others.
when_to_use_negative:
- Glycan-related KEGG pathways (mostly category 1.7 'Glycan biosynthesis and metabolism')
  that use 'line' link nodes instead of nodes — these cannot be expanded even if they
  contain multiple genes and cannot be coloured in Cytoscape
- Pathways where only the first listed gene per node is an acceptable rendering, i.e.
  when no functional homolog (OR-group) expansion is needed
- When the goal is collapsing multi-gene nodes into a single expression value rather
  than expanding and displaying every gene (that is FunHoP's count-aggregation step,
  a separate concern)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0602
tools:
- name: KEGG
  role: Source of KGML pathway XML files and the hsa ID-to-gene-name conversion list
- name: KEGGScape
  role: KGML-reader app used to load the KGML files into Cytoscape; baseline reader
    that only presents the first listed gene per node, motivating the expansion
- name: Cytoscape
  role: Visualization platform (v3.4) in which the expanded group nodes are rendered
    with original node coordinates and custom node size/border styling
- name: FunHoP
  role: Method/tool that implements the KGML expansion and grouping of functional
    homologs into group nodes
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:58bee0885edeab8cf523d096df6758c968edc3f8c4142522c74b2e38e6d2b9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGG
  - KEGGScape
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# protein-complex-group-modeling

## Summary

Expands KEGG KGML pathway nodes that contain multiple functional homologs by parsing the KGML XML with ElementTree and regrouping every hidden gene into a common 'group' node, reusing KEGG's protein-complex (AND-group) representation so Cytoscape/KEGGScape renders all responsible genes instead of only the first listed one.

## When to use

Use when KEGG KGML pathway files are loaded in Cytoscape via KEGGScape and nodes whose name strings contain more than one gene (functional homolog or OR-group nodes) display only the first gene, hiding the others.

## When NOT to use

- Glycan-related KEGG pathways (mostly category 1.7 'Glycan biosynthesis and metabolism') that use 'line' link nodes instead of nodes — these cannot be expanded even if they contain multiple genes and cannot be coloured in Cytoscape
- Pathways where only the first listed gene per node is an acceptable rendering, i.e. when no functional homolog (OR-group) expansion is needed
- When the goal is collapsing multi-gene nodes into a single expression value rather than expanding and displaying every gene (that is FunHoP's count-aggregation step, a separate concern)

## Inputs

- KEGG KGML XML pathway files for human metabolic pathways (e.g. hsa00340, hsa00564)
- hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa (tab-separated values)

## Outputs

- Expanded KGML/network XML files in which each multi-gene node is a 'group' node containing child elements for all functional homolog genes (71 usable pathways of the 85 downloaded)
- Cytoscape networks in which expanded homolog nodes appear in place at original coordinates, visually distinct from protein complexes by larger node size with white field/border

## How to apply

Download the KGML XML files for the human metabolic pathways (e.g. hsa00340 histidine metabolism, hsa00564 glycerophospholipid metabolism) together with the hsa ID-to-gene-name list from http://rest.kegg.jp/list/hsa. Load the original XML into Cytoscape (v3.4) with KEGGScape to establish the baseline rendering, then parse each XML with the ElementTree XML API and identify nodes whose name strings contain more than one gene. For each multi-gene node, create a new child element for every gene not present in the initial child and combine the new children with the old child in a common 'group' node, reusing KEGG's protein-complex (AND-group) solution while keeping the original node coordinates so the expanded genes appear in place. Visually distinguish expanded homolog nodes from true protein complexes by rendering the gene nodes larger than the default size with a white field/border on each side. Emit expanded network XML files for all usable non-glycan metabolic pathways (71 of the 85 downloaded pathways; exclude those with 'line' link nodes such as category 1.7 glycans).

## Related tools

- **KEGG** (Source of KGML pathway XML files and the hsa ID-to-gene-name conversion list)
- **KEGGScape** (KGML-reader app used to load the KGML files into Cytoscape; baseline reader that only presents the first listed gene per node, motivating the expansion)
- **Cytoscape** (Visualization platform (v3.4) in which the expanded group nodes are rendered with original node coordinates and custom node size/border styling)
- **FunHoP** (Method/tool that implements the KGML expansion and grouping of functional homologs into group nodes) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- The ALDH3A1 node in hsa00340 (histidine metabolism) now contains ALDH3A1, ALDH1A3, ALDH3B1, and ALDH3B2 as children in the expanded XML
- The PLA2G4B node in hsa00564 (glycerophosphocholine/glycerophospholipid metabolism) carries all 21 genes, matching the expanded networks in the paper's File S1
- Expanded genes appear in place at the original node coordinates of the multi-gene node
- Expanded homolog nodes are visually distinguishable from true protein complexes (larger than default node size, white field/border on each side)
- Exactly 71 of the 85 downloaded non-glycan metabolic pathways produce expanded XML files; those with 'line' link nodes (category 1.7) are excluded

## Limitations

- Glycan-related KEGG pathways (category 1.7) contain 'line' link nodes that cannot be expanded or coloured in Cytoscape and are therefore excluded
- Grouping by AND-group (protein-complex) representation means expanded homolog nodes structurally resemble true complexes, requiring manual visual styling to distinguish them
- Cytoscape 3.4 and KEGGScape are the validated versions; FunHoP is not yet available as a Cytoscape app, so the XML expansion must be run outside Cytoscape

## Evidence

- [other] Parse each XML with the ElementTree XML API and identify nodes whose name strings contain more than one gene (functional homolog nodes, OR groups).: [evidence span withheld — anchor status: not_found]
- [other] For each multi-gene node, create a new child element for every gene not present in the initial child and combine the new children with the old child in a common 'group' node.: [evidence span withheld — anchor status: not_found]
- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways.: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations"
- [results] the ALDH3A1 node in the histidine metabolism pathway actually contains four genes (ALDH3A1, ALDH1A3, ALDH3B1, and ALDH3B2), and the PLA2G4B node in glycerophosphocholine metabolism contains 21 genes: [evidence span withheld — anchor status: not_found]
- [readme] the pathways from KEGG category 1.7 'glycol biosynthesis and metabolism' contains some paths that use 'links' instead of nodes, and these cannot be expanded even if they contained multiple genes.: "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded"
