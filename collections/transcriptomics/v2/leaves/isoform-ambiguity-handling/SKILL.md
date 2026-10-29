---
name: isoform-ambiguity-handling
description: Use when transcriptome elements (e.g. POSTAR3 RBP sites or miCLIP-seq
  m6A sites) have known genomic positions but no clear isoform association, and a
  permutation-based colocalization test at the transcriptome level is required.
when_to_use_negative:
- Features already have a clear isoform association — use randomizeTx/permTestTx with
  overlapCountsTx on specified transcript IDs instead
- The analysis is purely genome-based with no transcriptome annotation involved —
  existing genome-based tools (GREAT, LOLA, GenomeRunner, regioneR) are the appropriate
  choice
- The input is not region data but, e.g., an expression matrix or feature table with
  no genomic coordinates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3512
tools:
- name: RgnTX
  role: R/Bioconductor package providing randomizeTx, permTestTxIA_customPick and
    overlapCountsTx for isoform-aware permutation of ambiguous transcriptome elements
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Execution environment in which RgnTX randomization and permutation tests are
    run
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcriptome annotation supplying transcript, exon and CDS structures
    used to map features and restrict randomization to permitted transcripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:92ee3c69d9cdde1d1771245aa96639b4f34e69f69bf9623224776ca2ed54e38b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# isoform-ambiguity-handling

## Summary

Handles transcriptome elements with unknown isoform association in RgnTX colocalization analysis by projecting features to the genome, collecting all overlapping isoform transcripts, and permuting each feature only over those permitted transcripts while preserving feature width.

## When to use

Use when transcriptome elements (e.g. POSTAR3 RBP sites or miCLIP-seq m6A sites) have known genomic positions but no clear isoform association, and a permutation-based colocalization test at the transcriptome level is required.

## When NOT to use

- Features already have a clear isoform association — use randomizeTx/permTestTx with overlapCountsTx on specified transcript IDs instead
- The analysis is purely genome-based with no transcriptome annotation involved — existing genome-based tools (GREAT, LOLA, GenomeRunner, regioneR) are the appropriate choice
- The input is not region data but, e.g., an expression matrix or feature table with no genomic coordinates

## Inputs

- GRanges/GRangesList of transcriptome or genomic features lacking isoform association (e.g. m6A_sites_data.rds, first 500 sites)
- TxDb transcriptome annotation (TxDb.Hsapiens.UCSC.hg19.knownGene)
- customPick_function defining regions of interest per isoform (e.g. getStopCodon, getCDS)
- Random seed for reproducibility

## Outputs

- permTestTxIA_customPick result object with observed_overlap, random_region_list, random_overlap_list, pvalue and zscore
- Per-region verification table (feature, randomized region coordinates, associated transcripts, on-permitted-transcript flag) written to isoform_ambiguity_randomization_check.csv
- Fraction of randomized regions located on permitted transcripts

## How to apply

Load RgnTX with a TxDb annotation such as TxDb.Hsapiens.UCSC.hg19.knownGene and fix a seed (e.g. 12345677) for reproducibility. For each isoform-ambiguous feature, project it from the transcriptome to the genome and find the K isoform transcripts it can be aligned to; per the README pseudocode, for each of the K isoforms a random region of the same width as the feature is drawn over that isoform, and per-feature overlaps are averaged as overlap/K. Use permTestTxIA_customPick with ev_function_1 = overlapCountsTxIA (the isoform-aware evaluation function) and a customPick_function such as getStopCodon to define the regions of interest, running e.g. 50 permutations (ntimes = 50). Verify correctness by checking that every randomized region lies on one of the permitted overlapping transcripts and that feature widths are preserved, then write a per-region table (feature, randomized coordinates, associated transcripts, on-permitted-transcript flag) to CSV and report the fraction of randomized regions located on permitted transcripts.

## Related tools

- **RgnTX** (R/Bioconductor package providing randomizeTx, permTestTxIA_customPick and overlapCountsTx for isoform-aware permutation of ambiguous transcriptome elements) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Execution environment in which RgnTX randomization and permutation tests are run)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcriptome annotation supplying transcript, exon and CDS structures used to map features and restrict randomization to permitted transcripts)

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = m6A_sites_data[1:500], txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTx, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- Every randomized region overlaps at least one of the K transcripts that the original feature overlaps when projected to the genome (on-permitted-transcript flag TRUE)
- Width of each randomized region equals the width of the original feature (width preserved)
- Fraction of randomized regions located on permitted transcripts reported in the CSV check table; unexpected zeros indicate mis-projection
- Per-feature overlaps averaged over K isoforms (overlap/K) as implemented in overlapCountsTxIA
- Fixed seed (e.g. 12345677) reproduces identical randomized regions across runs

## Limitations

- Randomized regions are restricted to transcripts overlapping the projected feature, so the null space is limited to those isoforms and may be small for uniquely-mapped features
- No version number, R version or dependency versions are stated for the protocol, affecting reproducibility
- The method targets isoform ambiguity at the transcriptome level; it does not substitute for genome-based colocalization tools when transcriptome annotation is irrelevant

## Evidence

- [other] For features with isoform ambiguity, RgnTX's isoform-aware randomization projects each feature to the genome and then permutes it only over the transcripts that overlap with it: [evidence span withheld — anchor status: not_found]
- [abstract] Importantly, it supports the testing of transcriptome elements without clear isoform association: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [readme] permTestTxIA_customPick performs permutation test to evaluate association between genomic features (RS) that one only know its genomic positions: [evidence span withheld — anchor status: external_doc]
- [readme] random_region_i_k: pick a random region having the same width with RS1_i over the k-th isoform: [evidence span withheld — anchor status: external_doc]
- [intro] the RNA-binding protein target sites collected in POSTAR3 database lack information about the specific isoform they originate from: "the RNA-binding protein target sites collected in POSTAR3 database [32] lack information about the specific isoform they originate from"
