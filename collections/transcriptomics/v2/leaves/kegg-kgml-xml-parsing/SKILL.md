---
name: kegg-kgml-xml-parsing
description: Use when KEGG KGML pathway XMLs must be inspected or rewritten in Cytoscape
  so that multi-gene nodes (functional homolog / OR groups) show all responsible genes,
  not just the first listed gene that KEGGScape renders by default.
when_to_use_negative:
- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'line'
  link nodes instead of nodes — these cannot be expanded or coloured in Cytoscape
  and are excluded from FunHoP
- The goal is only the baseline KEGGScape rendering showing the first listed gene
  per node — no XML rewriting is needed in that case
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0602
tools:
- name: Cytoscape
  role: Visualization platform into which original and expanded KGML XML files are
    loaded to render pathway networks
- name: KEGGScape
  role: Cytoscape app used to load KEGG KGML XML files; baseline reader that only
    displays the first listed gene per node, motivating the XML expansion
- name: FunHoP
  role: Method/code implementing the KGML node-expansion and gene-collapsing pipeline
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:ac20690a269cc6aba79325cf8b0ff66d9295b061c216c2f44feffdd30a417298
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGG
  - KEGGScape
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kegg-kgml-xml-parsing

## Summary

Parse KEGG KGML XML pathway files with Python's ElementTree API to identify nodes whose name strings contain multiple functional homolog genes, and rewrite the XML so every gene in a multi-gene node is represented as a child element grouped under the original node while preserving node coordinates.

## When to use

Use when KEGG KGML pathway XMLs must be inspected or rewritten in Cytoscape so that multi-gene nodes (functional homolog / OR groups) show all responsible genes, not just the first listed gene that KEGGScape renders by default.

## When NOT to use

- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'line' link nodes instead of nodes — these cannot be expanded or coloured in Cytoscape and are excluded from FunHoP
- The goal is only the baseline KEGGScape rendering showing the first listed gene per node — no XML rewriting is needed in that case

## Inputs

- KEGG KGML XML pathway files (e.g. hsa00340, hsa00564)
- hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa

## Outputs

- Expanded KGML network XML files in which every gene of each multi-gene node is a child element grouped under the original node
- Cytoscape networks (via KEGGScape) rendering all functional homologs, with expanded nodes visually distinguished from protein complexes

## How to apply

Download the KEGG KGML XML files for the human metabolic pathways of interest (e.g. hsa00340 histidine metabolism, hsa00564 glycerophospholipid metabolism) together with the hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa. Load the original XMLs into Cytoscape (v3.4) via KEGGScape to establish the baseline rendering, then parse each XML with the ElementTree XML API and identify nodes whose name strings contain more than one gene. For each multi-gene node, create a new child element for every gene absent from the initial child and combine all children in a common 'group' node, reusing KEGG's protein-complex (AND-group) solution while keeping the original node coordinates so expanded genes appear in place. Visually distinguish expanded homolog nodes from true protein complexes by rendering the gene nodes larger than default with a white field/border. Emit expanded network XML files; in the FunHoP work 71 of 85 non-glycan metabolic pathways were usable, excluding pathways with 'line' link nodes such as category 1.7 glycans.

## Related tools

- **Cytoscape** (Visualization platform into which original and expanded KGML XML files are loaded to render pathway networks)
- **KEGGScape** (Cytoscape app used to load KEGG KGML XML files; baseline reader that only displays the first listed gene per node, motivating the XML expansion)
- **FunHoP** (Method/code implementing the KGML node-expansion and gene-collapsing pipeline) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- The ALDH3A1 node in hsa00340 (histidine metabolism) contains four genes after expansion: ALDH3A1, ALDH1A3, ALDH3B1, and ALDH3B2
- The PLA2G4B node in hsa00564 (glycerophosphocholine metabolism) carries all 21 genes after expansion, matching the paper's expanded networks (File S1)
- Original node coordinates are preserved so expanded genes appear in place of the original node
- Expanded homolog nodes are distinguishable from true protein-complex AND-groups by larger node size and a white field/border
- Usable expanded networks correspond to 71 of the 85 non-glycan metabolic pathways

## Limitations

- Glycan-related KEGG pathways (mostly category 1.7) are excluded because they contain 'line' link nodes that cannot be expanded or coloured in Cytoscape
- KEGGScape is not modified directly; the expansion works by rewriting KGML XML upstream of KEGGScape, reusing the AND-group complex solution for OR-group homologs
- The current FunHoP version supports only human metabolic KEGG pathways, and pathway/hsa-list files should be periodically re-downloaded as KEGG is constantly updated

## Evidence

- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways.: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways."
- [methods] To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded: "To expand the XML files to show all the genes in all the nodes, the list of human IDs and corresponding gene names was downloaded"
- [other] Parse each XML with the ElementTree XML API and identify nodes whose name strings contain more than one gene (functional homolog nodes, OR groups).: [evidence span withheld — anchor status: not_found]
- [readme] the pathways from KEGG category 1.7 'glycol biosynthesis and metabolism' contains some paths that use 'links' instead of nodes, and these cannot be expanded even if they contained multiple genes.: "the pathways from KEGG category 1.7 'glycol biosynthesis and metabolism' contains some paths that use 'links' instead of nodes, and these cannot be expanded even if they contained multiple genes."
- [readme] This list can be found here: http://rest.kegg.jp/list/hsa: "This list can be found here: http://rest.kegg.jp/list/hsa"
