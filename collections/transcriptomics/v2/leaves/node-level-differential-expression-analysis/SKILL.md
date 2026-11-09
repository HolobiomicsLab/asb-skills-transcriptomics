---
name: node-level-differential-expression-analysis
description: Use when RNA-seq read-count data must be analyzed per KEGG pathway node
  containing multiple functionally homologous genes, to identify which homolog dominates
  enzymatic activity and the node's overall up/down regulation (e.g., the 21-gene
  PLA2G4B node in hsa00564).
when_to_use_negative:
- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- Gene-level analysis where each node already contains a single gene and no homolog
  collapsing is needed
- Non-KEGG pathway sources lacking KGML node definitions or multi-gene homolog groups
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FunHoP
  role: Expands multi-gene KEGG nodes, aggregates read counts into node-level expression
    values, and produces network files for node-level DE analysis
  repo: https://github.com/kjerstirise/FunHoP
- name: Voom
  role: Performs the differential expression analysis on aggregated node counts comparing
    PCa versus normal tissue
- name: Cytoscape
  role: Visualization of node-level DE results using the −1200 to 600 color gradient
- name: KEGGScape
  role: KGML-reader app used to load KEGG pathway files into Cytoscape
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:2e202bb50ed4c0044deadc48a1e36982b4f5fa56728619881d0e5a657cbb7b18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - ElementTree XML API
  - FunHoP
  - KEGGScape
  - Voom
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# node-level-differential-expression-analysis

## Summary

FunHoP's approach of running differential expression on aggregated KEGG network nodes rather than individual genes: multi-gene nodes are collapsed using RNA-seq read counts (weighted by gene length and ratio), then Voom is applied at the node level with P < 0.05 cutoff and a log2(P)-transformed value scale.

## When to use

Use when RNA-seq read-count data must be analyzed per KEGG pathway node containing multiple functionally homologous genes, to identify which homolog dominates enzymatic activity and the node's overall up/down regulation (e.g., the 21-gene PLA2G4B node in hsa00564).

## When NOT to use

- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- Gene-level analysis where each node already contains a single gene and no homolog collapsing is needed
- Non-KEGG pathway sources lacking KGML node definitions or multi-gene homolog groups

## Inputs

- KEGG KGML pathway XML files (e.g., hsa00564 glycerophospholipid metabolism; 85 human metabolic pathways, categories 1.1–1.11)
- Gene-length-adjusted average RNA-seq read counts per gene (TCGA PCa cancer and normal samples; also Prensner cohort)
- Sample metadata distinguishing tissue status (cancer vs control; column 6 in test files)
- hsa-ID to gene-name conversion list (http://rest.kegg.jp/list/hsa)
- Gene expression table with gene names in column 0
- Updated gene symbols list

## Outputs

- Aggregated KEGG network files with nodes renamed 'gene1-Bx' (gene1 = first gene in node string, x = number of genes)
- Node-level differential expression results (DE nodes at P < 0.05) with Value = log2(P) × (−10) × regulation
- Node total read counts and per-gene read-count fractions for Cytoscape color scales
- Cytoscape networks styled with the −1200 to 600 DE gradient and 0 to ≥50,000 white-to-dark-blue count scale

## How to apply

First aggregate each multi-gene node: sum read counts of all genes in the node, compute each gene's fraction of the node total, and assume enzymatic activity depends mainly on the highest-read gene, weighting reads by gene length (sum of exons divided by average gene length over all genes). Run Voom differential expression on the aggregated node counts comparing the two tissue groups defined in the metadata (e.g., TCGA prostate cancer vs normal), extracting DE nodes at P < 0.05. Transform each node's significance to Value = log2(P value) × (−10) × regulation, with regulation = 1 for up- and −1 for down-regulated nodes. Visualize and validate in Cytoscape via KEGGScape using the −1200 black to 600 dark green gradient (via −600 purple, −300 bright red, 0 light yellow, 300 bright green), coloring read counts white to dark blue on a 0 to ≥50,000 scale. The rationale is that node-level analysis reveals regulation of reactions catalyzed by homologs that gene-level analysis obscures — e.g., the PLA2G4B node, non-significant per-gene, becomes up-regulated through dominant PLA2G2A.

## Related tools

- **FunHoP** (Expands multi-gene KEGG nodes, aggregates read counts into node-level expression values, and produces network files for node-level DE analysis) — https://github.com/kjerstirise/FunHoP
- **Voom** (Performs the differential expression analysis on aggregated node counts comparing PCa versus normal tissue)
- **Cytoscape** (Visualization of node-level DE results using the −1200 to 600 color gradient)
- **KEGGScape** (KGML-reader app used to load KEGG pathway files into Cytoscape)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Aggregated node names follow the 'gene1-Bx' convention (e.g., PLA2G4B node = 'PLA2G4B-B21' for 21 genes)
- The 21-gene PLA2G4B node is non-significant in the original network but becomes up-regulated in the aggregated network via dominant PLA2G2A (71,482 reads)
- Figure 4 values reproduce exactly: PLA2G2A 71,482 reads, PLA2G12A 4,502 reads, PLA2G16 3,246 reads
- DE nodes are extracted at P < 0.05 and values follow Value = log2(P) × (−10) × regulation with regulation = ±1
- FunHoP ends with a printed summary of expression counts usable for building Cytoscape count color schemes

## Limitations

- Assumes enzymatic activity depends mainly on the highest-read gene; enzyme kinetics/efficiency data are usually unavailable, making the equal-rate assumption a simplification
- Genes with particularly short or long dominant isoforms versus the canonical isoform model may yield aberrant expression estimates
- Only supports human metabolic KEGG pathways except glycan-related ones; KEGG provides no curated isoform-level enzyme activity data, so analysis is limited to gene level
- Calculate_counts.py contains hard-coding: gene names expected in first row, tissue status in column 7, and the 'correct' gene-length flag set manually

## Evidence

- [abstract] Differential expression at node level is then applied to the networks.: "Differential expression at node level is then applied to the networks."
- [methods] DEGs with a P value below 0.05 were extracted; Value=log2Pvalue×(-10)×regulation: "DEGs with a P value below 0.05 were extracted"
- [methods] P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation: "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [readme] README run command for funhop.py: "./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>"
- [readme] README hard-coding warning on gene-length correction flag: "If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function"
