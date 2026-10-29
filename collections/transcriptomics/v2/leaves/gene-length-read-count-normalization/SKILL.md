---
name: gene-length-read-count-normalization
description: Use when raw featureCounts gene-level read counts (TopHat2/hg19 alignments,
  TCGA or Prensner PCa cohorts) must be corrected for sum-of-exons gene length before
  FunHoP node aggregation or Voom differential expression.
when_to_use_negative:
- The count matrix is already length-corrected - setting correct=True again in load_countdata
  applies the sum-of-exons factor twice and distorts node read fractions.
- Isoform- or transcript-level enzyme activity is the target - KEGG provides no curated
  isoform activity, so FunHoP limits the analysis to the gene level with a canonical
  isoform length.
- The 'expression table' is only a probe-value file used to supply gene names (column
  0) - do not length-adjust probe intensities in place of featureCounts read counts.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: FunHoP
  role: Implements the adjustment in load_countdata via the 'correct' flag, scaling
    the TCGA/Prensner count matrices before node aggregation and node-level Voom differential
    expression.
  repo: https://github.com/kjerstirise/FunHoP
- name: featureCounts
  role: Assigns TopHat2-aligned reads to genes and defines each gene length as the
    sum of exons used in the adjustment factor.
- name: Voom
  role: Runs cancer-vs-normal differential expression downstream on the gene-length-adjusted
    counts (DEGs at P < 0.05).
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:f485422bfec2f8a0626c089bad81649b1f7f03e9801975aa893703cb972f927d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - ElementTree
  - FunHoP
  - KEGGScape
  - TopHat2
  - Voom
  - featureCounts
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-length-read-count-normalization

## Summary

Scales each gene's raw RNA-seq read counts by its sum-of-exons gene length divided by the average gene length over all genes, so FunHoP can identify the dominant homolog in multi-gene KEGG nodes by true read abundance rather than length bias. The length-adjusted counts feed Voom differential expression and per-node aggregation in the TCGA and Prensner PCa analyses.

## When to use

Use when raw featureCounts gene-level read counts (TopHat2/hg19 alignments, TCGA or Prensner PCa cohorts) must be corrected for sum-of-exons gene length before FunHoP node aggregation or Voom differential expression.

## When NOT to use

- The count matrix is already length-corrected - setting correct=True again in load_countdata applies the sum-of-exons factor twice and distorts node read fractions.
- Isoform- or transcript-level enzyme activity is the target - KEGG provides no curated isoform activity, so FunHoP limits the analysis to the gene level with a canonical isoform length.
- The 'expression table' is only a probe-value file used to supply gene names (column 0) - do not length-adjust probe intensities in place of featureCounts read counts.

## Inputs

- Gene-level raw RNA-seq read-count matrix (tab-separated, genes x samples) from featureCounts on TopHat2/hg19 alignments
- Gene lengths as sum of exons per gene (from the featureCounts annotation), used to build the adjustment factor
- Gene-name list aligned to the count matrix rows (README expects gene names in the first row of the expression table)

## Outputs

- Gene-length-adjusted read-count matrix in which each gene's counts are multiplied by (sum-of-exons gene length / average gene length over all genes), tab-separated

## How to apply

Decide whether the input matrix is already length-corrected; per the README, if the data are not corrected on gene length, set correct=True when calling FunHoP's load_countdata function, otherwise leave it off to avoid applying the factor twice. For every gene, compute the factor as its gene length (sum of exons, as defined when featureCounts assigns reads) divided by the average gene length over all genes, and multiply that gene's counts in every sample (cancer and normal) by this constant. Downstream, FunHoP computes average counts as the mean of the cancer-group and normal-group averages and derives each gene's fraction of the total node read count, so the dominant homolog in a multi-gene node (e.g., PLA2G2A with ~71,482 reads in the 21-gene PLA2G4B node of hsa00564, or ALDH1A3 with 46,595 reads in the ALDH3A1 node of hsa00340) reflects abundance rather than exon-length artifacts. Apply the identical per-gene factor in both cohorts (TCGA and Prensner) so log2 counts and abundance ratios remain comparable in the cross-cohort Pearson correlation of Figure 2C-D, then run Voom differential expression (DEGs at P < 0.05) on the adjusted counts.

## Related tools

- **FunHoP** (Implements the adjustment in load_countdata via the 'correct' flag, scaling the TCGA/Prensner count matrices before node aggregation and node-level Voom differential expression.) — https://github.com/kjerstirise/FunHoP
- **featureCounts** (Assigns TopHat2-aligned reads to genes and defines each gene length as the sum of exons used in the adjustment factor.)
- **Voom** (Runs cancer-vs-normal differential expression downstream on the gene-length-adjusted counts (DEGs at P < 0.05).)

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- For spot-checked genes, adjusted count equals raw count x (sum-of-exons length / average gene length over all genes); recompute the factor and compare.
- Within each gene the per-gene constant factor leaves cancer-vs-normal sample ratios unchanged; only cross-gene comparisons shift, confirming per-gene (not per-sample) scaling.
- The adjusted matrix has the same number of genes as the expression-table gene names and the same number of samples as tissue statuses (README input contract).
- After adjustment, dominant homologs match reported values: PLA2G2A (~71,482 reads) dominates the 21-gene PLA2G4B node and ALDH1A3 (46,595 reads) dominates the ALDH3A1 node in histidine metabolism.
- Toggling 'correct' in load_countdata on an already-corrected matrix visibly inflates counts for long exons - verify the flag state matches the input matrix before each run.

## Limitations

- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression-level estimates.
- The correction behavior is hard-coded: mis-setting 'correct' in load_countdata either double-corrects or leaves raw, length-biased counts (README warning).
- Enzyme kinetics/rate data are usually unavailable, so assuming the highest-read-count gene carries activity is a simplification of enzymatic activity.

## Evidence

- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [readme] If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function: "If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function"
- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio"
- [discussion] genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation: "genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation"
- [methods] featureCounts [25] was used to assign the reads to each gene: "featureCounts [25] was used to assign the reads to each gene"
