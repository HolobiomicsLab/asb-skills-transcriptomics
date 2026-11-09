---
name: cytoscape-network-color-styling
description: Use when FunHoP-expanded or aggregated KEGG network files (KGML-derived,
  node names like gene1-Bx) are loaded into Cytoscape via KEGGScape and must be colored
  by differential expression and read counts.
when_to_use_negative:
- KEGG glycan-related pathways (category 1.7) that use 'links' instead of nodes, which
  cannot be expanded or colored similarly in Cytoscape
- Networks whose nodes are not FunHoP-expanded (i.e. multi-gene homologs still collapsed
  by KEGGScape, which only shows the first listed gene)
- Read counts that have not been adjusted for gene length, since the count-based color
  mapping assumes length-corrected values
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Cytoscape
  role: Visualization environment in which the expanded/aggregated networks are loaded
    and styled with the DE and read-count color gradients
  repo: https://github.com/kjerstirise/FunHoP
- name: KEGGScape
  role: KGML-reader app used to import KEGG pathway networks into Cytoscape (noting
    it shows only the first gene per node, hence the FunHoP expansion)
- name: FunHoP
  role: Generates the expanded and aggregated network files and prints the expression-count
    summary used to define the count color scheme
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:843bc5c9accff08b9386aab9300fdc17ad3f47903015434b9d735583eb9476ee
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cytoscape-network-color-styling

## Summary

Style FunHoP-expanded and aggregated KEGG metabolic networks in Cytoscape by mapping node-level log2-transformed P values and gene-length-adjusted RNA-seq read counts onto defined color gradients, so that dominant enzymes in multi-gene nodes are visually distinguishable.

## When to use

Use when FunHoP-expanded or aggregated KEGG network files (KGML-derived, node names like gene1-Bx) are loaded into Cytoscape via KEGGScape and must be colored by differential expression and read counts.

## When NOT to use

- KEGG glycan-related pathways (category 1.7) that use 'links' instead of nodes, which cannot be expanded or colored similarly in Cytoscape
- Networks whose nodes are not FunHoP-expanded (i.e. multi-gene homologs still collapsed by KEGGScape, which only shows the first listed gene)
- Read counts that have not been adjusted for gene length, since the count-based color mapping assumes length-corrected values

## Inputs

- FunHoP expanded network XML files (one child per gene in multi-gene nodes)
- FunHoP aggregated network files with gene1-Bx node names
- Node-level Voom differential expression results with log2-transformed P values (P < 0.05 cutoff)
- Gene-length-adjusted average RNA-seq read counts (sum-of-exons length / average gene length)
- Expression-count summary printed by funhop.py (for defining count color bins)

## Outputs

- Cytoscape network visualizations colored on the −1200-to-600 differential-expression gradient
- Cytoscape network visualizations colored on the 0-to-≥50,000 read-count scale (white to dark blue)
- Expanded networks with white-bordered functional-homolog (OR) nodes
- Aggregated networks (e.g. Figure 3D histidine, Figure 4 glycerophospholipid) suitable for export

## How to apply

Import the expanded or aggregated pathway networks into Cytoscape using the KEGGScape KGML reader, since KEGGScape alone only presents the first listed gene per reaction node. Apply two independent visual mappings: (1) differential expression, where P values from Voom (PCa vs normal, DEGs at P < 0.05) are log2-transformed as Value = log2(P) × (−10) × regulation and rendered on a −1200-to-600 gradient (black through −600 purple, −300 bright red, 0 light yellow, 300 bright green, to 600 dark green); and (2) read counts, rendered on a 0 to ≥50,000 scale (white through pink/blue shades to dark blue), using the expression-count summary printed by funhop.py to set the upper bin. Mark functional-homolog (OR) nodes in the expanded network with a white border, and verify aggregated node names follow the gene1-Bx convention before styling. The rationale is that read-count dominance (e.g. ALDH1A3 with 46,595 reads, PLA2G2A with 71,482 reads) identifies the enzyme most likely responsible for a reaction, which the differential-expression gradient alone cannot convey.

## Related tools

- **Cytoscape** (Visualization environment in which the expanded/aggregated networks are loaded and styled with the DE and read-count color gradients) — https://github.com/kjerstirise/FunHoP
- **KEGGScape** (KGML-reader app used to import KEGG pathway networks into Cytoscape (noting it shows only the first gene per node, hence the FunHoP expansion))
- **FunHoP** (Generates the expanded and aggregated network files and prints the expression-count summary used to define the count color scheme) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Differential-expression gradient spans exactly −1200 (black) to 600 (dark green) with intermediate anchors at −600 purple, −300 bright red, 0 light yellow, and 300 bright green
- Read-count scale upper bound matches the summary counts printed by funhop.py (e.g. ≥50,000, white to dark blue via pink/blue shades)
- Functional-homolog (OR) nodes in the expanded network carry a white border and aggregated node names follow the gene1-Bx convention
- Dominant enzymes are visually resolvable, e.g. ALDH1A3 at 46,595 reads dominating the ALDH3A1 node and PLA2G2A at 71,482 reads dominating the 21-gene PLA2G4B node
- Aggregated network reproduces reported node regulation, e.g. the non-significant original PLA2G4B node becomes up-regulated after aggregation

## Limitations

- KEGG glycan-related pathways (category 1.7) contain paths using 'links' instead of nodes, which cannot be expanded or colored in a similar manner and are excluded from FunHoP
- FunHoP is not yet available as a Cytoscape app, so styling must be applied manually per network
- calculate_counts.py contains hard-coded assumptions (gene names in the first row, tissue status in column 7, gene-length correction flag), which affect the values feeding the color mappings

## Evidence

- [abstract] Differential expression at node level is then applied to the networks.: "Differential expression at node level is then applied to the networks."
- [abstract] KEGGScape first-gene limitation: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction"
- [methods] Log2 P-value transform: "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [intro] Cytoscape as pathway visualization tool: "Cytoscape is a common tool for pathway visualization and analysis, often with data from KEGG"
- [readme] Count summary for color schemes: "ends up with a summary of the expression counts, which will be useful in creating the colour schemes for counts in Cytoscape"
- [readme] Glycan links cannot be colored: "These lines cannot be coloured in a similar matter in Cytoscape either, and are hence excluded from FunHoP."
