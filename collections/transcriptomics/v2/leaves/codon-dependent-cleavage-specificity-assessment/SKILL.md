---
name: codon-dependent-cleavage-specificity-assessment
description: Use when sRNA-seq-derived tncRNA catalogs plus progenitor mature tRNA
  sequences with 50 bp flanks are available and the goal is to test whether cleavage
  sites show non-random, codon-dependent nucleotide bias and a conserved interface
  motif.
when_to_use_negative:
- When the input is raw sRNA-seq FASTQ that has not yet been adapter-trimmed (Trim
  Galore v0.6.6) and classified by the tncRNA Toolkit pipeline — run detection/classification
  first
- When the goal is small-RNA discovery outside tRNA-derived fragments (e.g. miRNA/piRNA/rRNA
  profiling, for which SPORTS1.0 is suggested) rather than assessing tRNA cleavage
  specificity
- When tncRNA reads cannot be mapped exclusively to the artificial chromosomes (tRNA
  plus 50 nt leader/trailer), since ambiguous non-tRNA reads invalidate the cleavage-position
  composition analysis
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: tncRNA Toolkit
  role: Produces the per-sample classified tncRNA catalogs and artificial-genome alignments
    that feed the cleavage-position and codon-dependence analyses
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Biopython
  role: pairwise2 module used to score local alignments of tRF-5::tRNA interface sequences
    (identical = 1, non-identical = -1, gap-open = -1, gap-extend = -0.5) to compile
    the conserved motif
- name: pandas
  role: Tabulation of nucleotide frequencies at cleavage positions and grouping of
    counts by anticodon (v1.1.0 dependency of the toolkit)
- name: HAMR
  role: Predicts modified nucleosides on tncRNAs; conserved modifications may influence
    tRNA cleavage and inform motif interpretation
- name: Bowtie
  role: bowtie1 alignment of processed reads to the artificial genome index ('-v2
    --best'; '--norc -v 0 -m 50' to discard reads mapping >50 times) supplying uniquely
    mapped tncRNAs for composition analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:44a0ea16789575fd01d7cc13980b06257dcce85b80abc69d4a5d899f2f683877
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# codon-dependent-cleavage-specificity-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses whether plant tncRNA (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) generation is codon/anticodon-dependent and site-specific rather than random tRNA degradation, by tallying nucleotide composition at cleavage positions and extracting conserved tRF-5::tRNA interface motifs (e.g. AG::TGG in 15–19 nt tRF-5s) across angiosperms.

## When to use

Use when sRNA-seq-derived tncRNA catalogs plus progenitor mature tRNA sequences with 50 bp flanks are available and the goal is to test whether cleavage sites show non-random, codon-dependent nucleotide bias and a conserved interface motif.

## When NOT to use

- When the input is raw sRNA-seq FASTQ that has not yet been adapter-trimmed (Trim Galore v0.6.6) and classified by the tncRNA Toolkit pipeline — run detection/classification first
- When the goal is small-RNA discovery outside tRNA-derived fragments (e.g. miRNA/piRNA/rRNA profiling, for which SPORTS1.0 is suggested) rather than assessing tRNA cleavage specificity
- When tncRNA reads cannot be mapped exclusively to the artificial chromosomes (tRNA plus 50 nt leader/trailer), since ambiguous non-tRNA reads invalidate the cleavage-position composition analysis

## Inputs

- Per-sample tncRNA catalog with classified reads (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) from tncRNA Toolkit output
- Progenitor mature tRNA sequences plus 50 bp upstream/downstream flanks
- tRNA anticodon annotations (e.g. from tRNAscan-SE v2.0.6 predictions, score >= 50, pseudogenes removed)

## Outputs

- Cleavage-position nucleotide composition table (5' and 3' cleavage sites per tncRNA class)
- Codon/anticodon-grouped cleavage-count table compared to a random-degradation null expectation
- Conserved tRF-5::tRNA interface motif table across the six angiosperm species (e.g. AG::TGG)

## How to apply

Load the per-sample tncRNA catalog from the tncRNA Toolkit (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) together with progenitor mature tRNA sequences plus 50 nt flanks. For each tncRNA class, tally nucleotide frequencies at the 5' and 3' cleavage positions relative to the progenitor tRNA locus, and group cleavage counts by tRNA anticodon to quantify codon-dependent non-uniformity against a random-degradation null expectation. Extract tRF-5::tRNA interface sequences and score local alignments with Biopython pairwise2 (identical = 1, non-identical = -1, gap-open = -1, gap-extend = -0.5), then compile the conserved interface motif across species (the paper reports AG::TGG for tRF-5s of 15–19 nt in all six angiosperms). Accept the reproduction when the composition table shows non-random, codon/anticodon-dependent bias at cleavage sites, consistent with precise endoribonucleolytic cleavage rather than random degradation.

## Related tools

- **tncRNA Toolkit** (Produces the per-sample classified tncRNA catalogs and artificial-genome alignments that feed the cleavage-position and codon-dependence analyses) — https://github.com/skbinfo/tncRNA-Toolkit
- **Biopython** (pairwise2 module used to score local alignments of tRF-5::tRNA interface sequences (identical = 1, non-identical = -1, gap-open = -1, gap-extend = -0.5) to compile the conserved motif)
- **pandas** (Tabulation of nucleotide frequencies at cleavage positions and grouping of counts by anticodon (v1.1.0 dependency of the toolkit))
- **HAMR** (Predicts modified nucleosides on tncRNAs; conserved modifications may influence tRNA cleavage and inform motif interpretation)
- **Bowtie** (bowtie1 alignment of processed reads to the artificial genome index ('-v2 --best'; '--norc -v 0 -m 50' to discard reads mapping >50 times) supplying uniquely mapped tncRNAs for composition analysis)

## Examples

```
python3 tncRNAs.py -s <species_name> -i <processed_smallRNA.fastq> -o <output_dir>
```

## Evaluation signals

- The cleavage-position composition table shows non-random, codon/anticodon-dependent nucleotide bias at 5' and 3' cleavage sites, consistent with specific rather than random degradation
- A common conserved motif, AG::TGG, is recovered at the tRF-5::tRNA interface across all six angiosperms for tRF-5s of 15–19 nt
- Nucleotide-composition patterns are consistent with precise endoribonucleolytic cleavage machinery rather than stochastic tRNA breakdown
- tRNA halves are correctly defined by cleavage at the anticodon loop (2 nt + 3 nt of anticodon + 2 nt = 7 nt), confirming cleavage-position assignment is anchored to the progenitor tRNA locus
- Only reads mapping exclusively to the artificial chromosomes (masked genome with 50 nt upstream/downstream) contribute counts, so no ambiguous non-tRNA-region reads inflate the composition table

## Limitations

- Allowing a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives, distorting cleavage-site nucleotide calls
- Detailed mechanisms of how tRNA post-transcriptional modifications regulate cleavage remain unclear, so modified-nucleoside effects on motifs are interpretive rather than mechanistic
- The existence of isodecoders (tRNAs with the same anticodon) complicates attributing cleavage counts to individual tRNA genes and needs further exploration
- Experimental validation and characterization of predicted tncRNAs are still needed to distinguish bona fide tncRNAs

## Evidence
