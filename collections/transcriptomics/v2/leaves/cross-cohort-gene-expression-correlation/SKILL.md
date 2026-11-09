---
name: cross-cohort-gene-expression-correlation
description: Use when the same RNA-seq pipeline has been run on two independent patient
  cohorts and you must quantify whether node-level (FunHoP-aggregated) pathway results
  — dominant genes and read-count ratios — are consistent across cohorts.
when_to_use_negative:
- Pathways are from KEGG category 1.7 'Glycan biosynthesis and metabolism' — these
  use 'links' instead of nodes and cannot be expanded or coloured by FunHoP
- Data are already gene-length corrected, so the gene-length adjustment step (correct=True
  in load_countdata) would double-correct the counts
- The goal is pathway analysis of non-metabolic processes; the current FunHoP version
  only supports human metabolic pathways from KEGG (excluding glycan pathways)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FunHoP
  role: Expands KEGG nodes to show all homologous genes, collapses their RNA-seq read
    counts into node-level measurements, and identifies the dominant gene per node
    used for cross-cohort comparison
  repo: https://github.com/kjerstirise/FunHoP
- name: Cytoscape
  role: Visualization of the expanded/consolidated pathway networks with node colouring
    based on counts across both cohorts
- name: KEGGScape
  role: KGML reader used to load KEGG pathway files into Cytoscape; contrasted with
    FunHoP because it only presents the first listed gene in a multi-gene node
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:d2a60cf2957713d2147b590386c567e09ba838decb42620900ab521533b2666f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - Cytoscape
  - FunHoP
  - KEGGScape
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-cohort-gene-expression-correlation

## Summary

Validates reproducibility of FunHoP node-level pathway analyses by correlating log2 RNA-seq read counts and per-gene abundance ratios between two independent prostate cancer cohorts (TCGA and Prensner), and checking that dominant genes in multi-gene KEGG nodes correspond across cohorts.

## When to use

Use when the same RNA-seq pipeline has been run on two independent patient cohorts and you must quantify whether node-level (FunHoP-aggregated) pathway results — dominant genes and read-count ratios — are consistent across cohorts.

## When NOT to use

- Pathways are from KEGG category 1.7 'Glycan biosynthesis and metabolism' — these use 'links' instead of nodes and cannot be expanded or coloured by FunHoP
- Data are already gene-length corrected, so the gene-length adjustment step (correct=True in load_countdata) would double-correct the counts
- The goal is pathway analysis of non-metabolic processes; the current FunHoP version only supports human metabolic pathways from KEGG (excluding glycan pathways)

## Inputs

- TCGA gene-level RNA-seq read-count table (tab-separated)
- Prensner cohort gene-level RNA-seq read-count table (tab-separated)
- KEGG KGML pathway XML files expanded by FunHoP (85 human metabolic pathways, categories 1.1-1.11)
- hsa ID-to-gene-name conversion list (from http://rest.kegg.jp/list/hsa)
- Sample metadata with cancer/control tissue status

## Outputs

- cohort_dominant_gene_comparison.csv with per-node dominant-gene assignments
- Pearson correlation coefficients for log2 read counts and per-gene abundance ratios between cohorts
- High (>0.9) / low (<0.1) abundance-ratio concordance statistics per gene

## How to apply

Load the TCGA and Prensner gene-level read-count tables and adjust counts for gene length by the factor (sum of exons per gene / average gene length over all genes), as done in FunHoP. For each multi-gene node in the expanded KEGG networks, compute per-gene relative abundance as the fraction of each gene's read count over the total node read count, and identify the dominant gene per node (the gene assumed responsible for enzymatic activity, e.g. PLA2G2A in the 21-gene PLA2G4B node of hsa00564). Correlate log2 read counts and per-gene abundance ratios between cohorts using Pearson correlation, replicating the paper's Figure 2C-D validation. Flag genes with very high (>0.9) or very low (<0.1) abundance ratios and check that these dominant/minor genes correspond between cohorts. Write dominant-gene assignments, correlation coefficients, and ratio concordance statistics to cohort_dominant_gene_comparison.csv.

## Related tools

- **FunHoP** (Expands KEGG nodes to show all homologous genes, collapses their RNA-seq read counts into node-level measurements, and identifies the dominant gene per node used for cross-cohort comparison) — https://github.com/kjerstirise/FunHoP
- **Cytoscape** (Visualization of the expanded/consolidated pathway networks with node colouring based on counts across both cohorts)
- **KEGGScape** (KGML reader used to load KEGG pathway files into Cytoscape; contrasted with FunHoP because it only presents the first listed gene in a multi-gene node)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- Dominant genes (e.g. PLA2G2A in the 21-gene PLA2G4B node of hsa00564; CNDP2/ALDH1A3 in histidine metabolism) correspond between TCGA and Prensner
- Genes flagged with abundance ratios >0.9 or <0.1 are the same genes in both cohorts
- Pearson correlation of log2 read counts and per-gene abundance ratios between cohorts is high, matching the pattern reported for Figures 2C-D and S1-S2
- Fewer changes reach P < 0.05 in Prensner than TCGA, consistent with its smaller sample size and lower sequencing depth (an expected, not erroneous, asymmetry)
- cohort_dominant_gene_comparison.csv contains one dominant-gene assignment and correlation row per multi-gene node analyzed

## Limitations

- Prensner cohort has smaller sample size and lower sequencing depth, so fewer changes reach P < 0.05 significance
- Calculate_counts.py contains hard coding: gene names expected in first row, tissue status in column 7, and a correct flag for gene-length correction
- Equal enzyme rate is assumed because enzyme kinetics/efficiency data are usually not available
- KEGG provides no curated isoform-level enzyme activity data, so analysis is limited to the gene level
- Genes with atypical dominant isoforms may yield aberrant expression estimates under the canonical isoform assumption

## Evidence

- [other] Correlate log2 read counts and per-gene abundance ratios between TCGA and Prensner with Pearson correlation, following the paper's validation approach in Figure 2C-D.: [evidence span withheld — anchor status: not_found]
- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [readme] It uses read counts from RNA-sequencing to find the homolog in a node which is (probably) responsible for the enzymatic activity in a node.: "It uses read counts from RNA-sequencing to find the homolog in a node which is (probably) responsible for the enzymatic activity in a node."
- [other] note that fewer changes reach P < 0.05 significance in Prensner due to its smaller sample size and lower sequencing depth.: [evidence span withheld — anchor status: not_found]
