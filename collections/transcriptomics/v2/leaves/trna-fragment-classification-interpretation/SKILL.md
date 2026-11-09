---
name: trna-fragment-classification-interpretation
description: Use when single-end sRNA-seq FASTQ reads must be assigned to tRF-5/tRF-3/tRF-1/tRNA-half
  classes against progenitor mature tRNA loci, or when testing whether tRNA cleavage
  is non-random and codon/anticodon-dependent.
when_to_use_negative:
- The reads are not small RNA-seq (14-50 nt) or derive from organisms lacking curated
  tRNA predictions — the pipeline and its human-trained alternatives (tDRmapper, MINTmap,
  tRF2Cancer) do not generalize.
- The goal is general small-RNA annotation (miRNA, piRNA, rRNA) rather than tRNA-derived
  fragments; a pipeline like SPORTS1.0 is better suited.
- Input is already a classified tncRNA count matrix and the task is downstream expression
  or pathway analysis, not cleavage-position classification.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: tncRNA Toolkit
  role: Core workflow to detect and classify tRNA-derived non-coding RNAs (tRF-5s,
    tRF-3s, tRF-1, tRNA halves) from genome FASTA plus single-end sRNA FASTQ
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Bowtie1
  role: Aligns processed reads to the artificial-genome index with '-v2 --best' and
    '--norc -v 0 -m 50' to keep uniquely/exclusively mapped tncRNA reads
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Adapter and quality trimming of single-end small RNA reads prior to tncRNA
    prediction
  repo: https://github.com/FelixKrueger/TrimGalore
- name: tRNAscan-SE (v2.0.6)
  role: Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs; pseudogenes
    and score <50 eliminated to build the mapping reference
- name: HAMR (v1.2)
  role: Predicts modified nucleosides on tncRNAs from high-throughput sequencing data
    to assess conserved modifications influencing cleavage
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Biopython
  role: pairwise2 local alignment (identical=1, non-identical=-1, gap-open=-1, gap-extend=-0.5)
    of tRF-5::tRNA interface sequences to score conserved motifs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:5b890fe38fed6e63e6255bd2844dcf24024f0f104e43a11b92e1b9a079a0bdf7
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
  - Bowtie1
  - HAMR
  - HAMR (v1.2)
  - Trim Galore
  - pandas
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# trna-fragment-classification-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies small RNA-seq reads into tRNA-derived non-coding RNA (tncRNA) subtypes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) and interprets their generation as codon-dependent, precise endoribonucleolytic cleavage rather than random tRNA degradation.

## When to use

Use when single-end sRNA-seq FASTQ reads must be assigned to tRF-5/tRF-3/tRF-1/tRNA-half classes against progenitor mature tRNA loci, or when testing whether tRNA cleavage is non-random and codon/anticodon-dependent.

## When NOT to use

- The reads are not small RNA-seq (14-50 nt) or derive from organisms lacking curated tRNA predictions — the pipeline and its human-trained alternatives (tDRmapper, MINTmap, tRF2Cancer) do not generalize.
- The goal is general small-RNA annotation (miRNA, piRNA, rRNA) rather than tRNA-derived fragments; a pipeline like SPORTS1.0 is better suited.
- Input is already a classified tncRNA count matrix and the task is downstream expression or pathway analysis, not cleavage-position classification.

## Inputs

- Genome FASTA with headers '>chr[Num]' (>chrMt, >chrPt for organellar contigs)
- Single-end small RNA reads FASTQ (quality filtered and adapter trimmed), e.g. from 2,448 SRA/NCBI sRNA-seq samples
- Progenitor mature tRNA sequences plus 50 bp flanks (tRNAscan-SE v2.0.6 predictions, pseudogenes and score <50 removed)
- Per-sample tncRNA catalog classified by subtype from tncRNA Toolkit output

## Outputs

- Classified and quantified tncRNA catalog by subtype (tRF-5, tRF-3, tRF-1, tRNA halves), 14-50 nt
- Cleavage-position nucleotide composition table grouped by anticodon vs random-degradation null
- Conserved tRF-5::tRNA interface motif table (e.g. AG::TGG across six angiosperms, 15-19 nt tRF-5s)
- Modified nucleoside site annotations along individual tncRNAs (HAMR)

## How to apply

Run the tncRNA Toolkit workflow: first build a bowtie index from the genome FASTA whose headers start with '>chr[Num]' (>chrMt, >chrPt for organelles), which masks genuine tRNA gene regions with 50 nt upstream/downstream into artificial chromosomes. Align quality- and adapter-trimmed (Trim Galore v0.6.6) reads with bowtie1 '-v2 --best' and discard reads mapping more than 50 times ('--norc -v 0 -m 50'); only reads mapping exclusively to artificial chromosomes are classified as tncRNAs. Classify subtypes by cleavage position relative to the progenitor tRNA, with tRNA halves defined by cleavage at the anticodon loop (2 nt + 3 nt of anticodon + 2 nt = 7 nt). For interpretation, tally nucleotide frequencies at 5' and 3' cleavage positions grouped by tRNA anticodon against a random-degradation null, and align tRF-5::tRNA interface sequences with Biopython pairwise2 (match 1, mismatch -1, gap-open -1, gap-extend -0.5) to recover conserved motifs such as AG::TGG in 15-19 nt tRF-5s. Optionally annotate modified nucleosides with HAMR (v1.2, python2) to check for conserved modifications that may influence cleavage.

## Related tools

- **tncRNA Toolkit** (Core workflow to detect and classify tRNA-derived non-coding RNAs (tRF-5s, tRF-3s, tRF-1, tRNA halves) from genome FASTA plus single-end sRNA FASTQ) — https://github.com/skbinfo/tncRNA-Toolkit
- **Bowtie1** (Aligns processed reads to the artificial-genome index with '-v2 --best' and '--norc -v 0 -m 50' to keep uniquely/exclusively mapped tncRNA reads) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Adapter and quality trimming of single-end small RNA reads prior to tncRNA prediction) — https://github.com/FelixKrueger/TrimGalore
- **tRNAscan-SE (v2.0.6)** (Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs; pseudogenes and score <50 eliminated to build the mapping reference)
- **HAMR (v1.2)** (Predicts modified nucleosides on tncRNAs from high-throughput sequencing data to assess conserved modifications influencing cleavage) — https://github.com/skbinfo/tncRNA-Toolkit
- **Biopython** (pairwise2 local alignment (identical=1, non-identical=-1, gap-open=-1, gap-extend=-0.5) of tRF-5::tRNA interface sequences to score conserved motifs)

## Examples

```
python3 tncRNAs.py -g genome.fa -s species_name && python3 tncRNAs.py -s species_name -i trimmed_reads.fastq -o out_dir
```

## Evaluation signals

- Composition table shows non-random, codon/anticodon-dependent nucleotide bias at cleavage sites, rejecting the random tRNA-degradation null.
- A conserved interface motif (AG::TGG) is recovered at the tRF-5::tRNA interface across all six angiosperms for 15-19 nt tRF-5s.
- Only reads mapping exclusively to artificial chromosomes (tRNA ± 50 nt flanks) enter the tncRNA catalog; multi-mapped reads (>50 alignments) are discarded.
- All retained tRNAs have tRNAscan-SE score ≥ 50 and pseudogenes removed; subtypes sum to a plausible 14-50 nt length distribution.
- Toolkit scale check: ~43,153 novel tncRNAs identified across 2,448 samples in six angiosperms in the published run.

## Limitations

- Allowing a single mismatch to overcome sequencing errors may cause base misidentification and inflation of false negatives.
- HAMR v1.2 requires python2; tncRNA Toolkit requires python3, tRNAscan-SE, samtools (v1.10), bedtools (v2.29.2), and bowtie1 (v1.3.0) installed globally or on PATH.
- Genome FASTA headers must start with '>chr[Num]' (>chrMt, >chrPt for organelles) for pipeline automation; bowtie -v should not exceed 3.
- Conventional RNA-seq carries modification biases — tRNA-seq would be advantageous for tncRNA identification; experimental validation is still needed to distinguish bona fide tncRNAs.

## Evidence
