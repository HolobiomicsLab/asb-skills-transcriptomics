---
name: multi-gene-node-expression-aggregation
description: Use when KEGG KGML pathway nodes contain multiple functionally homologous
  genes but per-gene RNA-seq counts exist, and node-level expression values are needed
  for pathway visualization or differential expression in Cytoscape.
when_to_use_negative:
- Pathways in KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- Single-gene nodes with no homologs to collapse (aggregation is unnecessary; x =
  1 nodes need no read-count weighting)
- Non-human species or pathway databases (Reactome, Panther) outside the current FunHoP
  KEGG human-metabolism scope
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: FunHoP
  role: Implements node expansion, gene1-Bx renaming, and weighted read-count aggregation
    of multi-gene KEGG nodes
  repo: https://github.com/kjerstirise/FunHoP
- name: KEGGScape
  role: KGML reader used to load KEGG pathway files into Cytoscape; its limitation
    of showing only the first gene per node motivates the aggregation
- name: Voom
  role: Performs node-level differential expression on the aggregated node counts
    (PCa vs normal)
- name: Cytoscape
  role: Visualization of the aggregated networks with the node-level differential-expression
    color gradient
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:b5432c2de22c33490e5f69b01982e62dac3f03e388ffaef4f7df4680e6769c71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  - Voom
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-gene-node-expression-aggregation

## Summary

FunHoP-style aggregation that collapses all functionally homologous genes in a multi-gene KEGG network node into a single node-level RNA-seq expression value, weighting reads by gene length and the dominant gene's read ratio, so differential expression can be computed per node rather than per gene.

## When to use

Use when KEGG KGML pathway nodes contain multiple functionally homologous genes but per-gene RNA-seq counts exist, and node-level expression values are needed for pathway visualization or differential expression in Cytoscape.

## When NOT to use

- Pathways in KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- Single-gene nodes with no homologs to collapse (aggregation is unnecessary; x = 1 nodes need no read-count weighting)
- Non-human species or pathway databases (Reactome, Panther) outside the current FunHoP KEGG human-metabolism scope

## Inputs

- Expanded KEGG KGML network XML files (File S1) covering 85 human metabolic pathways (categories 1.1–1.11)
- Gene-length-adjusted average RNA-seq read counts (sum-of-exons length / average gene length) for TCGA PCa cancer and normal samples
- Sample metadata distinguishing cancer vs normal tissue status
- hsa-ID to gene-name conversion list (http://rest.kegg.jp/list/hsa)

## Outputs

- Aggregated network XML files with 'gene1-Bx' node names and one weighted expression value per node
- Node-level differential expression values: Value = log2(P value) × (−10) × regulation
- Cytoscape networks colored by the −1200 (black) to 600 (dark green) gradient

## How to apply

First expand the KEGG KGML network XML files so every gene in each node is shown, using the hsa-ID/gene-name conversion list, and rename each node 'gene1-Bx' where gene1 is the first gene name and x is the node's gene count. Use gene-length-adjusted read counts (each gene's count scaled by its sum-of-exons length divided by the average gene length over all genes) from cancer and normal TCGA PCa samples, and average the cancer/normal means into a single per-gene value. The aggregation assumes enzymatic activity mainly depends on the gene with the highest number of reads: sum all gene read counts within a node for the node total, then weight reads on gene length and each gene's ratio to the node total to compute one node expression value. Apply node-level differential expression with Voom comparing PCa versus normal tissue, extracting significant nodes at P < 0.05, and log2-transform P values as Value = log2(P value) × (−10) × regulation (regulation = 1 up, −1 down). Validate that node naming follows the gene1-Bx convention and that node-level regulation reproduces reported behavior, e.g. the 21-gene PLA2G4B node becoming up-regulated via dominant PLA2G2A (71,482 reads).

## Related tools

- **FunHoP** (Implements node expansion, gene1-Bx renaming, and weighted read-count aggregation of multi-gene KEGG nodes) — https://github.com/kjerstirise/FunHoP
- **KEGGScape** (KGML reader used to load KEGG pathway files into Cytoscape; its limitation of showing only the first gene per node motivates the aggregation)
- **Voom** (Performs node-level differential expression on the aggregated node counts (PCa vs normal))
- **Cytoscape** (Visualization of the aggregated networks with the node-level differential-expression color gradient)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- All aggregated node names follow the 'gene1-Bx' convention, where x equals the number of genes in the node (x = 1 for single-gene nodes)
- Node-level differential expression extracted at P < 0.05 and log2-transformed as Value = log2(P value) × (−10) × regulation
- The 21-gene PLA2G4B node, non-significant in the original network, becomes up-regulated via dominant PLA2G2A (71,482 reads)
- Read counts were length-adjusted before aggregation (gene sum-of-exons length / average gene length over all genes)
- Read count matrix has the same number of genes as the gene-name/expression table and matching sample counts vs metadata tissue statuses

## Limitations

- Enzyme kinetics data are usually not available, so the equal-enzyme-rate assumption behind read-count prioritization is a simplification
- KEGG does not provide curated enzyme-activity information for isoforms, so analysis is limited to the gene level
- Genes with particularly short or long dominant isoforms vs the canonical isoform model may yield aberrant expression estimates
- Glycan-related KEGG pathways (category 1.7) using 'links' cannot be expanded or colored and are excluded

## Evidence

- [abstract] Aggregation rationale (abstract): "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
- [methods] Gene-length adjustment (methods): "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] Log2 P-value transformation (methods): "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [abstract] Collapse goal (abstract): "FunHoP collapses all genes in a node into one measurement using read counts from RNA-seq."
- [readme] Pathway/link limitation (readme): "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded even if they contained multiple genes"
- [readme] Run command (readme): "./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>"
