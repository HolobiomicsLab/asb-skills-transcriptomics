---
name: pileup-nucleotide-frequency-normalization
description: Use when tabix-indexed REDItools outTables must be converted into per-site
  8×101 feature matrices of reference-base one-hot and log2-normalized RNAseq pileup
  frequencies to feed REDInet or an equivalent TCN classifier.
when_to_use_negative:
- Input is already a feature vector TSV or a model-ready 8×101 matrix — re-running
  encoding would double-transform the pileup frequencies
- BAM files have not yet been processed into a REDItools outTable via REDItoolDnaRna.py
  or REDItools3 — the skill operates on REDItools pileup tables, not raw alignments
- Sites lacking RNAseq support (coverage below the minimum, e.g. <50 reads) — encoding
  incomplete windows violates REDInet's requirement for fully covered 101-base intervals
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: REDInet
  role: Consumer of the encoded 8×101 feature matrices; its inference scripts (REDInet_Inference.py,
    REDInet_Inference_light_ver.py) perform filtering and encoding of tabix-indexed
    REDItools outTables
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: REDItools
  role: Generates the REDItools outTable (variant and invariant positions supported
    by RNAseq reads) that is the input to the encoding step
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: pysam
  role: Python library used to read the tabix-indexed outTable and the reference genome
    during window extraction and pileup normalization
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:523e302c42c615c460e6917bbb4584959410880fabf1e799cc2a7db8a3019340
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet.git
  tools:
  - Python
  - REDInet
  - REDItools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# pileup-nucleotide-frequency-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes 101-base RNAseq windows centered on putative A-to-I editing sites as 8×101 feature matrices — 4 one-hot channels for the reference base (A, C, G, T) plus 4 channels of log2-transformed normalized RNAseq nucleotide pileup frequencies with an alpha=0.0001 pseudocount — for input to the REDInet temporal convolutional network.

## When to use

Use when tabix-indexed REDItools outTables must be converted into per-site 8×101 feature matrices of reference-base one-hot and log2-normalized RNAseq pileup frequencies to feed REDInet or an equivalent TCN classifier.

## When NOT to use

- Input is already a feature vector TSV or a model-ready 8×101 matrix — re-running encoding would double-transform the pileup frequencies
- BAM files have not yet been processed into a REDItools outTable via REDItoolDnaRna.py or REDItools3 — the skill operates on REDItools pileup tables, not raw alignments
- Sites lacking RNAseq support (coverage below the minimum, e.g. <50 reads) — encoding incomplete windows violates REDInet's requirement for fully covered 101-base intervals

## Inputs

- tabix-indexed, bgzip-compressed REDItools outTable (.gz, indexed with 'tabix -s 1 -b 2 -e 2 -c R')
- GRCh37 or GRCh38 reference genome FASTA, faidx-indexed (e.g. GRCh37.primary_assembly.genome.fa)
- Python environment with pysam, pandas, numpy (REDInet env: pysam==0.22.1, pandas==2.2.2, numpy==1.26.4)

## Outputs

- <output_files_prefix>.feature_vectors.tsv — one 8×101 encoded feature matrix per site with a complete 101-nt interval
- <prefix>_discarded.txt — positions failing coverage/substitution filters
- <prefix>_incompleates.txt — intervals dropped for missing or strand-discordant nucleotides ('0 *' / '0 **')

## How to apply

Load the tabix-indexed, bgzip-compressed REDItools outTable (indexed with 'tabix -s 1 -b 2 -e 2 -c R') using Python with pysam/pandas, then filter candidate sites to REDInet defaults: minimum RNAseq coverage of 50 (--C), minimum of 3 guanosines substituting adenines (--M), and minimum A-to-G substitution frequency of 0.01 (--F), logging excluded positions in a <prefix>_discarded.txt file. For each retained site, extract the 101-nucleotide window (50 bases flanking each side, centered on the putative position) from the GRCh37 or GRCh38 reference genome, discarding intervals with missing or strand-discordant nucleotides (logged as '0 *' / '0 **' in <prefix>_incompleates.txt). Encode each complete window as an 8×101 matrix: rows 1–4 are one-hot channels for the reference base A, C, G, T, and rows 5–8 hold log2-transformed normalized RNAseq nucleotide pileup frequencies computed with an alpha=0.0001 pseudocount. Write the feature vectors of sites with complete intervals to <output_files_prefix>.feature_vectors.tsv. The rescaling/normalization rationale is that REDInet reads the 101-base window like an audio signal, so RNAseq nucleotide frequencies (not just the reference sequence) carry the discriminative signal for A-to-I editing classification.

## Related tools

- **REDInet** (Consumer of the encoded 8×101 feature matrices; its inference scripts (REDInet_Inference.py, REDInet_Inference_light_ver.py) perform filtering and encoding of tabix-indexed REDItools outTables) — https://github.com/BioinfoUNIBA/REDInet.git
- **REDItools** (Generates the REDItools outTable (variant and invariant positions supported by RNAseq reads) that is the input to the encoding step) — https://github.com/BioinfoUNIBA/REDItools
- **pysam** (Python library used to read the tabix-indexed outTable and the reference genome during window extraction and pileup normalization) — https://github.com/pysam-developers/pysam

## Examples

```
python3 ../REDInet/Package/Utilities/REDInet_Inference_light_ver.py -r outTable_SRR12492027.gz -o results/prefix -c 50 -s 3 -f 0.01
```

## Evaluation signals

- Output TSV <prefix>.feature_vectors.tsv exists and every feature vector corresponds to a complete 101-nt interval rendered as an 8×101 matrix
- Only sites passing the REDInet defaults (RNAseq coverage ≥50, ≥3 G substituting A, A-to-G frequency ≥0.01) appear in the feature vectors
- Positions failing filters are exhaustively recorded in <prefix>_discarded.txt; intervals with missing/strand-discordant bases are logged as '0 *' / '0 **' in <prefix>_incompleates.txt
- Normalized pileup channels are log2-transformed with the alpha=0.0001 pseudocount, so no channel value is undefined or -inf
- One-hot channels sum to 1 per position (single reference base per column), consistent with the GRCh37/GRCh38 reference base

## Limitations

- Intervals with missing or strand-discordant nucleotides cannot be encoded and are dropped (logged as '0 *' / '0 **'), so heavily uncovered loci yield no predictions
- The multiprocessing REDInet_Inference.py imputation of missing values (hg38/hg19) is a BETA feature; the light version is recommended for HTC-cluster parallelization
- Feature rescaling details are deferred to the paper's Supplementary Material, which is not included in the available text
- The encoding is specific to A-to-I/A-to-G editing detection on human GRCh37/GRCh38 assemblies with REDItools-format outTables

## Evidence
