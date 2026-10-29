---
name: skipped-composite-ipa-event-discrimination
description: Use when running InPACT's read module on coordinate-sorted BAM files
  to classify candidate intronic terminal exons as skipped or composite IPA events
  and predict novel IPA isoforms.
when_to_use_negative:
- When only 3'UTR-level APA quantification is needed — tools like DaPars, QAPA or
  LABRAT are designed for APA quantification, and this skill targets novel intronic
  (skipped/composite) IPA identification
- When input BAM has not had secondary alignments filtered with SAMtools and PCR duplicates
  removed with samtools rmdup, since read-module thresholds assume cleaned alignments
- When the goal is IPA site identification without distinguishing skipped from composite
  events — APAIQ-like approaches cannot differentiate event types, but InPACT's two-class
  design is the point of this skill, so using only one class would defeat it
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: InPACT
  role: 'Primary tool: trains ensembled random-forest classifiers for skipped and
    composite intronic terminal exon candidates from BAM alignment features'
  repo: https://github.com/YY-TMU/InPACT
- name: HISAT2
  role: Aligns HEK293 replicate RNA-seq reads to the GRCh38 genome to produce the
    sorted BAM input
- name: SAMtools
  role: Filters reads flagged as secondary alignment from the HISAT2 output before
    read-module classification
- name: featureCount
  role: Computes per-region RNA-seq alignment features (region length, normalized
    expression, CV, entropy efficiency, spliced/unspliced counts) used to train the
    classifiers
- name: Salmon
  role: Quantifies inferred IPA transcript abundances downstream so IPA usage (PAU)
    can be computed per site
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:61f1fa6706e010d35957b1cd8b8ff2b90414b545481162fde56511fc669b8036
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - SAMtools
  - Salmon
  - featureCount
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# skipped-composite-ipa-event-discrimination

## Summary

Train and apply separate random-forest read-module classifiers for skipped and composite intronic terminal exon candidates in InPACT, enabling sample-specific identification of novel IPA sites from conventional RNA-seq alignments.

## When to use

Use when running InPACT's read module on coordinate-sorted BAM files to classify candidate intronic terminal exons as skipped or composite IPA events and predict novel IPA isoforms.

## When NOT to use

- When only 3'UTR-level APA quantification is needed — tools like DaPars, QAPA or LABRAT are designed for APA quantification, and this skill targets novel intronic (skipped/composite) IPA identification
- When input BAM has not had secondary alignments filtered with SAMtools and PCR duplicates removed with samtools rmdup, since read-module thresholds assume cleaned alignments
- When the goal is IPA site identification without distinguishing skipped from composite events — APAIQ-like approaches cannot differentiate event types, but InPACT's two-class design is the point of this skill, so using only one class would defeat it

## Inputs

- Coordinate-sorted BAM file of RNA-seq alignments (secondary alignments removed with SAMtools)
- RefSeq gene annotation GTF (e.g. hg38.109.20211119.ncbiRefSeq.gtf from UCSC goldenPath)
- InPACT polyA candidate sites SAF (InPACT_polyAsites.hg38.saf) spanning 20 nt upstream of potential polyA sites

## Outputs

- predict.result.txt with per-site terminal exon, IPA type (Skipped or composite), gene, upstream coordinate, PolyAsite, and IPA usage columns
- Per-replicate classification metrics (TPR, TNR, accuracy, precision, F1, MCC, AUROC) for skipped and composite ensembles (auroc_results.csv)
- Ensembled random-forest classifiers for skipped and composite terminal exon classes

## How to apply

Align RNA-seq reads to GRCh38 with HISAT2, remove secondary alignments with SAMtools, and provide the sorted BAM plus the RefSeq GTF (hg38.ncbiRefSeq.gtf) and the InPACT polyA site SAF to the InPACT command. Train three classes from RefSeq annotation — terminal exons with ≥5 splice-in reads, internal exons between first and last exons, and background last exons with <5 splice-in reads — split 80:20 into training/testing sets, and compute alignment features per region with featureCount (relative length, normalized expression, coefficients of variation, entropy efficiency, spliced/unspliced reads at 5' and 3' ends). Train separate random-forest ensembles over ten randomized subsamples: for skipped candidates require ≥5 uniquely mapped spliced reads with the 3' end in the candidate region, and for composite candidates require ≥10 uniquely mapped unspliced reads crossing the closest upstream splice site. Evaluate each ensemble on the held-out 20% testing set, computing AUROC, accuracy, precision, F1, MCC, TPR and TNR; in HEK293 replicates this should yield AUROCs of ~0.997–0.999 per replicate and class.

## Related tools

- **InPACT** (Primary tool: trains ensembled random-forest classifiers for skipped and composite intronic terminal exon candidates from BAM alignment features) — https://github.com/YY-TMU/InPACT
- **HISAT2** (Aligns HEK293 replicate RNA-seq reads to the GRCh38 genome to produce the sorted BAM input)
- **SAMtools** (Filters reads flagged as secondary alignment from the HISAT2 output before read-module classification)
- **featureCount** (Computes per-region RNA-seq alignment features (region length, normalized expression, CV, entropy efficiency, spliced/unspliced counts) used to train the classifiers)
- **Salmon** (Quantifies inferred IPA transcript abundances downstream so IPA usage (PAU) can be computed per site) — https://github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf --minimum_spliced_reads_for_skipped 5 --minimum_span_reads_for_composite 10 -P 5
```

## Evaluation signals

- Per-replicate AUROC for each class must match expected values: rep1 skipped 0.999 / composite 0.997; rep2 skipped 0.998 / composite 0.998, within a small tolerance
- Every skipped candidate is supported by ≥5 uniquely mapped spliced reads with the 3' end inside the candidate region; every composite candidate by ≥10 uniquely mapped unspliced reads crossing the closest upstream splice site
- predict.result.txt contains the expected columns: Terminal exon, IPA type (Skipped or composite), Gene, Upstream coordinate, PolyAsite, IPA usage
- Training class definitions hold: terminal-exon class has ≥5 splice-in reads, background class has <5 splice-in reads, and the 80:20 train/test split is reproducibly randomized
- Ensemble metrics (TPR, TNR, accuracy, precision, F1, MCC) are computed on the 20% held-out testing set and reported per replicate per class

## Limitations

- Existing APA quantification tools (MISO, QAPA, LABRAT) lack this capability, but conversely the read module requires adequate sequencing depth, similar to the coverage-dependence criticized in IPAFinder
- No version number is given for InPACT or compared tools, complicating reproducibility of classifier performance
- Thresholds are defaults (5 spliced reads for skipped, 10 spanning reads for composite) and may need adjustment for shallow or single-cell libraries, e.g. SMART-seq2 data

## Evidence

- [methods] read module classifier training: "InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers."
- [readme] skipped threshold: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [readme] composite threshold: "Minimum number of reads that cross the boundary required to consider the potential composite terminal exon. [default=10]"
- [intro] existing tools limitation: "Although several computational tools, including MISO, QAPA, and LABRAT, enable APA quantification, they do not possess the capability to identify novel IPA isoforms"
- [readme] InPACT command: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
