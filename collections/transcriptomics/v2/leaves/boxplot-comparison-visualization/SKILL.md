---
name: boxplot-comparison-visualization
description: Use when you have replicate-level null statistics (e.g., 1000 random
  pairwise overlap counts) computed under several permutation backgrounds and need
  to visually compare their distributions to justify which background a colocalization
  test should use.
when_to_use_negative:
- You have only a single observed overlap statistic per space (no replicate null counts)
  — use a permutation test with z-score/p-value instead of a distribution comparison
  boxplot.
- The region sets under comparison were generated on different gene backgrounds, making
  the null distributions non-comparable in one plot.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0797
tools:
- name: RgnTX
  role: Provides overlapCountsTx to count overlaps on the transcriptome permutation
    space and randomization utilities for the transcriptome background
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: regioneR
  role: Provides numOverlaps to count pairwise overlaps for the DNA, exonic DNA, pre-mRNA
    and mRNA (projected) spaces
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 annotation used to extract the four backgrounds restricted to the
    same 300 randomly picked genes
- name: R
  role: Environment in which backgrounds are extracted, overlaps counted, the counts
    table assembled, and the boxplot drawn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:091de767553f7f49eed0fc1f1580e4c00a31fc055de5379a6066238295be1825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  - regioneR
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# boxplot-comparison-visualization

## Summary

Assembles replicate-level overlap counts (e.g., 1000 null overlap counts per permutation space) into a table and draws a boxplot comparing null distributions across genome- and transcriptome-based permutation spaces, exposing whether space choice shifts the null.

## When to use

Use when you have replicate-level null statistics (e.g., 1000 random pairwise overlap counts) computed under several permutation backgrounds and need to visually compare their distributions to justify which background a colocalization test should use.

## When NOT to use

- You have only a single observed overlap statistic per space (no replicate null counts) — use a permutation test with z-score/p-value instead of a distribution comparison boxplot.
- The region sets under comparison were generated on different gene backgrounds, making the null distributions non-comparable in one plot.

## Inputs

- 1000 random overlap counts per permutation space (DNA, exonic DNA, pre-mRNA, mRNA projected, transcriptome)
- Shared gene background restricted to 300 randomly picked hg19 genes from TxDb.Hsapiens.UCSC.hg19.knownGene
- Region-set pairs of 500 equal-length 200 nt regions per background

## Outputs

- A table of 1000 overlap counts per permutation space
- A boxplot comparing null distributions across the five permutation spaces

## How to apply

Fix a random seed (set.seed(12345677)) and extract all backgrounds from the same gene set — here 300 randomly picked hg19 genes from TxDb.Hsapiens.UCSC.hg19.knownGene covering DNA, exonic DNA, pre-mRNA, and mRNA spaces. In each background, generate 1000 random pairs of region sets, each region set containing 500 equal-length regions (200 nt), and count pairwise overlaps per pair using regioneR::numOverlaps for the genome-based spaces and RgnTX::overlapCountsTx for the transcriptome space. Assemble the 1000 counts per space into a single table keyed by space and draw a side-by-side boxplot so the five null distributions are directly comparable on the same 300-gene background. Optionally support the visual comparison with a two-sample t-test (as the paper did between 'mRNA (projected)' and other backgrounds) before interpreting the boxplot.

## Related tools

- **RgnTX** (Provides overlapCountsTx to count overlaps on the transcriptome permutation space and randomization utilities for the transcriptome background) — https://github.com/yue-wang-biomath/RgnTX
- **regioneR** (Provides numOverlaps to count pairwise overlaps for the DNA, exonic DNA, pre-mRNA and mRNA (projected) spaces)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 annotation used to extract the four backgrounds restricted to the same 300 randomly picked genes)
- **R** (Environment in which backgrounds are extracted, overlaps counted, the counts table assembled, and the boxplot drawn)

## Examples

```
set.seed(12345677); counts_tx <- sapply(1:1000, function(i) RgnTX::overlapCountsTx(randomizeTx(txdb, trans.ids, 500, 200), RS2)); boxplot(counts_dna, counts_exonic, counts_premrna, counts_mrna, counts_tx, names=c('DNA','exonic','pre-mRNA','mRNA','transcriptome'))
```

## Evaluation signals

- The boxplot displays five distinguishable null distributions over the same 300-gene background.
- Each box summarizes exactly 1000 overlap counts (one per random region-set pair).
- Overlapping counts differ significantly between backgrounds, with the fewest overlaps on the transcriptome space, matching the reported finding.
- All backgrounds were extracted from the same randomly picked 300 hg19 genes, so distributions are directly comparable.
- A fixed seed (set.seed(12345677)) makes the random region-set generation reproducible.

## Limitations

- The comparison is restricted to a 300-gene background; broader gene sets may change the null distributions.
- Region sets are limited to 500 equal-length 200 nt regions; unequal or variable-length region sets are not covered by this comparison.
- RgnTX has no version number stated in the article, and no R version or dependency versions are specified, complicating exact reproduction.

## Evidence

- [results] All the backgrounds were region sets related to the same (randomly picked) 300 genes in hg19: "All the backgrounds were region sets related to the same (randomly picked) 300 genes in hg19"
- [results] In each background, 1000 pairs of region sets were randomly picked to compare the overlap of each pair, with each region set containing 500 regions of the same length: "In each background, 1000 pairs of region sets were randomly picked to compare the overlap of each pair, with each region set containing 500 regions of the same length"
- [results] a two-sample t-test was conducted to evaluate the difference in the null distribution of random overlaps on ‘mRNA (projected)’ and overlaps on other backgrounds: "a two-sample t-test was conducted to evaluate the difference in the null distribution of random overlaps on ‘mRNA (projected)’ and overlaps on other backgrounds"
- [results] We used the annotation database from UCSC to extract the following backgrounds: "We used the annotation database from UCSC to extract the following backgrounds"
