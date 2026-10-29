---
name: node-level-read-count-aggregation
description: Use when KEGG KGML pathway nodes contain multiple functionally homologous
  genes catalyzing the same reaction and you need a single node-level RNA-seq read-count
  value plus identification of the dominant (most-expressed) gene per node.
when_to_use_negative:
- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored in Cytoscape and are excluded
  from FunHoP
- Count data already corrected for gene length — set 'correct' to False rather than
  re-correcting in load_countdata
- Pathway databases other than human KEGG metabolic pathways — the current FunHoP
  version supports only KEGG human metabolic pathways (excluding glycan-related ones)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: FunHoP
  role: Implements node expansion, read-count aggregation, gene-length correction,
    and dominant-gene prioritization for KEGG pathway nodes
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization platform into which the expanded, node-aggregated pathways are
    loaded and colored by expression counts
- name: KEGGScape
  role: KGML reader used to load KEGG pathway files into Cytoscape (baseline behavior
    that FunHoP improves upon by showing all genes per node)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:f19ec164e5ac36e8b730216dab5e361576658fd2f9b776701e34d50c2fdf653f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# node-level-read-count-aggregation

## Summary

Aggregates RNA-seq read counts across all functionally homologous genes in a multi-gene KEGG pathway node into a single node-level expression value, weighting reads by gene length and per-gene abundance ratio, as implemented in FunHoP for prostate cancer RNA-seq data analyzed in Cytoscape.

## When to use

Use when KEGG KGML pathway nodes contain multiple functionally homologous genes catalyzing the same reaction and you need a single node-level RNA-seq read-count value plus identification of the dominant (most-expressed) gene per node.

## When NOT to use

- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored in Cytoscape and are excluded from FunHoP
- Count data already corrected for gene length — set 'correct' to False rather than re-correcting in load_countdata
- Pathway databases other than human KEGG metabolic pathways — the current FunHoP version supports only KEGG human metabolic pathways (excluding glycan-related ones)

## Inputs

- KEGG KGML pathway XML files (human metabolic pathways, categories 1.1–1.11)
- hsa ID-to-gene-name conversion list (e.g. from http://rest.kegg.jp/list/hsa)
- Gene expression table with gene names, gene IDs, and probe values (tab-separated)
- Sample metadata indicating cancer vs. control tissue status (tab-separated)
- RNA-seq read count matrix (genes × samples, tab-separated, same gene order as expression table)
- Updated gene symbols list

## Outputs

- Expanded KEGG pathway files with all genes shown per node
- Node-level aggregated read-count summary for constructing Cytoscape color schemes
- Per-node dominant-gene assignments and overall node regulation values
- Node-level differential expression values (log2-transformed P values with regulation sign)

## How to apply

Load KEGG KGML pathway files into Cytoscape and expand each node to show all homologous genes using the hsa-ID conversion list from http://rest.kegg.jp/list/hsa. Adjust raw read counts for gene length by the factor (gene exon-sum length divided by the average gene length over all genes), assuming activity of a reaction mainly depends on the gene with the highest read count. For each multi-gene node, sum all gene reads to compute the node-level count and compute per-gene relative abundance as the fraction of each gene's reads over the total node read count, producing a new expression value showing overall node regulation. Extract DEGs with P < 0.05, log2 transform P values as Value = log2(Pvalue × (-10)) × regulation, and apply differential expression at node level. Optionally validate cross-cohort consistency by correlating log2 counts and abundance ratios (e.g. Pearson) between cohorts and checking that dominant genes (e.g. PLA2G2A in the 21-gene PLA2G4B node of hsa00564) correspond.

## Related tools

- **FunHoP** (Implements node expansion, read-count aggregation, gene-length correction, and dominant-gene prioritization for KEGG pathway nodes) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization platform into which the expanded, node-aggregated pathways are loaded and colored by expression counts)
- **KEGGScape** (KGML reader used to load KEGG pathway files into Cytoscape (baseline behavior that FunHoP improves upon by showing all genes per node))

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Dominant genes and very-high (> 0.9) / very-low (< 0.1) abundance-ratio genes correspond between TCGA and Prensner cohorts, as reported for Figures 2C-D and S1-S2
- The expression-count matrix in the summary output has the same number of genes as the gene names in the expression table and the same number of samples as tissue statuses in the metadata
- Every multi-gene node in the expanded pathway contains all homologous genes, not just the first listed gene as in KEGGScape
- Read counts are gene-length-adjusted (exon-sum length / average gene length over all genes) before aggregation when the input data are not already corrected
- Fewer Prensner changes reach P < 0.05 than TCGA, consistent with its smaller sample size and lower sequencing depth — a sanity check on the DE threshold behavior

## Limitations

- Equal enzyme rate is assumed since enzyme kinetics/efficiency data are usually not available, which is a simplification
- KEGG provides no curated information on enzyme activity of isoforms, so analysis is limited to the gene level
- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation
- Calculate_counts.py contains hard coding: gene names expected in the first row, tissue-status column expected to be column 7, and a 'correct' flag for gene-length adjustment
- No version numbers are given for FunHoP, Cytoscape, or KEGGScape, and FunHoP is not yet available as a Cytoscape app (conversion in development)

## Evidence

- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] To make more interpretable networks yet containing all the information, all genes within a node were aggregated into one: "To make more interpretable networks yet containing all the information, all genes within a node were aggregated into one"
- [readme] It uses read counts from RNA-sequencing to find the homolog in a node which is (probably) responsible for the enzymatic activity in a node.: "It uses read counts from RNA-sequencing to find the homolog in a node which is (probably) responsible for the enzymatic activity in a node."
- [readme] the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded: "the pathways from KEGG category 1.7 ‘glycol biosynthesis and metabolism’ contains some paths that use ‘links’ instead of nodes, and these cannot be expanded"
