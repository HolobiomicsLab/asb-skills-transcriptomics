---
name: local-alignment-scoring-with-pairwise2
description: Use when you have tRF-5::tRNA interface sequences (tRF-5s of 15-19 nt)
  extracted from tncRNA Toolkit output and need to score their local alignments to
  detect a conserved cleavage motif across plant species.
when_to_use_negative:
- Input is raw sRNA FASTQ rather than a classified tncRNA catalog - reads must first
  be classified by tncRNA Toolkit as mapping exclusively to the artificial tRNA+50nt
  chromosomes
- The sequences are not tRNA-derived fragments (e.g., miRNA or siRNA analysis); this
  scoring was applied specifically to tRF-5::tRNA cleavage interfaces
- You need to detect tncRNAs or modified nucleosides de novo (HAMR step) rather than
  score already-extracted interface sequences - use the upstream toolkit pipeline
  steps instead
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Biopython (pairwise2)
  role: Performs the local alignment scoring of tRF-5::tRNA interface sequences with
    the identical=1/non-identical=-1/gap-open=-1/gap-extend=-0.5 scheme
- name: tncRNA Toolkit
  role: Supplies the per-sample tncRNA catalog (tRF-5s, tRF-3s, tRF-1, tRNA halves)
    and progenitor tRNA+50 bp flank sequences that serve as the alignment inputs
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Assembles the cleavage-position composition and conserved-motif tables from
    the pairwise2 scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:b9fc5129fc2653a6cbcd012c31dc389269144adb207893897c53d85bfaac1bcf
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
  - Biopython (pairwise2)
  - HAMR
  - pandas
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# local-alignment-scoring-with-pairwise2

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores tRF-5::tRNA cleavage-interface sequences from plant tncRNA catalogs using Biopython pairwise2 local alignment with the scoring scheme match +1, mismatch -1, gap-open -1, gap-extend -0.5, to recover the conserved AG::TGG cleavage motif across angiosperms.

## When to use

Use when you have tRF-5::tRNA interface sequences (tRF-5s of 15-19 nt) extracted from tncRNA Toolkit output and need to score their local alignments to detect a conserved cleavage motif across plant species.

## When NOT to use

- Input is raw sRNA FASTQ rather than a classified tncRNA catalog - reads must first be classified by tncRNA Toolkit as mapping exclusively to the artificial tRNA+50nt chromosomes
- The sequences are not tRNA-derived fragments (e.g., miRNA or siRNA analysis); this scoring was applied specifically to tRF-5::tRNA cleavage interfaces
- You need to detect tncRNAs or modified nucleosides de novo (HAMR step) rather than score already-extracted interface sequences - use the upstream toolkit pipeline steps instead

## Inputs

- Per-sample tncRNA catalog of classified reads (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) from tncRNA Toolkit
- Progenitor mature tRNA sequences plus 50 bp upstream/downstream flanks (artificial tRNA+50nt chromosomes)
- tRF-5 sequences restricted to the 15-19 nt size window

## Outputs

- Pairwise local-alignment scores for tRF-5::tRNA interface sequence pairs
- Conserved tRF-5::tRNA interface motif table (AG::TGG) across the six angiosperms
- Cleavage-position nucleotide composition table

## How to apply

Load the per-sample tncRNA catalog (classified tRF-5, tRF-3, tRF-1, 5'tRH, and 3'tRH reads) plus the progenitor mature tRNA sequences with 50 bp flanks produced by tncRNA Toolkit, which classified reads mapping exclusively to artificial tRNA+50nt chromosomes with bowtie '-v2 --best'. Extract tRF-5::tRNA interface subsequences for the 15-19 nt tRF-5 window in which motif conservation was reported. Score each interface pair with pairwise2 local alignment using identical = +1, non-identical = -1, gap-open = -1, gap-extend = -0.5, so exact matches dominate and cleavage-site mismatches stand out. Compile the top-scoring aligned interfaces across all six angiosperms and write the motif and cleavage-position composition tables with pandas. Accept the analysis when the alignments converge on the AG::TGG motif and the composition table shows non-random, codon/anticodon-dependent nucleotide bias consistent with specific endoribonucleolytic cleavage rather than random tRNA degradation.

## Related tools

- **Biopython (pairwise2)** (Performs the local alignment scoring of tRF-5::tRNA interface sequences with the identical=1/non-identical=-1/gap-open=-1/gap-extend=-0.5 scheme)
- **tncRNA Toolkit** (Supplies the per-sample tncRNA catalog (tRF-5s, tRF-3s, tRF-1, tRNA halves) and progenitor tRNA+50 bp flank sequences that serve as the alignment inputs) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Assembles the cleavage-position composition and conserved-motif tables from the pairwise2 scores)

## Examples

```
from Bio import pairwise2; score = pairwise2.align.localms(tRF5_iface, trna_iface, 1, -1, -1, -0.5, score_only=True)
```

## Evaluation signals

- All six angiosperm tRF-5::tRNA interface alignments (15-19 nt tRF-5s) converge on the AG::TGG motif
- Exact-match interfaces score as 2 x alignment length (match +1, no gaps); substantially lower scores flag mismatches/gaps at the cleavage site
- The cleavage-position composition table shows non-random, codon/anticodon-dependent nucleotide bias versus the random-degradation null expectation
- The script passes exactly (1, -1, -1, -0.5) to pairwise2.align.localms; re-running on identical inputs reproduces identical deterministic scores
- Only reads classified exclusively to the artificial tRNA+50nt chromosomes enter the alignment step (toolkit classification filter)

## Limitations

- AG::TGG motif conservation was demonstrated only for tRF-5s of 15-19 nt across the six angiosperms studied; it may not transfer outside this size window or clade
- Gap penalties (gap-open -1, gap-extend -0.5) penalize gapped interfaces heavily, so indel-containing cleavage products may be under-represented in top alignments
- The toolkit pins biopython v1.77; pairwise2 is deprecated/removed in newer Biopython releases, so pin the version or port the scoring to Bio.Align.PairwiseAligner
- Isodecoders (tRNAs sharing the same anticodon) complicate attributing a tncRNA to a unique progenitor tRNA during motif compilation

## Evidence

- [other] pairwise2 scoring scheme: [evidence span withheld — anchor status: not_found]
- [results] AG::TGG conserved motif finding: [evidence span withheld — anchor status: not_found]
