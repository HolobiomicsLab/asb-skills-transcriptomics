---
name: feature-to-genome-coordinate-projection
description: Use when transcriptome elements (e.g. m6A sites, RNA-binding protein
  target sites from POSTAR3) are known only by genomic positions and the specific
  isoform they originate from is unclear.
when_to_use_negative:
- Features already have a known specific isoform/transcript association — use transcript-restricted
  randomization (randomizeTx / permTestTx) instead
- The analysis is purely genome-based on genomic region sets (e.g. ChIP-seq peaks
  with no transcriptome context) — conventional genome-based tools like regioneR or
  GenomeRunner are appropriate
- The goal is generic colocalization testing between two genomic region sets without
  isoform heterogeneity concerns
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: Provides randomizeTx, permTestTxIA_customPick and overlapCountsTxIA for isoform-ambiguity-aware
    projection and permutation of transcriptome features
  repo: https://bioconductor.org/packages/RgnTX
- name: R
  role: Runtime environment for the RgnTX package and the randomization/verification
    scripts
  repo: https://bioconductor.org/packages/RgnTX
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcriptome annotation supplying transcript/exon/CDS/stop codon
    structures used as the randomization background
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:62b04aad19aa7963d15e987104f1d657bc146d6f1752a2351860ab0b1afba40e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# feature-to-genome-coordinate-projection

## Summary

Projects transcriptome elements that lack a known isoform association (e.g. m6A sites from POSTAR3/miCLIP-seq) onto genomic coordinates and permutes them only over the transcripts that overlap the projected region, so RgnTX colocalization tests respect isoform ambiguity.

## When to use

Use when transcriptome elements (e.g. m6A sites, RNA-binding protein target sites from POSTAR3) are known only by genomic positions and the specific isoform they originate from is unclear.

## When NOT to use

- Features already have a known specific isoform/transcript association — use transcript-restricted randomization (randomizeTx / permTestTx) instead
- The analysis is purely genome-based on genomic region sets (e.g. ChIP-seq peaks with no transcriptome context) — conventional genome-based tools like regioneR or GenomeRunner are appropriate
- The goal is generic colocalization testing between two genomic region sets without isoform heterogeneity concerns

## Inputs

- Region set RS1 of transcriptome/genomic features without clear isoform association (e.g. m6A_sites_data.rds from RgnTX extdata)
- TxDb annotation object (TxDb.Hsapiens.UCSC.hg19.knownGene)
- customPick_function defining regions of interest (e.g. getStopCodon) per transcript

## Outputs

- Randomized region set restricted to overlapping isoform transcripts with original feature widths preserved
- Per-permutation random overlap counts (via overlapCountsTxIA) plus p-value and z-score from permTestTxIA_customPick
- Per-region verification table (feature, randomized region coordinates, associated transcripts, on-permitted-transcript flag), e.g. isoform_ambiguity_randomization_check.csv

## How to apply

Load RgnTX with TxDb.Hsapiens.UCSC.hg19.knownGene and set a fixed random seed (e.g. 12345677) for reproducibility. For each feature, project its transcriptome coordinates to the genome, then collect the K isoform transcripts that overlap the projected region. Restrict randomization to exactly those K transcripts, drawing a replacement region of the same width as the original feature on each candidate isoform (as done by permTestTxIA_customPick, which divides counts by K). Verify that each randomized region falls on its permitted transcripts and write the per-region check table (feature, randomized coordinates, associated transcripts, on-permitted-transcript flag). The rationale is that genome-based permutation over shared exons inflates overlaps differently than transcriptome-level permutation, since null distributions on DNA, exonic DNA, pre-mRNA and mRNA backgrounds are significantly different even when restricted to the same 300 genes.

## Related tools

- **RgnTX** (Provides randomizeTx, permTestTxIA_customPick and overlapCountsTxIA for isoform-ambiguity-aware projection and permutation of transcriptome features) — https://bioconductor.org/packages/RgnTX
- **R** (Runtime environment for the RgnTX package and the randomization/verification scripts) — https://bioconductor.org/packages/RgnTX
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcriptome annotation supplying transcript/exon/CDS/stop codon structures used as the randomization background)

## Examples

```
randomizeTx(txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, trans_ids = c("170","782","974","1364","1387"), random_num = 10, type = "mature", random_length = 100)
```

## Evaluation signals

- Every randomized region overlaps (is contained in) one of the K transcripts that also overlap the original projected feature; fraction on permitted transcripts should be 1.0
- Each randomized region has the same width as its original feature
- Random overlap counts are averaged over the K candidate isoforms (divided by K) as in overlapCountsTxIA
- Re-running with the same seed (e.g. 12345677) reproduces identical randomized coordinates
- Resulting p-values/z-scores from permTestTxIA_customPick differ from genome-based permutation results, consistent with significantly different null distributions across backgrounds

## Limitations

- Requires a TxDb-style transcript annotation; results depend on the chosen annotation (UCSC hg19 knownGene in the paper)
- Counts per feature must be normalized by the number of candidate isoforms K; features overlapping many isoforms dilute per-isoform evidence
- The skill addresses isoform ambiguity only for features whose genomic positions are known; it cannot recover the true originating isoform
- No version/dependency information is stated for RgnTX or R in the source materials

## Evidence

- [abstract] RgnTX supports testing transcriptome elements without clear isoform association: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [readme] README pseudocode for isoform-ambiguity permutation: [evidence span withheld — anchor status: external_doc]
- [intro] Existing approaches ignore transcriptome heterogeneity: "existing approaches have a fundamental limitation as they were primarily designed for genome-based analysis and do not account for the heterogeneity of the transcriptome"
- [discussion] RgnTX randomization within transcriptome background: "RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space."
