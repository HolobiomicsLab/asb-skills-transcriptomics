---
name: prostate-cancer-pathway-network-interpretation
description: Use when analyzing differential RNA-seq expression on KEGG metabolic
  pathways in Cytoscape, where nodes contain multiple functionally homologous genes
  and standard KGML readers show only the first listed gene, masking the true catalytic
  driver.
when_to_use_negative:
- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- When only isoform-level enzyme activity matters — KEGG provides no curated isoform
  activity data, so analysis is limited to the gene level
- When per-sample pathway statistics from generic enrichment tools suffice and no
  multi-gene homologous nodes need disambiguation
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: FunHoP
  role: Expands KEGG nodes to show all homologous genes and collapses their RNA-seq
    read counts into one node-level expression value for differential analysis
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization platform into which expanded/aggregated pathway networks are
    loaded for coloring and interpretation
- name: KEGGScape
  role: KGML-reader Cytoscape app used to import KEGG pathway files; its limitation
    of showing only the first listed gene motivates FunHoP expansion
- name: TopHat2
  role: Maps raw prostate cancer RNA-seq reads to the hg19 transcriptome before gene-level
    counting
- name: featureCounts
  role: Assigns mapped reads to each gene to build the expression count matrix
- name: Voom
  role: Performs differential expression analysis on node-level/gene-level counts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:ab100c894ad3b4c81898536c8bf1bfec423035367da47115657f54e97bd5bf34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  - TopHat2
  - Voom
  - featureCounts
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# prostate-cancer-pathway-network-interpretation

## Summary

Interprets KEGG metabolic pathway networks for prostate cancer RNA-seq data using FunHoP, which expands multi-gene nodes to show all functionally homologous proteins and collapses their read counts into one node-level expression value, enabling identification of the dominant enzyme driving each reaction.

## When to use

Use when analyzing differential RNA-seq expression on KEGG metabolic pathways in Cytoscape, where nodes contain multiple functionally homologous genes and standard KGML readers show only the first listed gene, masking the true catalytic driver.

## When NOT to use

- Pathways from KEGG category 1.7 'glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- When only isoform-level enzyme activity matters — KEGG provides no curated isoform activity data, so analysis is limited to the gene level
- When per-sample pathway statistics from generic enrichment tools suffice and no multi-gene homologous nodes need disambiguation

## Inputs

- KEGG KGML pathway XML files (human metabolic pathways, categories 1.1–1.11)
- hsa ID-to-gene-name conversion list (from http://rest.kegg.jp/list/hsa)
- Gene expression table (tab-separated; gene names, gene IDs, probe values; gene names in first row)
- Sample metadata (tab-separated; tissue status in column 6/7 distinguishing cancer vs control)
- Expression count matrix (tab-separated; read counts for all samples, same genes as expression table)
- Updated gene symbols list
- RNA-seq read-count tables from TCGA and Prensner (dbGap phs000443.v1.p1) prostate cancer cohorts

## Outputs

- Expanded and aggregated KEGG pathway networks with all homologous genes shown per node
- Node-level collapsed expression values and regulation summaries for Cytoscape color schemes
- Dominant-gene assignments per multi-gene node (e.g., PLA2G2A in the 21-gene PLA2G4B node of hsa00564)
- Cross-cohort correlation coefficients and abundance-ratio concordance statistics (cohort_dominant_gene_comparison.csv)

## How to apply

Download the ~85 human metabolic KEGG pathways (KGML XML, categories 1.1–1.11, excluding glycan pathways) plus the hsa ID-to-gene-name conversion list from http://rest.kegg.jp/list/hsa, and expand each node to show all homologous genes. Normalize read counts for gene length by the factor (sum of exons of the gene) / (average gene length over all genes), assuming the reaction activity mainly depends on the gene with the highest number of reads. Collapse all genes in a node into a single measurement and apply differential expression at node level, extracting DEGs with P < 0.05 and log2-transforming P values as Value = log2(Pvalue) × (−10) × regulation. Validate reproducibility by comparing dominant genes and very high (> 0.9) or very low (< 0.1) abundance ratios across the TCGA and Prensner prostate cancer cohorts with Pearson correlation, as in Figures 2C–D. Load the expanded networks into Cytoscape with KEGGScape to build color schemes from the summarized node counts and interpret metabolic changes (e.g., elevated histidine and GPC in PCa).

## Related tools

- **FunHoP** (Expands KEGG nodes to show all homologous genes and collapses their RNA-seq read counts into one node-level expression value for differential analysis) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization platform into which expanded/aggregated pathway networks are loaded for coloring and interpretation)
- **KEGGScape** (KGML-reader Cytoscape app used to import KEGG pathway files; its limitation of showing only the first listed gene motivates FunHoP expansion)
- **TopHat2** (Maps raw prostate cancer RNA-seq reads to the hg19 transcriptome before gene-level counting)
- **featureCounts** (Assigns mapped reads to each gene to build the expression count matrix)
- **Voom** (Performs differential expression analysis on node-level/gene-level counts)

## Examples

```
./funhop.py hsa_list.tsv expression_table.tsv metadata.tsv counts.tsv updated_symbols.tsv
```

## Evaluation signals

- Dominant genes in multi-gene nodes (e.g., PLA2G2A in the 21-gene PLA2G4B node of hsa00564; CNDP2/ALDH1A3 in histidine metabolism) correspond between TCGA and Prensner cohorts
- Genes flagged with very high (> 0.9) or very low (< 0.1) per-gene abundance ratios match across cohorts, reproducing Figures 2C–D and S1–S2
- Pearson correlation of log2 read counts and abundance ratios between cohorts is high
- Count matrix gene count equals the number of gene names in the expression table, and sample count equals the number of tissue statuses
- FunHoP prints completed steps per pathway to screen and ends with a summary of expression counts usable for Cytoscape color schemes

## Limitations

- Fewer changes reach P < 0.05 significance in the Prensner cohort due to smaller sample size and lower sequencing depth
- Glycan-related KEGG pathways (category 1.7) are excluded because 'links'-based paths cannot be expanded or colored in Cytoscape
- Equal enzyme rate is assumed since enzyme kinetics/efficiency data are usually unavailable — a simplification
- Genes with unusually short or long dominant isoforms versus the canonical isoform model may get aberrant expression estimates
- Calculate_counts.py contains hard coding: gene names expected in first row, tissue-type column fixed (column 6/7), and gene-length correction toggled via 'correct=True'

## Evidence

- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways.: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways."
- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2: "Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2"
- [readme] ./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>: "./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>"
- [readme] the pathways from KEGG category 1.7 'glycol biosynthesis and metabolism' contains some paths that use 'links' instead of nodes, and these cannot be expanded even if they contained multiple genes: "the pathways from KEGG category 1.7 'glycol biosynthesis and metabolism' contains some paths that use 'links' instead of nodes, and these cannot be expanded even if they contained multiple genes"
