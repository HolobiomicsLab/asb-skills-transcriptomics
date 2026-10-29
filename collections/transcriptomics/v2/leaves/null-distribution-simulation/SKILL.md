---
name: null-distribution-simulation
description: Use when comparing colocalization/permutation-test null models over genomic
  vs transcriptomic backgrounds, or when establishing an empirical null distribution
  of overlap counts before running a permutation test.
when_to_use_negative:
- When testing a specific observed association between two region sets (use permTestTx
  / permTestTx_customPick directly rather than manually simulating nulls)
- When regions of interest have clear isoform ambiguity handling requirements beyond
  overlap counting (use overlapCountsTxIA via permTestTxIA_customPick)
- When the study space is genome-only with no transcriptome comparison, since a genome-based
  tool like regioneR alone already provides the null
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: Randomizes regions over transcriptome space (randomizeTx) and computes transcriptome
    overlap counts (overlapCountsTx) for the transcriptome null distribution
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: regioneR
  role: Provides numOverlaps to count pairwise overlaps for the genome-based permutation
    spaces (DNA, exonic DNA, pre-mRNA, mRNA projected)
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 annotation database used to extract the DNA, exonic DNA, pre-mRNA
    and mRNA backgrounds for the 300-gene set
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:88070bb702c3ae9b0e8e97b3736ddf52fb9bfa7355779133dbca5c2958657dc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  - regioneR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# null-distribution-simulation

## Summary

Simulates null distributions of random region-set overlaps by repeatedly randomizing pairs of region sets across genome-based (DNA, exonic DNA, pre-mRNA, mRNA-projected) and transcriptome-based permutation spaces, then comparing the resulting overlap-count distributions to reveal that null models differ significantly across spaces.

## When to use

Use when comparing colocalization/permutation-test null models over genomic vs transcriptomic backgrounds, or when establishing an empirical null distribution of overlap counts before running a permutation test.

## When NOT to use

- When testing a specific observed association between two region sets (use permTestTx / permTestTx_customPick directly rather than manually simulating nulls)
- When regions of interest have clear isoform ambiguity handling requirements beyond overlap counting (use overlapCountsTxIA via permTestTxIA_customPick)
- When the study space is genome-only with no transcriptome comparison, since a genome-based tool like regioneR alone already provides the null

## Inputs

- TxDb.Hsapiens.UCSC.hg19.knownGene annotation database (txdb)
- Set of 300 randomly selected hg19 gene identifiers defining the background
- Region-set generation parameters: 1000 random pairs, 500 regions per set, 200 nt equal region length
- RgnTX transcriptome background (mature mRNA space via exonsBy/transcript ids)

## Outputs

- Table of 1000 pairwise overlap counts per permutation space (DNA, exonic DNA, pre-mRNA, mRNA projected, transcriptome)
- Boxplot comparing the five null distributions over the same 300-gene background
- Two-sample t-test results quantifying differences between null distributions (e.g. mRNA-projected vs other backgrounds)

## How to apply

Fix the random seed (e.g. set.seed(12345677)) in R, load RgnTX with the TxDb.Hsapiens.UCSC.hg19.knownGene annotation, and extract the four backgrounds (DNA, exonic DNA, pre-mRNA, mRNA) restricted to the same 300 randomly picked hg19 genes. In each background, randomly generate 1000 pairs of region sets, each region set containing 500 regions of equal length (200 nt), using region randomization utilities (randomizeTx for the transcriptome space). Count pairwise overlaps per pair with regioneR::numOverlaps for the genome spaces and RgnTX::overlapCountsTx for the transcriptome space. Assemble the 1000 overlap counts per space into a table and draw a boxplot comparing the five null distributions; optionally run a two-sample t-test (e.g. mRNA-projected vs other backgrounds) as the article did. The rationale is that only a simulation matched to the actual permutation space yields a valid null — the article showed transcriptome-space overlaps are significantly fewer than genome-space overlaps even on the same 300 genes.

## Related tools

- **RgnTX** (Randomizes regions over transcriptome space (randomizeTx) and computes transcriptome overlap counts (overlapCountsTx) for the transcriptome null distribution) — https://github.com/yue-wang-biomath/RgnTX
- **regioneR** (Provides numOverlaps to count pairwise overlaps for the genome-based permutation spaces (DNA, exonic DNA, pre-mRNA, mRNA projected))
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 annotation database used to extract the DNA, exonic DNA, pre-mRNA and mRNA backgrounds for the 300-gene set)

## Examples

```
randomizeTx(txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, trans_ids = trans.ids, random_num = 500, type = "mature", random_length = 200)
```

## Evaluation signals

- Boxplot must show five distinguishable null distributions computed over the same 300-gene background
- Transcriptome-space overlap counts should be the fewest and significantly lower than DNA, exonic DNA and mRNA-projected counts (per the article's finding)
- Each background must contribute exactly 1000 overlap counts from pairs of region sets of 500 regions of 200 nt each
- All spaces must use the identical randomly picked 300 hg19 genes so distributions are comparable
- t-tests between spaces (e.g. mRNA-projected vs others) should yield significant differences in null distributions

## Limitations

- Random overlap counts depend heavily on the permutation space chosen; genome-based nulls are not interchangeable with transcriptome-based nulls
- Results were demonstrated on only 300 randomly picked hg19 genes and equal-length 200 nt regions; other backgrounds or region-length distributions may shift the null
- No R version or dependency versions are specified in the article/README, so reproducibility of exact distributions requires seed fixing (set.seed(12345677))

## Evidence

- [results] Background extraction restricted to same 300 genes: "All the backgrounds were region sets related to the same (randomly picked) 300 genes in hg19"
- [results] Random pair generation parameters: "In each background, 1000 pairs of region sets were randomly picked to compare the overlap of each pair, with each region set containing 500 regions of the same length"
- [results] Null distributions differ across spaces: "Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different"
- [intro] Research question on null distribution difference: "we revealed that the null distribution of random overlaps generated by the permutation model over the genome and the transcriptome could be significantly"
- [readme] randomizeTx randomizes over transcripts: [evidence span withheld — anchor status: external_doc]
