---
name: dominant-gene-ratio-comparison
description: Use when FunHoP node-level pathway analysis has been run on RNA-seq data
  from two independent patient cohorts (e.g. TCGA and Prensner prostate cancer) and
  you need to verify that dominant genes and very-high/very-low abundance-ratio genes
  in multi-gene KEGG nodes reproduce across cohorts.
when_to_use_negative:
- You only have single-end-node KEGG networks from KEGGScape without FunHoP expansion,
  so no multi-gene nodes or within-node abundance ratios exist to compare.
- The two RNA-seq datasets are not from comparable conditions or tissues (the concordance
  logic assumes the same cancer-vs-normal contrast in both cohorts).
- Input data have not been read-count-normalized for gene length; running ratio comparison
  on raw counts can mis-assign the dominant gene.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FunHoP
  role: Expands KEGG KGML pathways to show all homologous genes per node, collapses
    node reads, and produces the node-level read counts and dominant-gene assignments
    compared across cohorts
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization of the expanded/aggregated KEGG networks in which dominant-gene
    assignments and abundance-based colour schemes are inspected
- name: KEGGScape
  role: KGML reader used to load KEGG pathway files into Cytoscape for the pathway
    networks being compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:3503e835c2bc1c413ff0f6d921693ce969403ee4a2f878934eea6b05e600d0bf
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# dominant-gene-ratio-comparison

## Summary

Cross-cohort reproducibility check for FunHoP pathway analysis: identify the dominant (highest-read-count) gene in each multi-gene KEGG node for two independent RNA-seq cohorts (TCGA and Prensner), correlate log2 read counts and per-gene abundance ratios with Pearson correlation, and flag very-high (> 0.9) or very-low (< 0.1) abundance-ratio genes for concordance.

## When to use

Use when FunHoP node-level pathway analysis has been run on RNA-seq data from two independent patient cohorts (e.g. TCGA and Prensner prostate cancer) and you need to verify that dominant genes and very-high/very-low abundance-ratio genes in multi-gene KEGG nodes reproduce across cohorts.

## When NOT to use

- You only have single-end-node KEGG networks from KEGGScape without FunHoP expansion, so no multi-gene nodes or within-node abundance ratios exist to compare.
- The two RNA-seq datasets are not from comparable conditions or tissues (the concordance logic assumes the same cancer-vs-normal contrast in both cohorts).
- Input data have not been read-count-normalized for gene length; running ratio comparison on raw counts can mis-assign the dominant gene.

## Inputs

- Gene-level read-count tables for each cohort (e.g. TCGA and Prensner RNA-seq, tab-separated matrix of all genes × samples)
- Expanded/aggregated KEGG node-gene mapping (multi-gene nodes from FunHoP-expanded KGML pathways, e.g. 21-gene PLA2G4B node in hsa00564)
- Gene length table (exon-sum lengths) for FunHoP gene-length correction
- Per-cohort sample metadata / tissue status for differential-expression context

## Outputs

- cohort_dominant_gene_comparison.csv with per-node dominant-gene assignments for each cohort
- Pearson correlation coefficients of log2 read counts between cohorts per gene/node
- Pearson correlation coefficients of per-gene abundance ratios between cohorts
- Very-high (> 0.9) and very-low (< 0.1) abundance-ratio gene flags and cross-cohort concordance statistics

## How to apply

Load the TCGA and Prensner gene-level read-count tables and adjust counts for gene length by the factor computed as the gene's exon-sum length divided by the average gene length over all genes, as FunHoP does. For each multi-gene node in the expanded/aggregated networks, compute per-gene relative abundance as the fraction of each gene's read count over the total node read count, and identify the dominant gene (the gene with the highest read count, since FunHoP assumes enzymatic activity mainly depends on the gene with the highest number of reads). Correlate log2 read counts and per-gene abundance ratios between the two cohorts using Pearson correlation, following the paper's validation approach in Figures 2C-D. Flag genes with abundance ratios > 0.9 (very high) or < 0.1 (very low) and check that these dominant/minor genes correspond between cohorts. Write per-node dominant-gene assignments, correlation coefficients, and ratio-concordance statistics to a comparison table such as cohort_dominant_gene_comparison.csv. Expect fewer changes to reach P < 0.05 significance in the Prensner cohort due to its smaller sample size and lower sequencing depth.

## Related tools

- **FunHoP** (Expands KEGG KGML pathways to show all homologous genes per node, collapses node reads, and produces the node-level read counts and dominant-gene assignments compared across cohorts) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization of the expanded/aggregated KEGG networks in which dominant-gene assignments and abundance-based colour schemes are inspected)
- **KEGGScape** (KGML reader used to load KEGG pathway files into Cytoscape for the pathway networks being compared)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Dominant genes per multi-gene node correspond between TCGA and Prensner (e.g. PLA2G2A dominant in the 21-gene PLA2G4B node of hsa00564 in both cohorts)
- Genes flagged with abundance ratio > 0.9 or < 0.1 in one cohort are flagged the same way in the other cohort
- Pearson correlation of log2 read counts and of per-gene abundance ratios between cohorts is high, consistent with Figures 2C-D and S1-S2
- Per-gene abundance fractions sum to 1 within each multi-gene node in both cohorts
- Read counts are gene-length-adjusted (exon-sum length / average gene length) before ratios are computed; differences reaching P < 0.05 are fewer in Prensner as expected from its smaller sample size and lower sequencing depth

## Limitations

- FunHoP assumes enzymatic activity mainly depends on the gene with the highest read count; enzyme kinetics data are usually unavailable, so equal enzyme rates are a simplification.
- KEGG does not provide curated isoform-level enzyme activity, so analysis is limited to the gene level; genes with atypical dominant isoforms may get aberrant expression estimates.
- Glycan-related KEGG pathways (category 1.7) use 'links' instead of nodes and cannot be expanded, so they are excluded from any node-level comparison.
- Fewer changes reach P < 0.05 significance in the Prensner cohort due to its smaller sample size and lower sequencing depth, which affects cross-cohort significance concordance.

## Evidence

- [abstract] Here we show that FunHoP gives more consistent pathways that are easier to interpret biologically.: "Here we show that FunHoP gives more consistent pathways that are easier to interpret biologically."
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads"
- [discussion] does not have any of the features of FunHoP with regard to using reads from RNA-seq to determine an overall expression value for all genes in a multi-gene node.: "does not have any of the features of FunHoP with regard to using reads from RNA-seq to determine an overall expression value for all genes in a multi-gene node."
- [readme] It uses read counts from RNA-sequencing to find the homolog in a node which is (probably) responsible for the enzymatic activity in a node.: "It uses read counts from RNA-sequencing to find the homolog in a node which is (probably) responsible for the enzymatic activity in a node."
