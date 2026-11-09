---
name: intronic-terminal-exon-classification
description: Use when you must identify previously unannotated intronic terminal exons
  / IPA sites from a standard RNA-seq BAM aligned to GRCh38 with RefSeq annotation,
  and need sample-specific classification of skipped vs composite candidates.
when_to_use_negative:
- Input is 3'-end sequencing, A-seq/3P-seq or long-read data — use dedicated benchmarking/comparison
  workflows rather than the read-module classifier
- Goal is only quantifying already-annotated 3'UTR APA events, which tools like DaPars
  or QAPA handle without novel terminal exon classification
- The sample lacks sufficient depth for the default read thresholds (5 spliced reads
  for skipped, 10 spanning reads for composite candidates)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: 'Implements the read module: builds candidate intronic terminal exons and
    trains/ensembles the random-forest classifiers (InPACT, InPACT_transcript, InPACT_quantify)'
  repo: https://github.com/YY-TMU/InPACT
- name: HISAT2
  role: Aligns RNA-seq reads to GRCh38 to produce the sorted BAM input
- name: SAMtools
  role: Filters out reads flagged as secondary alignment before feature computation
- name: featureCount
  role: Computes per-region RNA-seq alignment features used to train and evaluate
    the classifiers
- name: salmon
  role: Quantifies assembled transcript abundances so IPA usage can be computed after
    classification
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:bbd13361508897fec5e08886bb6e8b9edcfbdcef7f387d5a6589bff471a6f112
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - SAMtools
  - featureCount
  - salmon
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# intronic-terminal-exon-classification

## Summary

Trains sample-specific random-forest classifiers to distinguish true intronic terminal exons (skipped vs composite IPA candidates) from background in conventional RNA-seq, reproducing InPACT's read module with near-perfect AUROC (~0.997–0.999) on HEK293 replicates.

## When to use

Use when you must identify previously unannotated intronic terminal exons / IPA sites from a standard RNA-seq BAM aligned to GRCh38 with RefSeq annotation, and need sample-specific classification of skipped vs composite candidates.

## When NOT to use

- Input is 3'-end sequencing, A-seq/3P-seq or long-read data — use dedicated benchmarking/comparison workflows rather than the read-module classifier
- Goal is only quantifying already-annotated 3'UTR APA events, which tools like DaPars or QAPA handle without novel terminal exon classification
- The sample lacks sufficient depth for the default read thresholds (5 spliced reads for skipped, 10 spanning reads for composite candidates)

## Inputs

- Coordinate-sorted BAM of RNA-seq reads aligned to GRCh38 (secondary alignments removed)
- RefSeq GTF annotation (e.g. hg38.109.20211119.ncbiRefSeq.gtf)
- Candidate IPA site SAF regions (20nt upstream of potential polyA sites)
- Reference genome FASTA (for transcript assembly step)

## Outputs

- Trained skipped and composite random-forest ensembles (ten subsampled classifiers each)
- predict.result.txt with predicted terminal exons, IPA type (skipped/composite), gene, upstream coordinate and PolyAsite
- Per-replicate/class AUROC and classification metrics (TPR, TNR, accuracy, precision, F1, MCC)

## How to apply

Align the replicate RNA-seq reads to GRCh38 with HISAT2 and remove secondary alignments with SAMtools. Define three training classes from the RefSeq GTF: annotated last exons with at least five splice-in reads (terminal exons), internal exons between first and last with at least five splice-in reads, and annotated last exons with fewer than five splice-in reads (background), then split regions 80:20 into training/testing sets. Compute per-region alignment features with featureCount (relative region length, normalized expression, coefficients of variation, entropy efficiency, spliced/unspliced counts around 5' and 3' ends). Train separate random-forest ensembles over ten randomized subsamples for putative skipped terminal exons (>=5 uniquely mapped spliced reads with the 3' end in the candidate region) and composite terminal exons (>=10 uniquely mapped unspliced reads crossing the closest upstream splice site), then evaluate on the held-out 20% computing TPR, TNR, accuracy, precision, F1, MCC and AUROC.

## Related tools

- **InPACT** (Implements the read module: builds candidate intronic terminal exons and trains/ensembles the random-forest classifiers (InPACT, InPACT_transcript, InPACT_quantify)) — https://github.com/YY-TMU/InPACT
- **HISAT2** (Aligns RNA-seq reads to GRCh38 to produce the sorted BAM input)
- **SAMtools** (Filters out reads flagged as secondary alignment before feature computation)
- **featureCount** (Computes per-region RNA-seq alignment features used to train and evaluate the classifiers)
- **salmon** (Quantifies assembled transcript abundances so IPA usage can be computed after classification) — https://github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- Held-out (20%) AUROC per replicate and class must match auroc_results.csv expectations: rep1 skipped 0.999 / composite 0.997; rep2 skipped 0.998 / composite 0.998, within tolerance
- Classification metrics reported must include TPR, TNR, accuracy, precision, F1 score and Matthew's correlation coefficient, not AUROC alone
- Skipped candidates must satisfy >=5 uniquely mapped spliced reads with the 3' end in the region; composite candidates >=10 uniquely mapped unspliced reads crossing the closest upstream splice site (defaults in InPACT CLI)
- Terminal exon training class must contain annotated last exons with >=5 splice-in reads; background class annotated last exons with <5 splice-in reads
- predict.result.txt schema must contain columns: Terminal exon, IPA type (Skipped or composite), Gene, Upstream coordinate, PolyAsite, IPA usage

## Limitations

- IPAFinder-style coverage-fluctuation approaches need adequate sequencing depth; the classifier also relies on sufficient read support (default 5 spliced / 10 spanning reads)
- Classification is sample-specific: models are trained on alignment features from the RNA-seq data under analysis, so results depend on the replicate's depth and quality
- No version number is given for InPACT or the compared tools, complicating exact reproduction
- The test BAM takes about an hour to run with five cores; paired-end status must match the --paired/-p setting (default True)

## Evidence

- [results] read module operates by training separate models for skipped and composite terminal exon candidates: [evidence span withheld — anchor status: not_found]
- [methods] InPACT trains classifiers on ten randomized subsamples: "InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers."
- [readme] minimum spliced reads for skipped: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [methods] annotated unique last exons with at least five splice-in reads: "annotated unique last exons that have at least five splice-in reads"
- [methods] Reads flagged as secondary alignment were filtered out using SAMtools: "Reads flagged as secondary alignment were filtered out using SAMtools"
- [intro] InPACT can reproducibly and accurately identify IPA sites: "InPACT can reproducibly and accurately identify IPA sites and reconstruct IPA isoforms."
