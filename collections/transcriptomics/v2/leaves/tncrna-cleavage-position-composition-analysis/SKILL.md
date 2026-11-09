---
name: tncrna-cleavage-position-composition-analysis
description: Use when you have classified tncRNA reads (tRF-5, tRF-3, tRF-1, 5'tRH,
  3'tRH) mapped to tRNA loci plus flanks and want to test whether cleavage is precise/codon-dependent
  rather than random tRNA degradation, or to find a conserved tRF-5::tRNA interface
  motif.
when_to_use_negative:
- The goal is general tncRNA identification/annotation from raw sRNA-seq FASTQ rather
  than post-classification cleavage-position analysis — run the tncRNA Toolkit mapping/classification
  steps first
- The input reads are not uniquely attributable to a single tRNA locus (ambiguous
  multi-mapped reads were discarded if aligned more than 50 times, so unfiltered alignments
  would confound cleavage-position tallies)
- The analysis targets miRNA/piRNA/rRNA-derived small RNAs (e.g. via SPORTS1.0) rather
  than tRNA-derived fragments
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: tncRNA Toolkit
  role: Produces the classified tncRNA catalog (tRF-5s, tRF-3s, tRF-1, tRNA halves)
    and masked artificial genome (tRNA plus 50 nt leader/trailer) that this cleavage-position
    analysis operates on
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Biopython
  role: pairwise2 module used to score local alignments of tRF-5::tRNA interface sequences
    (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5)
- name: pandas
  role: Used to compile the cleavage-position composition tables and conserved motif
    tables from the per-sample tncRNA tallies
- name: Bowtie
  role: Aligns processed reads to the artificial genome index with '-v2 --best' options
    for error-tolerant, uniquely mappable reads underlying cleavage-position counts
- name: HAMR
  role: Predicts modified nucleosides on tncRNAs, whose conservation may influence
    tRNA cleavage position interpretation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:34cf27ef6d4a16a44f4ab2c75a4d5587eb2dd5b7fd1704801371e5b09f8a660c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Biopython
  - Bowtie
  - HAMR
  - pandas
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tncrna-cleavage-position-composition-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies nucleotide composition at tncRNA cleavage positions and groups counts by anticodon to test codon-dependent, non-random tRNA cleavage, then scores tRF-5::tRNA interface alignments to compile conserved cleavage motifs (e.g. AG::TGG) across angiosperms.

## When to use

Use when you have classified tncRNA reads (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) mapped to tRNA loci plus flanks and want to test whether cleavage is precise/codon-dependent rather than random tRNA degradation, or to find a conserved tRF-5::tRNA interface motif.

## When NOT to use

- The goal is general tncRNA identification/annotation from raw sRNA-seq FASTQ rather than post-classification cleavage-position analysis — run the tncRNA Toolkit mapping/classification steps first
- The input reads are not uniquely attributable to a single tRNA locus (ambiguous multi-mapped reads were discarded if aligned more than 50 times, so unfiltered alignments would confound cleavage-position tallies)
- The analysis targets miRNA/piRNA/rRNA-derived small RNAs (e.g. via SPORTS1.0) rather than tRNA-derived fragments

## Inputs

- Per-sample tncRNA catalog of classified reads (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) from tncRNA Toolkit
- Progenitor mature tRNA sequences with 50 bp up- and downstream flanks (artificial chromosome FASTA from tRNAscan-SE predictions)
- Processed sRNA-seq FASTQ (quality-filtered, adapter-trimmed via Trim Galore) mapped with bowtie '-v2 --best' to the artificial genome

## Outputs

- Nucleotide composition table at tncRNA 5' and 3' cleavage positions per tncRNA class
- Anticodon-grouped cleavage counts versus random-degradation null expectation
- Conserved tRF-5::tRNA interface motif table across the six angiosperms (e.g. AG::TGG for 15–19 nt tRF-5s)

## How to apply

Load the per-sample tncRNA catalog (classified tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH reads) from the tncRNA Toolkit output together with the progenitor mature tRNA sequences and their 50 nt flanks. For each tncRNA class, tally nucleotide frequencies at the 5' and 3' cleavage positions relative to the progenitor tRNA locus, and group the cleavage counts by tRNA anticodon to quantify non-uniformity against a random-degradation null expectation. Extract tRF-5::tRNA interface sequences and score local alignments with Biopython's pairwise2 module (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5), restricting the conserved-motif search to tRF-5s of 15–19 nt at the tRF-5::tRNA interface. Compile the cleavage-position composition table and the cross-species motif table; accept the reproduction when composition shows non-random, codon/anticodon-dependent nucleotide bias consistent with the paper's finding (e.g. the conserved AG::TGG motif in all six angiosperms).

## Related tools

- **tncRNA Toolkit** (Produces the classified tncRNA catalog (tRF-5s, tRF-3s, tRF-1, tRNA halves) and masked artificial genome (tRNA plus 50 nt leader/trailer) that this cleavage-position analysis operates on) — https://github.com/skbinfo/tncRNA-Toolkit
- **Biopython** (pairwise2 module used to score local alignments of tRF-5::tRNA interface sequences (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5))
- **pandas** (Used to compile the cleavage-position composition tables and conserved motif tables from the per-sample tncRNA tallies)
- **Bowtie** (Aligns processed reads to the artificial genome index with '-v2 --best' options for error-tolerant, uniquely mappable reads underlying cleavage-position counts)
- **HAMR** (Predicts modified nucleosides on tncRNAs, whose conservation may influence tRNA cleavage position interpretation)

## Examples

```
python3 tncRNAs.py -s <species_name> -i <processed_smallRNA.fastq> -o <output_dir>
```

## Evaluation signals

- Cleavage-position composition table shows non-random, codon/anticodon-dependent nucleotide bias rather than uniform degradation background
- A conserved interface motif (AG::TGG in the paper) is recovered at the tRF-5::tRNA interface in all six angiosperms for tRF-5s of 15–19 nt
- tncRNA sizes fall within the expected 14–50 nt range reported for the 2,448-sample, six-angiosperm dataset
- pairwise2 scoring used the documented parameters (identical=1, non-identical=-1, gap-open=-1, gap-extend=-0.5)
- Classification counts derive only from reads mapped exclusively to artificial chromosomes (tRNA plus 50 nt flanks)

## Limitations

- Allowing a single mismatch during mapping to overcome sequencing errors may lead to base misidentification and inflation of false negatives, biasing cleavage-position nucleotide calls
- tRNAscan-SE predicts a huge number of tRNA gene copies including pseudogenes; only high-quality tRNAs with score equal to or above 50 were retained, so incompletely filtered pools distort anticodon grouping
- Conventional RNA-seq (rather than tRNA-seq) suffers from tRNA modification biases that affect tncRNA identification and hence cleavage-position composition
- How tRNA post-transcriptional modifications affect regulation of tRNA cleavage remains unclear, so composition signals cannot be causally attributed

## Evidence
