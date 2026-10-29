---
name: pathway-flux-interpretation-from-dominant-genes
description: Use when a KEGG metabolic pathway node contains multiple functionally
  homologous genes (e.g. the four-gene ALDH3A1 node in hsa00340) and read-count data
  must be aggregated per node to identify the dominant enzyme.
when_to_use_negative:
- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links'
  instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- When only a single gene maps to each reaction node — node expansion and count-based
  prioritization add nothing
- When enzyme-level (isoform) activity resolution is required — KEGG does not provide
  curated isoform-level enzyme activity, so analysis is limited to the gene level
edam_operation: http://edamontology.org/operation_2497
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: FunHoP
  role: Expands multi-gene KGML nodes into per-gene children and collapses node reads
    into one aggregated expression value
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Platform for visualizing the expanded and aggregated networks with differential-expression
    and read-count color gradients
- name: KEGGScape
  role: KGML reader used to import KEGG pathway files into Cytoscape; FunHoP corrects
    its first-gene-only node limitation
- name: Voom
  role: Differential expression analysis on genes and on aggregated nodes, with DEGs
    extracted at P < 0.05
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:d6365d389a1427ebc62af5e52b4e9492b782f9663c2acc4e9357c739e41879ea
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
  edam_operation: http://edamontology.org/operation_2497
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# pathway-flux-interpretation-from-dominant-genes

## Summary

Prioritize the enzyme most likely responsible for a reaction in multi-gene KEGG pathway nodes by assuming reaction activity mainly depends on the gene with the highest RNA-seq read count, then collapse all homologous genes in the node into a single aggregated measurement for differential-expression interpretation.

## When to use

Use when a KEGG metabolic pathway node contains multiple functionally homologous genes (e.g. the four-gene ALDH3A1 node in hsa00340) and read-count data must be aggregated per node to identify the dominant enzyme.

## When NOT to use

- Pathways from KEGG category 1.7 'Glycan biosynthesis and metabolism' that use 'links' instead of nodes — these cannot be expanded or colored and are excluded from FunHoP
- When only a single gene maps to each reaction node — node expansion and count-based prioritization add nothing
- When enzyme-level (isoform) activity resolution is required — KEGG does not provide curated isoform-level enzyme activity, so analysis is limited to the gene level

## Inputs

- KEGG pathway KGML XML file (e.g. hsa00340 histidine metabolism)
- hsa-ID-to-gene-name conversion list (http://rest.kegg.jp/list/hsa)
- TCGA RNA-seq raw read-count matrix (tab-separated, genes × samples)
- Sample metadata table with cancer/normal tissue status
- Gene length table (sum of exons) and updated gene symbols list

## Outputs

- Expanded and node-aggregated pathway network files for Cytoscape visualization
- Node-level differential expression values (log2(P)×(−10)×regulation) with P < 0.05 cutoff
- Per-node read-count summary (e.g. histidine_pathway_funhop_verification.tsv) for color-scheme construction

## How to apply

Expand each multi-gene KEGG node into child nodes (one per gene) using FunHoP's ElementTree-based expansion of the KGML XML, then compute the average RNA-seq read count per gene as the mean of the cancer-group and normal-group averages, adjusting counts for gene length by (gene length as sum of exons) / average gene length over all genes. Run Voom differential expression (DEGs at P < 0.05), log2-transform P values as Value = log2(P)×(−10)×regulation (regulation = 1 up, −1 down), and style the expanded network in Cytoscape on the −1200-to-600 color gradient plus a 0 to ≥50,000 read-count scale. Then aggregate all genes within each node (naming nodes 'gene1-Bx'), sum read counts per node, and rerun differential expression at node level to render the aggregated network. The dominant gene with the highest reads (e.g. ALDH1A3 with 46,595 reads in the ALDH3A1 node) is interpreted as the likely driver of the node's enzymatic activity.

## Related tools

- **FunHoP** (Expands multi-gene KGML nodes into per-gene children and collapses node reads into one aggregated expression value) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Platform for visualizing the expanded and aggregated networks with differential-expression and read-count color gradients)
- **KEGGScape** (KGML reader used to import KEGG pathway files into Cytoscape; FunHoP corrects its first-gene-only node limitation)
- **Voom** (Differential expression analysis on genes and on aggregated nodes, with DEGs extracted at P < 0.05)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Verification table shows CNDP2 up-regulated with ~11,000 reads while down-regulated CNDP1 has near-zero reads
- ALDH1A3 dominates the ALDH3A1 node with 46,595 reads, consistent with highest-read-count dominance assumption
- HNMT (2,519 reads) and AOC1 (3,636 reads) appear down-regulated; CARNS1 and up-regulated HAL show near-zero or <100 reads
- Aggregated nodes are named 'gene1-Bx' and node-level read counts equal the sum of per-gene counts within the node
- FunHoP prints completed processing steps per pathway and ends with a summary of expression counts usable for Cytoscape color schemes

## Limitations

- Enzyme kinetics/efficiency data are usually unavailable, so equal enzyme rate across homologs is a simplification
- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may yield aberrant expression level estimates
- KEGG does not currently provide curated information on enzyme activity of isoforms; analysis is limited to the gene level
- Calculate_counts.py contains hard-coding: gene names expected in first row, tissue status expected in column 7, and gene-length correction toggled via 'correct=True' in load_countdata

## Evidence

- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] Average RNA-seq read count for each gene was calculated using the mean of the two average values calculated over cancer and normal samples: "Average RNA-seq read count for each gene was calculated using the mean of the two average values calculated over cancer and normal samples"
- [abstract] the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways.: "the KGML-reader KEGGScape only presents the first listed gene in a network node for a given reaction. This can lead to incorrect interpretations of the pathways."
- [readme] The file "Calculate_counts.py" still contains three cases of hard coding: "The file "Calculate_counts.py" still contains three cases of hard coding"
