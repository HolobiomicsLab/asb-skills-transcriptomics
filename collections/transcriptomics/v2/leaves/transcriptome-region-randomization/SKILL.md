---
name: transcriptome-region-randomization
description: Use when you need null/randomized region sets that respect transcriptome
  annotation (e.g., for permutation tests of RNA element colocalization) or want to
  benchmark shuffling runtime over region-count grids.
when_to_use_negative:
- The analysis target is purely genomic (e.g., ChIP-seq peaks on DNA space) with no
  transcriptome constraint — genome-based shufflers like regioneR are appropriate
  instead
- Features have known, unambiguous isoform association and a custom region-of-interest
  test is needed — permTestTx_customPick or permTestTx are the fitting entry points
  rather than raw randomization
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: Provides randomizeTx and randomizeFeaturesTx for transcriptome-background
    shuffling and randomizeFeaturesTx's isoform-aware permutation
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment in which RgnTX randomization and timing/benchmarking are
    executed
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: Supplies the hg19 transcriptome annotation (transcripts, exons, CDS/UTRs)
    used as the randomization background
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_001
  - task_003
  - task_002
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:3161d87773e684a926ba1e473271befe10e07fef6c3898ebfd243e3ac35d357d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  - regioneR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# transcriptome-region-randomization

## Summary

Randomizes region sets within a transcriptome background using RgnTX's randomizeTx and randomizeFeaturesTx, so null regions preserve transcriptome structure (mRNA space, alternative splicing) rather than being drawn from genome space.

## When to use

Use when you need null/randomized region sets that respect transcriptome annotation (e.g., for permutation tests of RNA element colocalization) or want to benchmark shuffling runtime over region-count grids.

## When NOT to use

- The analysis target is purely genomic (e.g., ChIP-seq peaks on DNA space) with no transcriptome constraint — genome-based shufflers like regioneR are appropriate instead
- Features have known, unambiguous isoform association and a custom region-of-interest test is needed — permTestTx_customPick or permTestTx are the fitting entry points rather than raw randomization

## Inputs

- TxDb transcriptome annotation (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene)
- Transcript IDs or 'all' specifying the sampling background
- Region set or number of regions to randomize (random_num) with desired widths (random_length)
- Region type string: 'mature', 'full', 'fiveUTR', 'CDS', or 'threeUTR'

## Outputs

- GRanges/GRangesList of randomized regions located within the transcriptome background
- Timing tables (mean elapsed seconds per function and region-set size) when benchmarking

## How to apply

Load RgnTX with the TxDb.Hsapiens.UCSC.hg19.knownGene txdb as the transcriptome background. Call randomizeTx with trans_ids, random_num (number of regions), random_length (region width, e.g., 100 or 200 nt) and type ('mature', 'full', 'fiveUTR', 'CDS', 'threeUTR') to sample regions directly over mRNA space; the README pseudocode shows it iterates N times, picking one transcript per region and drawing a region of the requested width on that transcript. For features lacking a clear isoform association, use randomizeFeaturesTx (or the isoform-aware mode) so each feature is projected to the genome and permuted only over the transcripts overlapping it, preserving feature width. When benchmarking cost, time both functions over grids of region-set sizes (e.g., 100–10000 regions) with 10 replicates per cell, aggregating mean elapsed seconds per cell. Validate that randomized regions fall within the specified transcripts and that runtime scales mainly with the number of regions, with region length having only a slight influence.

## Related tools

- **RgnTX** (Provides randomizeTx and randomizeFeaturesTx for transcriptome-background shuffling and randomizeFeaturesTx's isoform-aware permutation) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment in which RgnTX randomization and timing/benchmarking are executed)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (Supplies the hg19 transcriptome annotation (transcripts, exons, CDS/UTRs) used as the randomization background)

## Examples

```
randomResults <- randomizeTx(txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, trans_ids = c("170", "782", "974", "1364", "1387"), random_num = 10, type = "mature", random_length = 100)
```

## Evaluation signals

- Each randomized region lies on one of the specified trans_ids transcripts with the requested width (random_length)
- Benchmark timing tables contain one row per (function, region-set size) cell with elapsed times that increase approximately monotonically with region count
- Runtime scales mainly with the number of regions; region length has only a slight influence, reproducing Tables 1–2 behavior
- For isoform-ambiguity randomization, the fraction of randomized regions located on their permitted transcripts is reported and checked
- Randomized region sets, when compared to genome-based nulls (DNA, exonic DNA, pre-mRNA, mRNA projected), give clearly fewer overlap counts on the transcriptome space

## Limitations

- Runtime grows with region count, so very large region sets (e.g., 10000 regions) are computationally expensive per replicate
- randomizeTx requires regions to be placed on specified transcripts; it does not itself resolve isoform ambiguity — use randomizeFeaturesTx / IA modes for features lacking clear isoform association
- No version number or R/dependency versions are stated in the article or README, so reproducibility of exact timings across environments is not guaranteed

## Evidence

- [discussion] RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space.: "RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space."
- [readme] The function `randomizeTx` randomly picks regions over specified transcripts.: [evidence span withheld — anchor status: external_doc]
- [methods] For features with isoform ambiguity, RgnTX's isoform-aware randomization projects each feature to the genome and then permutes it only over the transcripts that overlap with it: [evidence span withheld — anchor status: not_found]
- [results] Benchmarking shows runtime scales mainly with the number of regions, with region length having only a slight influence.: [evidence span withheld — anchor status: not_found]
