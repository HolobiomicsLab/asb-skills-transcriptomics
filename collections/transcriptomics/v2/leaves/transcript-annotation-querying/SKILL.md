---
name: transcript-annotation-querying
description: Use when transcriptome elements (e.g., POSTAR3 m6A sites, miCLIP-seq
  peaks) carry only genomic coordinates and the specific isoform transcript they belong
  to is unknown, and isoform-aware randomization or colocalization testing is needed.
when_to_use_negative:
- The features already have a clear isoform association (use RgnTX's isoform-aware-free
  modes like permTestTx with overlapCountsTx on a specified transcript type instead)
- The analysis target is purely genome-based colocalization where transcriptome heterogeneity
  is irrelevant (existing genome-based tools like regioneR, GAT, or OLOGRAM suffice)
- The input regions are not transcriptome elements and need no transcriptomic null
  model (e.g., whole-genome ChIP peaks against genome backgrounds)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: RgnTX
  role: Provides permTestTxIA_customPick, randomizeTx, overlapCountsTxIA, and getStopCodon
    to query isoform overlap and run isoform-aware permutation tests
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcript annotation database used as the txdb source for extracting
    exons, CDS, and transcript IDs
- name: R
  role: Runtime environment (R/Bioconductor) in which RgnTX is executed
  repo: https://bioconductor.org/packages/RgnTX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:4652844f0efde820a341655d28a662ef373eb580e8cf77e00699b860f11438c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# transcript-annotation-querying

## Summary

Query transcript annotations (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene via RgnTX) to resolve which isoform transcripts overlap a genomic feature when its isoform association is unknown, so downstream isoform-aware permutation only draws replacement regions from permitted transcripts.

## When to use

Use when transcriptome elements (e.g., POSTAR3 m6A sites, miCLIP-seq peaks) carry only genomic coordinates and the specific isoform transcript they belong to is unknown, and isoform-aware randomization or colocalization testing is needed.

## When NOT to use

- The features already have a clear isoform association (use RgnTX's isoform-aware-free modes like permTestTx with overlapCountsTx on a specified transcript type instead)
- The analysis target is purely genome-based colocalization where transcriptome heterogeneity is irrelevant (existing genome-based tools like regioneR, GAT, or OLOGRAM suffice)
- The input regions are not transcriptome elements and need no transcriptomic null model (e.g., whole-genome ChIP peaks against genome backgrounds)

## Inputs

- Genomic coordinates of transcriptome features without isoform association (e.g., m6A_sites_data.rds from RgnTX extdata, POSTAR3 RBP sites, miCLIP-seq m6A sites)
- TxDb.Hsapiens.UCSC.hg19.knownGene transcript annotation (TxDb object)
- A customPick_function (e.g., getStopCodon) mapping each feature's aligned isoforms to regions of interest
- Number of permutations (ntimes, e.g., 50-100)

## Outputs

- Set of K overlapping isoform transcript IDs per feature
- Randomized regions restricted to permitted transcripts, preserving feature width
- permTestTx results: observed_overlap, random_region_list, random_overlap_list, pvalue, zscore
- Per-region verification table (feature, randomized coordinates, associated transcripts, on-permitted-transcript flag), e.g., isoform_ambiguity_randomization_check.csv

## How to apply

Load the RgnTX package with the TxDb.Hsapiens.UCSC.hg19.knownGene annotation (set seed, e.g. 12345677, for reproducibility). For each feature with isoform ambiguity, project the genomic coordinates to the transcriptome and query the TxDb to find the K isoform transcripts the feature can be aligned to (RgnTX's 'Find K isoforms that RS1_i can be aligned to' step in permTestTxIA_customPick). Restrict any randomization to these permitted transcripts, preserving the original feature's width (random regions drawn with the same width over each of the K isoforms, with per-feature counts divided by K when aggregating). Use overlapCountsTxIA for evaluating features lacking isoform association rather than overlapCountsTx. Correct for multiplicity with Benjamini-Hochberg when testing many feature groups (the article's m6A analysis used groups of 100 regions, 50-100 permutations).

## Related tools

- **RgnTX** (Provides permTestTxIA_customPick, randomizeTx, overlapCountsTxIA, and getStopCodon to query isoform overlap and run isoform-aware permutation tests) — https://github.com/yue-wang-biomath/RgnTX
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcript annotation database used as the txdb source for extracting exons, CDS, and transcript IDs)
- **R** (Runtime environment (R/Bioconductor) in which RgnTX is executed) — https://bioconductor.org/packages/RgnTX

## Examples

```
txdb <- TxDb.Hsapiens.UCSC.hg19.knownGene; m6A_sites_data <- readRDS(system.file(package="RgnTX", "extdata", "m6A_sites_data.rds")); permTestTx_results <- permTestTxIA_customPick(RS1 = m6A_sites_data[1:500], txdb = txdb, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTx, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- Every randomized region must fall on one of the K transcripts that overlap the original projected feature (on-permitted-transcript flag true for 100% of regions)
- Randomized regions preserve the original feature's width exactly (width of random_region_i equals width of RS1_i)
- Per-feature overlap counts are averaged over the K candidate isoforms (divided by K) when aggregating observed and random overlaps
- Resulting shifted z-scores and BH-adjusted p-values are returned by permTestTxIA_customPick and should use overlapCountsTxIA for RS1 and overlapCountsTx for evaluation functions
- Re-running with the same seed (e.g., set.seed(12345677)) reproduces the same randomization output

## Limitations

- No version number or dependency versions for RgnTX or R are stated in the article
- The method applies only when features can be mapped to the genome; features whose genomic position is itself uncertain cannot be handled
- Existing genome-based colocalization methods do not account for transcriptome heterogeneity, so results from RgnTX's novel testing modes are not directly comparable to genome-based null models

## Evidence

- [readme] Importantly, RgnTX supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, RgnTX supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [readme] The function permTestTxIA_customPick performs permutation test to evaluate association between genomic features (RS) that one only know its genomic positions and do not know which specific transcript is should be aligned to: [evidence span withheld — anchor status: external_doc]
- [readme] Find K isoforms that RS1_i can be aligned to: [evidence span withheld — anchor status: external_doc]
- [intro] the RNA-binding protein target sites collected in POSTAR3 database [32] lack information about the specific isoform they originate from: "the RNA-binding protein target sites collected in POSTAR3 database [32] lack information about the specific isoform they originate from"
- [intro] To date, there are no computational methods available to address the issue of isoform ambiguity during transcriptome element colocalization analysis.: "To date, there are no computational methods available to address the issue of isoform ambiguity during transcriptome element colocalization analysis."
