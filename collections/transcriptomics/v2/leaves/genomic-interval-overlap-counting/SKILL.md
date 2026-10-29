---
name: genomic-interval-overlap-counting
description: Use when comparing null distributions of random overlap counts between
  region sets over different genomic or transcriptome backgrounds, e.g., quantifying
  how many overlaps arise by chance over DNA vs mRNA permutation spaces.
when_to_use_negative:
- When the goal is testing a real observed association between two region sets (use
  permTestTx / permTestTx_customPick with p-values and z-scores rather than raw overlap
  counting)
- When region sets have heterogeneous lengths or unknown isoform association that
  requires the isoform-aware overlapCountsTxIA measure instead of plain interval overlap
  counting
- When the background must preserve transcriptome structure (alternative splicing)
  — plain genome-based overlap counting ignores isoform heterogeneity by design
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0085
tools:
- name: RgnTX
  role: Provides overlapCountsTx to count overlaps of region pairs on the transcriptome
    permutation space and randomizeTx to generate randomized regions
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: regioneR
  role: Provides numOverlaps to count pairwise overlaps for the genome-based spaces
    (DNA, exonic DNA, pre-mRNA, mRNA projected)
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 annotation database used to extract the DNA, exonic DNA, pre-mRNA
    and mRNA backgrounds for the 300 randomly picked genes
- name: R
  role: Implementation environment for the overlap-counting and randomization workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:d0ab219466e4e7610d3f0398331644891bd1e8c7c0d6dbc6aee8834969bd1714
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  - regioneR
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genomic-interval-overlap-counting

## Summary

Count pairwise overlaps between genomic/transcriptomic region sets (e.g., 1000 random pairs of 500 equal-length 200 nt regions) to characterize null distributions of random overlaps across genome-based (DNA, exonic DNA, pre-mRNA, mRNA projected) and transcriptome-based permutation spaces, as done in the RgnTX paper.

## When to use

Use when comparing null distributions of random overlap counts between region sets over different genomic or transcriptome backgrounds, e.g., quantifying how many overlaps arise by chance over DNA vs mRNA permutation spaces.

## When NOT to use

- When the goal is testing a real observed association between two region sets (use permTestTx / permTestTx_customPick with p-values and z-scores rather than raw overlap counting)
- When region sets have heterogeneous lengths or unknown isoform association that requires the isoform-aware overlapCountsTxIA measure instead of plain interval overlap counting
- When the background must preserve transcriptome structure (alternative splicing) — plain genome-based overlap counting ignores isoform heterogeneity by design

## Inputs

- TxDb.Hsapiens.UCSC.hg19.knownGene annotation database (txdb object)
- Region-set backgrounds (DNA, exonic DNA, pre-mRNA, mRNA) restricted to 300 randomly picked hg19 genes
- Randomized region sets (GRanges) generated per background, e.g., 500 regions of 200 nt each

## Outputs

- Table of 1000 overlap counts per permutation space (DNA, exonic DNA, pre-mRNA, mRNA projected, transcriptome)
- Boxplot comparing null distributions of overlap counts across the five permutation spaces
- Two-sample t-test statistics comparing null distributions across backgrounds

## How to apply

Fix a random seed (set.seed(12345677)) and load RgnTX together with TxDb.Hsapiens.UCSC.hg19.knownGene. Extract region-set backgrounds restricted to the same randomly picked 300 hg19 genes (DNA, exonic DNA, pre-mRNA, mRNA from UCSC hg19 annotation). In each background, randomly generate 1000 pairs of region sets, each region set containing 500 regions of the same length (200 nt), using the vignette's region randomization utilities such as randomizeTx with a specified random_num and random_length. Count pairwise overlaps with regioneR::numOverlaps for the genome-based spaces (DNA, pre-mRNA, mRNA projected, exonic DNA) and RgnTX::overlapCountsTx for the transcriptome space. Assemble the 1000 overlap counts per space into a table, draw a boxplot comparing the five null distributions, and optionally assess differences with a two-sample t-test (e.g., mRNA (projected) vs other backgrounds).

## Related tools

- **RgnTX** (Provides overlapCountsTx to count overlaps of region pairs on the transcriptome permutation space and randomizeTx to generate randomized regions) — https://github.com/yue-wang-biomath/RgnTX
- **regioneR** (Provides numOverlaps to count pairwise overlaps for the genome-based spaces (DNA, exonic DNA, pre-mRNA, mRNA projected))
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 annotation database used to extract the DNA, exonic DNA, pre-mRNA and mRNA backgrounds for the 300 randomly picked genes)
- **R** (Implementation environment for the overlap-counting and randomization workflow)

## Examples

```
randomResults <- randomizeTx(txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, trans_ids = trans.ids, random_num = 10, type = "mature", random_length = 100)
```

## Evaluation signals

- Boxplot displays five distinguishable null distributions over the same 300-gene background (DNA, exonic DNA, pre-mRNA, mRNA projected, transcriptome)
- Overlap counts on the transcriptome space are significantly fewer than those from the DNA, exonic DNA and mRNA (projected) spaces
- Each space contributes exactly 1000 overlap counts from 1000 randomly picked pairs of region sets, each set containing 500 regions of 200 nt
- Two-sample t-test comparing 'mRNA (projected)' against other backgrounds yields a statistically significant difference
- Random seed (set.seed(12345677)) reproduces the same overlap-count table across runs

## Limitations

- All backgrounds were restricted to only 300 genes, so overlap counts per background depend on the sampled gene set
- Genome-based overlap counting does not account for transcriptome heterogeneity — multiple isoform transcripts sharing exons when mapped to the genome
- No R version or dependency versions specified in the article, so exact numeric results may vary across package versions

## Evidence

- [results] Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different: "Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different"
- [results] In each background, 1000 pairs of region sets were randomly picked to compare the overlap of each pair, with each region set containing 500 regions of the same length: "In each background, 1000 pairs of region sets were randomly picked to compare the overlap of each pair, with each region set containing 500 regions of the same length"
- [readme] The function randomizeTx randomly picks regions over specified transcripts.: [evidence span withheld — anchor status: external_doc]
- [results] We used the annotation database from UCSC to extract the following backgrounds: "We used the annotation database from UCSC to extract the following backgrounds"
- [intro] existing approaches have a fundamental limitation as they were primarily designed for genome-based analysis and do not account for the heterogeneity of the transcriptome: "existing approaches have a fundamental limitation as they were primarily designed for genome-based analysis and do not account for the heterogeneity of the transcriptome"
