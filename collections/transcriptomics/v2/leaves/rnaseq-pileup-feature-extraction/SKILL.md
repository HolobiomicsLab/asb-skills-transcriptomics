---
name: rnaseq-pileup-feature-extraction
description: Use when stranded RNAseq BAM files must be converted into per-position
  nucleotide frequency matrices (8 × 101) to feed a deep learning model for RNA editing
  site classification without coupled WGS/WES data.
when_to_use_negative:
- Sites with fewer than 50 RNAseq reads, fewer than 3 A-to-G substitutions, or AG
  substitution frequency below 0.01 — these are discarded at the first filtering step
  and cannot yield complete feature windows
- Sites whose 101-base window is not fully covered by RNAseq reads and imputation
  is not acceptable for the downstream analysis
- Analyses that already possess a REDItools outTable with per-base counts and only
  need statistical filtering (e.g. empirical WGS/WES-based filtering) rather than
  deep-learning feature extraction
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: REDItools
  role: Produces the RNAseq pileup outTable via the DNA-RNA pipeline at low stringency
    so complete 101-nt intervals can be retrieved for feature extraction
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster alternative REDItools implementation; completed pileup profiles in
    <60 minutes versus ~9 hours for REDItools v1
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: REDInet
  role: Consumes the tabix-indexed outTable, filters positions on coverage/AG thresholds,
    extracts the 8 × 101 nucleotide-frequency windows, and runs the TCN classifier
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: samtools
  role: faidx-indexes the downloaded reference genome FASTA so REDInet can retrieve
    reference nucleotides for the 101-base windows
- name: pysam
  role: Python wrapper over samtools/tabix C libraries used by REDInet to access BAM
    pileups and indexed outTables
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:a57859c571f1244526bdb5cba244858ebc5b61806d6708a5c5610e774af5aa61
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDItools
  tools:
  - Python
  - REDInet
  - REDItools
  - REDItools3
  - TensorFlow
  - numpy
  - pandas
  - pysam
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# rnaseq-pileup-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates RNAseq pileup nucleotide-frequency features from 101-base windows centered on candidate A-to-I editing positions using REDItools, as the input encoding for REDInet's temporal convolutional network classifier.

## When to use

Use when stranded RNAseq BAM files must be converted into per-position nucleotide frequency matrices (8 × 101) to feed a deep learning model for RNA editing site classification without coupled WGS/WES data.

## When NOT to use

- Sites with fewer than 50 RNAseq reads, fewer than 3 A-to-G substitutions, or AG substitution frequency below 0.01 — these are discarded at the first filtering step and cannot yield complete feature windows
- Sites whose 101-base window is not fully covered by RNAseq reads and imputation is not acceptable for the downstream analysis
- Analyses that already possess a REDItools outTable with per-base counts and only need statistical filtering (e.g. empirical WGS/WES-based filtering) rather than deep-learning feature extraction

## Inputs

- Sorted stranded RNAseq BAM file(s)
- faidx-indexed reference genome FASTA (GRCh37 or GRCh38 primary assembly from GENCODE)
- Tabix-indexed (bgzip-compressed) REDItools outTable of RNAseq pileup positions

## Outputs

- Per-site 101-nt feature matrices (8 × 101) of RNAseq nucleotide frequencies centered on each candidate editing position
- REDInet per-site predictions (softmax class 0 = SNP / 1 = editing site)
- Filtered-out position log (<.gz name>_discarded.txt)

## How to apply

Run the REDItools DNA-RNA pipeline (e.g. REDItoolDnaRna.py with low-stringency settings: -t 40, -c 0,1, -m 0,255, -v 1, -q 0,30, -n 0.0, -N 0.0, -S) over the RNAseq BAM and a faidx-indexed reference genome (GRCh37 or GRCh38) to produce an outTable of variant and invariant sites supported by RNAseq reads. bgzip-compress each outTable and build a tabix index (tabix -s 1 -b 2 -e 2 -c R). Launch REDInet_Inference.py on the indexed outTables with the matching assembly and default thresholds (minimum coverage 50, minimum 3 A-to-G substitutions, minimum AG substitution frequency 0.01) so it extracts 101-nt windows centered on each candidate site and encodes each interval as a matrix of 8 × 101 features. Sites failing coverage or missing nucleotides within the 101-base window are filtered out (recorded in the <.gz name>_discarded.txt file), optionally with imputation of missing values using the reference genome (BETA feature). The resulting feature matrices are passed directly to the TCN model, which outputs per-site softmax predictions (class 0 = SNP, class 1 = editing site).

## Related tools

- **REDItools** (Produces the RNAseq pileup outTable via the DNA-RNA pipeline at low stringency so complete 101-nt intervals can be retrieved for feature extraction) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster alternative REDItools implementation; completed pileup profiles in <60 minutes versus ~9 hours for REDItools v1) — https://github.com/BioinfoUNIBA/REDItools3
- **REDInet** (Consumes the tabix-indexed outTable, filters positions on coverage/AG thresholds, extracts the 8 × 101 nucleotide-frequency windows, and runs the TCN classifier) — https://github.com/BioinfoUNIBA/REDInet.git
- **samtools** (faidx-indexes the downloaded reference genome FASTA so REDInet can retrieve reference nucleotides for the 101-base windows)
- **pysam** (Python wrapper over samtools/tabix C libraries used by REDInet to access BAM pileups and indexed outTables) — https://github.com/pysam-developers/pysam

## Examples

```
python ../REDItools/main/REDItoolDnaRna.py -o ../REDInet/Package/Data -i SRR12492027.SRR12492028.Aligned.sortedByCoord.out.chr10.bam -f GRCh37.primary_assembly.genome.fa -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -e -n 0.0 -N 0.0 -u -l -p -s 2 -g 2 -S
```

## Evaluation signals

- Each outTable is bgzip-compressed and tabix-indexed without error (tabix -s 1 -b 2 -e 2 -c R); REDInet can query it positionally
- The <.gz name>_discarded.txt file correctly records every position failing minimum coverage (50), minimum A-to-G count (3), or minimum AG frequency (0.01)
- Extracted feature matrices have shape 8 × 101 with the investigated position centered at column 51
- Downstream REDInet predictions achieve the expected classification regime (accuracy >95% on HEK293T KO-WT ground truth, mean accuracy >96% on A549)
- Reference genome build used for window extraction (GRCh37 vs GRCh38) matches the assembly coordinate system of the outTable

## Limitations

- Sites missing nucleotides within the 101-nt window are filtered out unless the BETA imputation feature (using reference genome hg38 or hg19) is enabled
- REDItools v1 pileup is ~10x slower than the REDItools v3 re-implementation (<60 minutes vs ~9 hours) on large RNAseq datasets
- REDInet classifies only A-to-G substitutions as editing vs non-editing; other substitution types are out of scope
- Complete REDInet architecture and feature rescaling details are deferred to Supplementary Material not included in the main text

## Evidence
