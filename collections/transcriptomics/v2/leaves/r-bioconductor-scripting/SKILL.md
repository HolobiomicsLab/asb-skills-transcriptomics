---
name: r-bioconductor-scripting
description: Use when a task requires R scripting against Bioconductor annotation
  packages (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene) to manipulate GRanges/GRangesList
  region sets, run RgnTX randomization or permutation functions, or benchmark runtime
  over region-set grids.
when_to_use_negative:
- Input is genomic feature data without transcriptome context and a genome-based colocalization
  tool (regioneR, LOLA, GenomeRunner) suffices
- Transcriptome elements have clear isoform association AND the analysis goal is unrelated
  to transcriptome backgrounds or RgnTX randomization performance
- The analysis needs to produce publication figures or hypothesis tests rather than
  reusable R scripts — a one-off interactive session may suffice
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RgnTX
  role: R/Bioconductor package providing randomizeTx, randomizeFeaturesTx, permTestTx
    and related functions whose scripting and benchmarking this skill covers
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Implementation language in which the annotation loading, randomization loops,
    timing, and aggregation scripts are written
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: Bioconductor annotation package supplying the UCSC hg19 transcriptome background
    extracted by RgnTX functions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:9a4865fdcb8bdc15cdf705c3f169a5348114499019052b79fcf1cdc0c4b32c03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# r-bioconductor-scripting

## Summary

Writing R/Bioconductor scripts — as used by RgnTX for transcriptome-level colocalization analysis and runtime benchmarking of its randomization functions (randomizeTx, randomizeFeaturesTx) against the UCSC hg19 transcriptome annotation.

## When to use

Use when a task requires R scripting against Bioconductor annotation packages (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene) to manipulate GRanges/GRangesList region sets, run RgnTX randomization or permutation functions, or benchmark runtime over region-set grids.

## When NOT to use

- Input is genomic feature data without transcriptome context and a genome-based colocalization tool (regioneR, LOLA, GenomeRunner) suffices
- Transcriptome elements have clear isoform association AND the analysis goal is unrelated to transcriptome backgrounds or RgnTX randomization performance
- The analysis needs to produce publication figures or hypothesis tests rather than reusable R scripts — a one-off interactive session may suffice

## Inputs

- TxDb.Hsapiens.UCSC.hg19.knownGene transcriptome annotation database
- Transcript IDs (e.g., '170', '1387', '4113', '10715') or 'all' for transcript selection
- Region sets as GRanges/GRangesList objects (e.g., m6A sites read from m6A_sites_data.rds)
- Randomization parameters: random_num, random_length, type ('mature'/'full'/'fiveUTR'/'CDS'/'threeUTR'), N permutations

## Outputs

- GRangesList of randomized regions returned by randomizeTx / randomizeFeaturesTx
- permTestTx result objects with observed_overlap, random_overlap_list, pvalue, and zscore
- Timing table (e.g., timing_tables.csv) with one row per (function, region-set size) cell and mean elapsed seconds over replicates

## How to apply

Load RgnTX (installed via BiocManager or devtools from github.com/yue-wang-biomath/RgnTX) together with TxDb.Hsapiens.UCSC.hg19.knownGene to supply the transcriptome background. Build region sets with randomizeTx by specifying txdb, trans_ids, random_num, random_length and a type such as 'mature', 'full', 'fiveUTR', 'CDS', or 'threeUTR'; the function loops N times picking random regions of the requested widths over the chosen transcripts and returns a GRangesList. For benchmarking, time randomizeTx and randomizeFeaturesTx over a grid of region-set sizes (e.g., 100–10000 regions) with ~10 replicates per cell, then aggregate mean and spread of elapsed seconds into a table (e.g., timing_tables.csv). For statistical tests, feed randomized region sets into permTestTx or permTestTxIA_customPick with ev_function_1/ev_function_2 = overlapCountsTx or overlapCountsTxIA, ntimes permutations, and adjust p-values with Benjamini-Hochberg correction.

## Related tools

- **RgnTX** (R/Bioconductor package providing randomizeTx, randomizeFeaturesTx, permTestTx and related functions whose scripting and benchmarking this skill covers) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Implementation language in which the annotation loading, randomization loops, timing, and aggregation scripts are written)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (Bioconductor annotation package supplying the UCSC hg19 transcriptome background extracted by RgnTX functions)

## Examples

```
library(RgnTX); library(TxDb.Hsapiens.UCSC.hg19.knownGene); randomResults <- randomizeTx(txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, trans_ids = c("170","782","974","1364","1387"), random_num = 10, type = "mature", random_length = 100)
```

## Evaluation signals

- timing_tables.csv exists with one row per (function, region-set size) cell and populated elapsed-time fields
- Measured runtime increases approximately monotonically with region-set size, consistent with Tables 1–2 of the paper
- randomizeTx returns a GRangesList whose per-element widths match the requested random_length values
- permTestTx outputs contain pvalue and zscore fields computed from the ntimes random overlap draws
- Adjusted p-values reflect Benjamini-Hochberg correction (reduced false positives relative to raw p-values)

## Limitations

- Runtime of randomization functions scales mainly with the number of regions, so very large region sets (e.g., 10000 regions) dominate benchmark wall-clock time
- No R version or dependency versions are specified for RgnTX, so reproducibility of timing measurements across environments is not guaranteed
- randomizeTx requires transcript IDs present in the supplied txdb; IDs absent from TxDb.Hsapiens.UCSC.hg19.knownGene will fail

## Evidence

- [abstract] RgnTX, an R/Bioconductor tool for the colocalization analysis of transcriptome elements with permutation tests: "Here, we developed RgnTX, an R/Bioconductor tool for the colocalization analysis of transcriptome elements with permutation tests."
- [readme] randomizeTx randomly picks regions over specified transcripts: [evidence span withheld — anchor status: external_doc]
- [discussion] RgnTX provides randomization functions that can shuffle regions within a transcriptome background: "RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space."
- [results] We used the annotation database from UCSC to extract the following backgrounds: "We used the annotation database from UCSC to extract the following backgrounds"
- [results] RgnTX provided Benjamini-Hochberg correction to adjust the p-values: "RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives"
