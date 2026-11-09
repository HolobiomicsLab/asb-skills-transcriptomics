---
name: differential-expression-log-transform-visualization
description: Use when signed differential-expression scores for a PCa-vs-normal RNA-seq
  contrast must be rendered as a Cytoscape color gradient (−1200 to 600) over expanded
  KEGG pathway nodes to show up/down regulation per gene or aggregated node.
when_to_use_negative:
- The KEGG pathway uses 'links' instead of nodes (glycan biosynthesis and metabolism,
  category 1.7) — these lines cannot be expanded or colored in Cytoscape
- No two-group contrast exists (e.g., metadata lacks cancer/normal tissue status),
  so Voom cannot assign the regulation sign needed for the transform
- The visualization target is absolute expression level (read counts on the 0 to ≥50,000
  scale) rather than regulation direction — style by counts instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0203
tools:
- name: Voom
  role: Performs the PCa-vs-normal differential expression whose P values and regulation
    signs feed the log2 transform
- name: Cytoscape
  role: Renders the expanded pathway and applies the −1200-to-600 continuous node-color
    gradient from the transformed values
- name: KEGGScape
  role: Loads the KEGG KGML pathway XML into Cytoscape so the expanded nodes can be
    styled by DE value
- name: FunHoP
  role: Expands multi-gene KEGG nodes so each homolog and aggregated node ('gene1-Bx')
    receives its transformed DE value for coloring
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:6961a6acd8554decc25f292ba94b2219ab44b1b7bb8dbb4054fd13c11b317518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - ElementTree
  - FunHoP
  - KEGGScape
  - Voom
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# differential-expression-log-transform-visualization

## Summary

Runs Voom differential expression on RNA-seq read counts (PCa vs normal), extracts DEGs at P < 0.05, log2-transforms P values as Value = log2(P)×(−10)×regulation, and uses the signed scores to color FunHoP-expanded KEGG pathway nodes in Cytoscape on a −1200-to-600 gradient.

## When to use

Use when signed differential-expression scores for a PCa-vs-normal RNA-seq contrast must be rendered as a Cytoscape color gradient (−1200 to 600) over expanded KEGG pathway nodes to show up/down regulation per gene or aggregated node.

## When NOT to use

- The KEGG pathway uses 'links' instead of nodes (glycan biosynthesis and metabolism, category 1.7) — these lines cannot be expanded or colored in Cytoscape
- No two-group contrast exists (e.g., metadata lacks cancer/normal tissue status), so Voom cannot assign the regulation sign needed for the transform
- The visualization target is absolute expression level (read counts on the 0 to ≥50,000 scale) rather than regulation direction — style by counts instead

## Inputs

- RNA-seq read-count matrix with sample metadata labeling cancer vs normal tissue (e.g., TCGA PCa featureCounts-assigned counts)
- FunHoP-expanded KEGG pathway network (KGML XML loaded in Cytoscape via KEGGScape)
- Voom differential expression results: P values and up/down regulation direction per gene

## Outputs

- Signed log2-transformed DE scores per gene/node (Value = log2(P)×(−10)×regulation)
- Filtered DEG list at P < 0.05
- Node-colored Cytoscape pathway network styled on a −1200-to-600 continuous gradient

## How to apply

Run Voom differential expression on the RNA-seq read-count matrix contrasting PCa against normal tissue and extract DEGs with P < 0.05. Transform each P value to Value = log2(P)×(−10)×regulation, where regulation = 1 for up-regulated and −1 for down-regulated, placing significance magnitude and direction on one signed scale. Attach these values to the FunHoP-expanded KEGG KGML network (loaded into Cytoscape via KEGGScape) and apply a continuous node-color mapping spanning −1200 to 600 so all expanded homologs are styled consistently. For node-level interpretation, aggregate all genes within a node ('gene1-Bx' naming), rerun Voom on the node values, and apply the same transform and gradient. The rationale is that raw P values compress near zero, whereas the log2 ×(−10) scaling spreads small P values across the gradient while the regulation sign keeps up- and down-regulation visually separable.

## Related tools

- **Voom** (Performs the PCa-vs-normal differential expression whose P values and regulation signs feed the log2 transform)
- **Cytoscape** (Renders the expanded pathway and applies the −1200-to-600 continuous node-color gradient from the transformed values)
- **KEGGScape** (Loads the KEGG KGML pathway XML into Cytoscape so the expanded nodes can be styled by DE value)
- **FunHoP** (Expands multi-gene KEGG nodes so each homolog and aggregated node ('gene1-Bx') receives its transformed DE value for coloring) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Every colored feature passed the Voom DEG cutoff P < 0.05 before transformation.
- Sign check: regulation = 1 yields a positive Value and regulation = −1 a negative Value (e.g., up-regulated CNDP2 with ~11,000 reads scores positive while down-regulated CNDP1 with near-zero reads scores negative in the histidine network).
- Arithmetic spot-check: a P value of 0.05 with regulation = 1 maps to ≈ +43.2 (log2(0.05) ≈ −4.32, ×(−10)).
- The Cytoscape continuous mapping spans exactly −1200 to 600 and no expanded node is left unstyled.
- After node aggregation ('gene1-Bx'), the node-level rerun of Voom yields one regulation sign per node consistent with the dominant homolog in the verification table (e.g., ALDH1A3 with 46,595 reads dominating its node).

## Limitations

- KEGG glycan-related pathways (category 1.7) that use 'links' instead of nodes cannot be expanded or colored in Cytoscape and are excluded from FunHoP.
- Reaction activity is assumed to depend mainly on the gene with the highest read count; enzyme kinetics/efficiency data are usually unavailable, so this is a simplification.
- Calculate_counts.py hard-codes gene names in the first row and tissue status in column 7, and the gene-length 'correct' flag must be set properly before Voom grouping works.
- Analysis is limited to the gene level because KEGG provides no curated isoform-level enzyme activity, so genes with atypical dominant isoforms may receive aberrant expression estimates.

## Evidence

- [methods] Voom used for differential expression: "Voom [26] was further used for differential expression analysis"
- [methods] DEG extraction threshold: "DEGs with a P value below 0.05 were extracted"
- [methods] Log2 transform formula for P values: "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [abstract] Node-level differential expression applied to networks: "Differential expression at node level is then applied to the networks."
- [readme] Cytoscape coloring limitation for link-based pathways: "These lines cannot be coloured in a similar matter in Cytoscape either, and are hence excluded from FunHoP."
- [readme] FunHoP output supports Cytoscape color schemes: "FunHoP prints to screen the completed steps on the pathways, and ends up with a summary of the expression counts, which will be useful in creating the colour schemes for counts in Cytoscape."
