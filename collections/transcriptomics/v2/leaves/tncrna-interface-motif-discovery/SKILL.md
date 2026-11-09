---
name: tncrna-interface-motif-discovery
description: Use when you have classified per-sample tncRNA catalogs (tRF-5, tRF-3,
  tRF-1, 5'tRH, 3'tRH) plus progenitor mature tRNA sequences with 50 bp flanks, and
  the goal is to distinguish specific endoribonucleolytic cleavage from random tRNA
  degradation and recover a conserved cleavage motif.
when_to_use_negative:
- The goal is de novo tncRNA detection from raw FASTQ files (use the tncRNA Toolkit
  prediction pipeline instead — this skill operates on already-classified tncRNA catalogs)
- Analyzing non-plant or single-species datasets where the angiosperm-wide conservation
  of the interface motif cannot be assessed
- Random-degradation behavior is expected a priori (e.g. degraded total RNA libraries)
  — the codon-dependent non-uniformity test would not be interpretable
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: tncRNA Toolkit
  role: Source of the per-sample tncRNA catalogs (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH)
    and progenitor mature tRNA sequences with 50 bp flanks used for cleavage-position
    composition and motif extraction
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Biopython
  role: pairwise2 module used to score local alignments of tRF-5::tRNA interface sequences
    (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5)
- name: pandas
  role: Used to compile the cleavage-position nucleotide composition table and the
    conserved interface motif table
- name: Bowtie
  role: bowtie1 ('-v2 --best', and '--norc -v 0 -m 50' for multi-mapping control)
    produced the unique tncRNA mappings that define cleavage positions relative to
    the progenitor tRNA
- name: HAMR
  role: Detected modified nucleosides on tncRNAs, which the article notes may influence
    tRNA cleavage (context for motif interpretation)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:dd3bb9b2c9444002f56a7baef6ac365287afab7455ef7e22690b45efe8eb1229
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tncrna-interface-motif-discovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for testing whether plant tncRNA generation is codon-dependent and whether tRF-5 cleavage at the tRF-5::tRNA interface is governed by a conserved nucleotide motif across angiosperms. It quantifies nucleotide composition at tRNA cleavage positions and aligns tRF-5::tRNA interface sequences to extract the shared cleavage motif (AG::TGG).

## When to use

Use when you have classified per-sample tncRNA catalogs (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) plus progenitor mature tRNA sequences with 50 bp flanks, and the goal is to distinguish specific endoribonucleolytic cleavage from random tRNA degradation and recover a conserved cleavage motif.

## When NOT to use

- The goal is de novo tncRNA detection from raw FASTQ files (use the tncRNA Toolkit prediction pipeline instead — this skill operates on already-classified tncRNA catalogs)
- Analyzing non-plant or single-species datasets where the angiosperm-wide conservation of the interface motif cannot be assessed
- Random-degradation behavior is expected a priori (e.g. degraded total RNA libraries) — the codon-dependent non-uniformity test would not be interpretable

## Inputs

- Per-sample tncRNA catalog with classified reads for tRF-5, tRF-3, tRF-1, 5'tRH, and 3'tRH (tncRNA Toolkit output)
- Progenitor mature tRNA sequences with 50 bp upstream/downstream flanks
- sRNA-seq-derived tncRNA alignments (bowtie1 '-v2 --best', reads mapping exclusively to artificial chromosomes)

## Outputs

- Cleavage-position nucleotide composition table (per tncRNA class, 5' and 3' cleavage sites)
- Anticodon-grouped codon-dependency statistics versus random-degradation null
- Conserved tRF-5::tRNA interface motif table across angiosperm species (expected motif: AG::TGG for 15–19 nt tRF-5s)

## How to apply

Load the per-sample tncRNA catalog from the tncRNA Toolkit output together with progenitor mature tRNA sequences plus 50 bp flanks. For each tncRNA class, tally nucleotide frequencies at the 5' and 3' cleavage positions relative to the progenitor tRNA locus, then group counts by tRNA anticodon and test non-uniformity against a random-degradation null expectation to demonstrate codon-dependent generation. Extract tRF-5::tRNA interface sequences (focus on tRF-5s of 15–19 nt) and score local alignments with the Biopython pairwise2 module using identical=1, non-identical=-1, gap-open=-1, gap-extend=-0.5. Compile the conserved interface motif across species and write the cleavage-position composition table and motif table. Accept the reproduction when the composition table shows non-random, codon/anticodon-dependent nucleotide bias consistent with the paper's conserved AG::TGG motif.

## Related tools

- **tncRNA Toolkit** (Source of the per-sample tncRNA catalogs (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) and progenitor mature tRNA sequences with 50 bp flanks used for cleavage-position composition and motif extraction) — https://github.com/skbinfo/tncRNA-Toolkit
- **Biopython** (pairwise2 module used to score local alignments of tRF-5::tRNA interface sequences (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5))
- **pandas** (Used to compile the cleavage-position nucleotide composition table and the conserved interface motif table)
- **Bowtie** (bowtie1 ('-v2 --best', and '--norc -v 0 -m 50' for multi-mapping control) produced the unique tncRNA mappings that define cleavage positions relative to the progenitor tRNA)
- **HAMR** (Detected modified nucleosides on tncRNAs, which the article notes may influence tRNA cleavage (context for motif interpretation))

## Examples

```
python3 tncRNAs.py -s <species name> -i <processed small RNA reads>.fastq -o <output dir>
```

## Evaluation signals

- Composition table shows non-random, codon/anticodon-dependent nucleotide bias at tncRNA cleavage sites, rejecting the random-degradation null
- tRF-5::tRNA interface alignments across all six angiosperms converge on the conserved motif AG::TGG
- Motif is observed specifically in tRF-5s of 15–19 nt; outside this size range the motif should not be expected
- Cleavage positions map precisely to the tRF-5::tRNA boundary in the progenitor tRNA sequences (50 nt flanks confirm the locus context)
- Outputs conform to the expected schema: one cleavage-position composition table and one motif table per analysis

## Limitations

- Allowing even a single mismatch during read alignment to overcome sequencing errors may cause base misidentification and inflation of false negatives, biasing cleavage-position nucleotide counts
- The detailed mechanisms by which tRNA post-transcriptional modifications regulate cleavage remain unclear, so motif interpretation is correlational
- The existence of isodecoders (tRNAs with the same anticodon) complicates attribution of codon-dependence to a single progenitor tRNA

## Evidence
