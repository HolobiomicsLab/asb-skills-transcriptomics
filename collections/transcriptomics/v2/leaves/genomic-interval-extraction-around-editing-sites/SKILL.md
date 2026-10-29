---
name: genomic-interval-extraction-around-editing-sites
description: Use when you have REDItools outTables of candidate A-to-G substitutions
  and need fixed-length 101-nt centered sequence contexts (with per-position nucleotide
  frequencies) to feed a deep-learning editing classifier such as REDInet.
when_to_use_negative:
- Your downstream tool does not require fixed-length centered windows (e.g. REDItools
  v3 alone for empirical calling) — window extraction is specific to REDInet-style
  classifiers
- 'Input is raw FASTQ or unindexed BAM: REDItools calling and tabix indexing must
  precede interval extraction'
- The 101-base window is not fully covered by RNAseq reads and you cannot use (or
  trust) the BETA genome-based imputation — predictions will be unreliable
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0622
tools:
- name: REDInet
  role: Filters tabix-indexed outTables, extracts the 101-nt centered intervals, encodes
    them as 8 x 101 feature matrices, and classifies them with the pretrained REDInet.h5
    TCN model
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: REDItools
  role: Produces the outTables of candidate A-to-G substitutions from RNAseq BAMs
    at low-stringency settings (-q 0,30, -c 0,1, -v 1, -n 0.0, -N 0.0) so complete
    intervals are retrieved
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: pysam
  role: Python interface to SAM/BAM and tabix used by REDInet for efficient access
    to indexed outTables and genome sequence
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:d2fa77ffe4f38e3bf5d1e6e78a2203abdaf866921c150347641c175ebe247874
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet
  tools:
  - Python
  - REDInet
  - REDItools
  - pysam
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# genomic-interval-extraction-around-editing-sites

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract 101-nucleotide RNAseq windows centered on candidate A-to-G positions from tabix-indexed REDItools outTables, encoding each interval as an 8 x 101 feature matrix for REDInet's temporal convolutional network. This windowing is what converts pileup substitutions into fixed-length sequence contexts the classifier can consume.

## When to use

Use when you have REDItools outTables of candidate A-to-G substitutions and need fixed-length 101-nt centered sequence contexts (with per-position nucleotide frequencies) to feed a deep-learning editing classifier such as REDInet.

## When NOT to use

- Your downstream tool does not require fixed-length centered windows (e.g. REDItools v3 alone for empirical calling) — window extraction is specific to REDInet-style classifiers
- Input is raw FASTQ or unindexed BAM: REDItools calling and tabix indexing must precede interval extraction
- The 101-base window is not fully covered by RNAseq reads and you cannot use (or trust) the BETA genome-based imputation — predictions will be unreliable

## Inputs

- bgzip-compressed, tabix-indexed REDItools outTable of candidate A-to-G substitutions (columns 1-2 = chromosome, position)
- Stranded paired-end RNAseq BAM file (e.g. from SRA) already processed by REDItools
- Fasta-indexed reference genome (GRCh37 or GRCh38 primary assembly, e.g. from GENCODE release 46) for interval extraction and optional missing-value imputation

## Outputs

- Per-site 8 x 101 feature matrices of RNAseq nucleotide frequencies in 101-nt windows centered at each candidate position
- REDInet *.predictions.tsv with per-site y_hat calls (0 = SNP, 1 = editing site)
- <outTable name>_discarded.txt listing positions filtered out by coverage/substitution thresholds

## How to apply

First call candidate sites with REDItools (e.g. REDItoolDnaRna.py at low stringency: -q 0,30, -c 0,1, -v 1, -n 0.0, -N 0.0) so the full flanking interval is retrieved, then compress each outTable with bgzip and index it with 'tabix -s 1 -b 2 -e 2 -c R', the format REDInet requires for random access by chromosome/position. Launch REDInet_Inference.py (or REDInet_Inference_light_ver.py) with the correct genome build (GRCh37 or GRCh38), which filters positions by minimum coverage (default 50 reads), minimum 3 G substitutions, and minimum A-to-G frequency 0.01, then extracts the 101-nt interval centered on each surviving site. Each interval is encoded as a matrix of 8 x 101 features (RNAseq nucleotide frequencies plus reference-base information) and passed to the pretrained REDInet.h5 TCN model. Discard positions whose 101-base window is not fully covered by RNAseq reads, or use the README's imputation option (missing values filled from the reference genome), which is a BETA feature. The rationale is that local sequence context around the adenosine, not coupled WGS/WES data, carries the signal the network needs to distinguish editing from SNPs.

## Related tools

- **REDInet** (Filters tabix-indexed outTables, extracts the 101-nt centered intervals, encodes them as 8 x 101 feature matrices, and classifies them with the pretrained REDInet.h5 TCN model) — https://github.com/BioinfoUNIBA/REDInet
- **REDItools** (Produces the outTables of candidate A-to-G substitutions from RNAseq BAMs at low-stringency settings (-q 0,30, -c 0,1, -v 1, -n 0.0, -N 0.0) so complete intervals are retrieved) — https://github.com/BioinfoUNIBA/REDItools
- **pysam** (Python interface to SAM/BAM and tabix used by REDInet for efficient access to indexed outTables and genome sequence) — https://github.com/pysam-developers/pysam

## Examples

```
bgzip outTable_1.gz.in && tabix -s 1 -b 2 -e 2 -c R outTable_1.gz; python3 REDInet_Inference.py --I Data/ --O results/ --A GRCh37 --C 50 --M 3 --N outTable_1.gz
```

## Evaluation signals

- Every input outTable is bgzipped and accepts 'tabix -s 1 -b 2 -e 2 -c R' indexing without error; the _discarded.txt file reports positions removed by the MinCov=50 / MinAG=3 / freq 0.01 filters
- Extracted intervals are exactly 101 nucleotides centered on the candidate position; sites with any uncovered window base are either absent from predictions or imputed from the reference genome
- Feature matrices have the expected 8 x 101 shape with rescaled RNAseq counts before feeding the TCN
- Per-site y_hat calls in *.predictions.tsv are restricted to {0, 1}, and on the A549 ground truth set (>13,000 sites) mean accuracy reaches >96%

## Limitations

- Only sites whose 101-base window is fully covered by RNAseq reads were used to build the paper's ground truth datasets; partial windows require the imputation feature, which the README flags as BETA
- The complete REDInet architecture and feature rescaling details are deferred to Supplementary Material not included in the main text
- Requires separate genome builds (GRCh37 vs GRCh38) to match the outTable coordinates; mismatched assembly will extract wrong intervals

## Evidence
