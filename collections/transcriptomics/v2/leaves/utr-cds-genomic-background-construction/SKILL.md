---
name: utr-cds-genomic-background-construction
description: Use when a region set of transcriptomic features (e.g. m7G/m6A methylation
  sites with unknown isoform origin) must be tested for colocalization against 5'UTR
  or CDS regions using permutation-based null models in R/RgnTX.
when_to_use_negative:
- When the region set already has clear isoform association — use permTestTx/permTestTx_customPick
  with overlapCountsTx instead of the IA mode
- When a genome-based colocalization background is intended (e.g. whole-genome shuffle
  via regioneR/BEDTools); RgnTX null models randomize over mRNA/transcriptome space
  and are not interchangeable with genome-based shuffling
- When testing associations with region types other than 5'UTR/CDS (e.g. stop codons
  or threeUTR) without adapting the customPick function accordingly
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0085
tools:
- name: RgnTX
  role: Provides permTestTxIA_customPick, randomizeTx and overlapCountsTxIA to build
    customPick backgrounds and run transcriptome-wide permutation tests for 5'UTR/CDS
    colocalization
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Implementation language in which the customPick functions and permutation
    workflow are written and executed
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcript annotation database used to extract 5'UTR and CDS region
    sets for the backgrounds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:d09070e89cfdb4698f5527631af84e11246b940802ee43a9dcaf683bf6ec0b84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# utr-cds-genomic-background-construction

## Summary

Constructs 5'UTR and CDS transcriptome-region backgrounds as customPick functions for RgnTX permutation tests, so that associations between features such as m7G (or m6A) sites and specific transcriptome regions can be assessed against a transcriptome-wide shuffling null model rather than a genome-based one.

## When to use

Use when a region set of transcriptomic features (e.g. m7G/m6A methylation sites with unknown isoform origin) must be tested for colocalization against 5'UTR or CDS regions using permutation-based null models in R/RgnTX.

## When NOT to use

- When the region set already has clear isoform association — use permTestTx/permTestTx_customPick with overlapCountsTx instead of the IA mode
- When a genome-based colocalization background is intended (e.g. whole-genome shuffle via regioneR/BEDTools); RgnTX null models randomize over mRNA/transcriptome space and are not interchangeable with genome-based shuffling
- When testing associations with region types other than 5'UTR/CDS (e.g. stop codons or threeUTR) without adapting the customPick function accordingly

## Inputs

- m7G (or m6A) site region set, e.g. RDS file extdata/m6A_sites_data.rds shipped with RgnTX, with genomic positions but no isoform association
- TxDb.Hsapiens.UCSC.hg19.knownGene UCSC hg19 transcript annotation (txdb object)
- customPick functions returning 5'UTR and CDS GRanges for given transcript ids

## Outputs

- permTestTxIA_customPick result objects containing ROI, random_region_list, observed_overlap, random_overlap_list, pvalue and zscore for the 5'UTR background
- equivalent result objects for the CDS background
- summary-statistics plots of observed overlap vs. null distribution (e.g. fig_S2_summary_statistics.png)

## How to apply

Load the m7G (or other feature) region set and TxDb.Hsapiens.UCSC.hg19.knownGene as the txdb annotation in R. Define customPick functions that extract the 5'UTR and CDS regions from annotated transcripts (mirroring the package's getCDS/getStopCodon pattern, e.g. via cdsBy(txdb, use.names=FALSE) intersected with the transcript ids). Because feature sites typically lack isoform information, run the IA (isoform-ambiguity) mode test permTestTxIA_customPick with ev_function_1 = overlapCountsTxIA and ev_function_2 = overlapCountsTx, which averages observed and random overlaps over the K isoforms each site can align to. Set ntimes (e.g. 50 in the README examples) to generate the null distribution of overlap counts, then compute p-value and z-score from observed vs. random overlaps. Finally plot observed overlap relative to the null distribution (summary-statistics plots) to reproduce Fig. S2-style outputs for the 5'UTR and CDS associations.

## Related tools

- **RgnTX** (Provides permTestTxIA_customPick, randomizeTx and overlapCountsTxIA to build customPick backgrounds and run transcriptome-wide permutation tests for 5'UTR/CDS colocalization) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Implementation language in which the customPick functions and permutation workflow are written and executed)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcript annotation database used to extract 5'UTR and CDS region sets for the backgrounds)

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = m7G_sites_data, txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, customPick_function = getFiveUTR, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- The permTestTxIA_customPick result contains non-empty ROI, random_overlap_list of length ntimes, and finite pvalue and zscore
- Observed_overlap should fall within (or be reported relative to) the range of random_overlap_list values; z-score sign indicates enrichment or depletion vs. the null
- Plot reproduces the Fig. S2 summary-statistics form: observed overlap marked against the permutation null distribution for the 5'UTR and CDS separately
- Custom pick functions must return GRanges restricted to transcripts intersecting the region set (intersect of names(cds.tx0) and trans_ids), mirroring the getCDS example
- Results should be reproducible under a fixed seed and consistent with transcriptome-based, not genome-based, overlap counts (fewest overlaps on transcriptome backgrounds as reported in the paper)

## Limitations

- Feature sites lacking isoform association require the IA mode, which averages overlaps over K candidate isoforms per site and can reduce sensitivity
- No version number or dependency versions for RgnTX are stated; no R version specified
- Null distributions differ significantly between genome and transcriptome backgrounds (fewest overlaps on transcriptome space), so results are not comparable to genome-based colocalization methods

## Evidence

- [readme] permTestTxIA_customPick performs permutation test to evaluate association between genomic features (RS) that one only know its genomic positions and do not know which specific transcript: [evidence span withheld — anchor status: external_doc]
- [readme] cds.tx0 <- cdsBy(txdb, use.names=FALSE); cds.names <- as.character(intersect(names(cds.tx0), trans_ids)): "cds.tx0 <- cdsBy(txdb, use.names=FALSE)
  cds.names <- as.character(intersect(names(cds.tx0), trans_ids))"
- [abstract] It supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [abstract] easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [results] We used the annotation database from UCSC to extract the following backgrounds: "We used the annotation database from UCSC to extract the following backgrounds"
