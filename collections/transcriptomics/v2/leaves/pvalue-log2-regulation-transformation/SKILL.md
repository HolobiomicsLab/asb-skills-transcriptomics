---
name: pvalue-log2-regulation-transformation
description: Use when node-level Voom differential expression P values (PCa vs normal,
  cutoff P < 0.05) must be converted into signed values for coloring KEGG/FunHoP networks
  in Cytoscape.
when_to_use_negative:
- Data has not yet passed differential expression testing with Voom (no P values or
  regulation direction available to transform)
- Visualizing genes already stored as fold-change or log2FC values, which are already
  signed and need no P-value scaling
- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links'
  instead of nodes and cannot be colored in Cytoscape
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Voom
  role: Performs the node-level differential expression analysis that produces the
    P values being transformed
- name: Cytoscape
  role: Renders the transformed values as a network color gradient (−1200 black to
    600 dark green)
- name: FunHoP
  role: Pipeline code that runs the transformation and outputs expression-count summaries
    used to build Cytoscape color schemes
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:cf14a9eea5ba265581325591dde1853fdcb7d695674e0b2740fc1315b3770f85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  - Voom
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pvalue-log2-regulation-transformation

## Summary

Transforms node- or gene-level differential expression P values from Voom into a signed log2-scaled value, Value = log2(P value) × (−10) × regulation, so that statistical significance and up/down direction can be rendered as a single numeric gradient in Cytoscape network coloring.

## When to use

Use when node-level Voom differential expression P values (PCa vs normal, cutoff P < 0.05) must be converted into signed values for coloring KEGG/FunHoP networks in Cytoscape.

## When NOT to use

- Data has not yet passed differential expression testing with Voom (no P values or regulation direction available to transform)
- Visualizing genes already stored as fold-change or log2FC values, which are already signed and need no P-value scaling
- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links' instead of nodes and cannot be colored in Cytoscape

## Inputs

- Node-level (or gene-level) differential expression P values from Voom (PCa vs normal tissue comparison)
- Regulation direction (up/down) for each significant node at P < 0.05
- Aggregated network XML files with gene1-Bx node names

## Outputs

- Signed log2-transformed node values: Value = log2(P value) × (−10) × regulation
- Cytoscape node color mappings on the −1200-to-600 gradient for expanded and aggregated KEGG networks

## How to apply

After running node-level differential expression with Voom comparing PCa versus normal tissue, extract significant nodes at P < 0.05. For each significant node compute Value = log2(P value) × (−10) × regulation, where regulation = 1 for up-regulated and −1 for down-regulated nodes; this magnifies small P values into a usable numeric range. Apply the same transformation in the aggregated gene1-Bx networks so color gradients are comparable between expanded and aggregated versions. Map the resulting values onto the Cytoscape color gradient spanning −1200 (black) to 600 (dark green) via −600 (purple), −300 (bright red), 0 (light yellow), and 300 (bright green). The FunHoP script emits a summary of expression counts that guides building these color schemes.

## Related tools

- **Voom** (Performs the node-level differential expression analysis that produces the P values being transformed)
- **Cytoscape** (Renders the transformed values as a network color gradient (−1200 black to 600 dark green))
- **FunHoP** (Pipeline code that runs the transformation and outputs expression-count summaries used to build Cytoscape color schemes) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Only nodes with P < 0.05 receive a transformed value; non-significant nodes are excluded
- Up-regulated nodes yield positive Values (regulation = 1) and down-regulated nodes negative Values (regulation = −1)
- Transformed values fall within the coloring range used by the Cytoscape gradient (roughly −1200 to 600)
- Node names in the colored network follow the gene1-Bx convention and node-level regulation reproduces reported behavior (e.g., the 21-gene PLA2G4B node becomes up-regulated via dominant PLA2G2A with 71,482 reads)

## Limitations

- Interpretation depends on the upstream assumption that enzymatic activity mainly depends on the gene with the highest number of RNA-seq reads
- Genes with particularly short or long dominant isoforms versus the canonical isoform model may lead to aberrant expression level estimates
- Read counts must be gene-length corrected; if not, the 'correct' flag must be set to True in load_countdata

## Evidence

- [methods] P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation: "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [methods] DEGs with a P value below 0.05 were extracted: "DEGs with a P value below 0.05 were extracted"
- [readme] summary of expression counts for Cytoscape color schemes: "ends up with a summary of the expression counts, which will be useful in creating the colour schemes for counts in Cytoscape"
- [full_text] node-level regulation validation via PLA2G4B node: [evidence span withheld — anchor status: not_found]
