---
name: kegg-network-node-name-mapping
description: Use when KGML pathway XMLs have been expanded to show all functionally
  homologous genes per node and node names must be unified (e.g., as 'PLA2G4B-B21')
  before mapping node-level differential-expression values in Cytoscape.
when_to_use_negative:
- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- Non-human pathway networks, since the current FunHoP version and the hsa conversion
  list support only human metabolic pathways
- Nodes you intend to keep at single-gene granularity (e.g., for standard KEGGScape
  rendering), where renaming to gene1-Bx is unnecessary
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0797
tools:
- name: FunHoP
  role: Implements node expansion and gene1-Bx node renaming over KGML pathway XMLs
  repo: https://github.com/kjerstirise/FunHoP
- name: KEGGScape
  role: KGML reader that loads KEGG pathways into Cytoscape; by default shows only
    the first listed gene per node, motivating the gene1-Bx renaming
- name: Cytoscape
  role: Renders the renamed gene1-Bx nodes with the differential-expression color
    gradient
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:730ae8e611fef5e72283c683feb38d4337e4ddcc118e13c55ad3b122876a031f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kegg-network-node-name-mapping

## Summary

Renames KEGG-derived metabolic network nodes from raw KGML gene-name strings to the FunHoP 'gene1-Bx' convention (first gene name plus number of genes in the node), so expanded multi-gene nodes and aggregated node-level expression values can be colored and traced consistently in Cytoscape.

## When to use

Use when KGML pathway XMLs have been expanded to show all functionally homologous genes per node and node names must be unified (e.g., as 'PLA2G4B-B21') before mapping node-level differential-expression values in Cytoscape.

## When NOT to use

- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- Non-human pathway networks, since the current FunHoP version and the hsa conversion list support only human metabolic pathways
- Nodes you intend to keep at single-gene granularity (e.g., for standard KEGGScape rendering), where renaming to gene1-Bx is unnecessary

## Inputs

- Expanded KEGG KGML pathway XML files (File S1, 85 human metabolism pathways, categories 1.1–1.11)
- hsa-ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa
- Updated gene symbols list for matching KEGG genes

## Outputs

- Network files with nodes renamed under the gene1-Bx convention (e.g., PLA2G4B-B21 for the 21-gene node)
- Aggregated network files whose node names match the node-level differential-expression values for Cytoscape coloring

## How to apply

First expand the KEGG KGML XML files with the human hsa-ID-to-gene-name conversion list (from http://rest.kegg.jp/list/hsa) so every gene in each reaction node is visible. Extract gene-name strings from all network files, define the set of unique nodes, and rename each node 'gene1-Bx', where gene1 is the first gene name in the node string and x is the number of genes in that node (x = 1 for single-gene nodes). After aggregating read counts within nodes, re-adapt the aggregated network files to the gene1-Bx names so the Cytoscape differential-expression color gradient (from −1200 black through −600 purple, −300 bright red, 0 light yellow, 300 bright green, to 600 dark green) maps onto the same node identifiers as the expression summary. Exclude KEGG category 1.7 (glycan biosynthesis and metabolism) pathways that use 'links' instead of nodes, since they cannot be expanded or colored.

## Related tools

- **FunHoP** (Implements node expansion and gene1-Bx node renaming over KGML pathway XMLs) — https://github.com/kjerstirise/FunHoP
- **KEGGScape** (KGML reader that loads KEGG pathways into Cytoscape; by default shows only the first listed gene per node, motivating the gene1-Bx renaming)
- **Cytoscape** (Renders the renamed gene1-Bx nodes with the differential-expression color gradient)

## Examples

```
./funhop.py hsa_list.txt expression_table.tsv metadata.tsv expression_counts.tsv updated_gene_symbols.txt
```

## Evaluation signals

- Every node name matches the gene1-Bx pattern, where gene1 equals the first gene name in the node string and Bx equals the number of genes in the node (B1 for single-gene nodes)
- Node counts are consistent before/after renaming — the number of unique nodes equals the number of unique gene1-Bx names
- The 21-gene PLA2G4B node is renamed PLA2G4B-B21 and its regulation reproduces the reported behavior: up-regulated via dominant PLA2G2A (71,482 reads)
- Aggregated network files open in Cytoscape with the color gradient applied on the same gene1-Bx identifiers as the node-level expression summary

## Limitations

- FunHoP currently supports only human metabolic KEGG pathways, with glycan-related pathways (category 1.7) excluded
- KEGGScape presents only the first listed gene in a network node for a given reaction, which can lead to incorrect interpretations if the gene1-Bx mapping is not applied
- KEGG does not provide curated isoform-level enzyme activity, so analysis is limited to the gene level

## Evidence

- [methods] Extract gene-name strings from all network files, define unique nodes, and rename each node as 'gene1-Bx', where gene1 is the first gene name in the node string and x is the number of genes in the node (x = 1 for single-gene nodes).: [evidence span withheld — anchor status: not_found]
- [methods] Adapt the aggregated network files to the gene1-Bx node names and reapply the differential-expression color gradient in Cytoscape.: [evidence span withheld — anchor status: not_found]
- [readme] the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded even if they contained multiple genes: "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes"
- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction"
- [readme] This is a simple copy-paste of the KEGG database of hsa IDs and the belonging gene names. This list can be found here: http://rest.kegg.jp/list/hsa: "This is a simple copy-paste of the KEGG database of hsa IDs and the belonging gene names. This list can be found here"
