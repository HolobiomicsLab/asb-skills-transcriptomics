---
name: small-rna-seq-read-alignment
description: Use when quality- and adapter-trimmed single-end Illumina sRNA-seq FASTQ
  reads must be uniquely aligned to a customized genome index to identify small RNA
  loci (e.g., tncRNAs) while avoiding ambiguous multi-mapping to repetitive regions.
when_to_use_negative:
- Reads are paired-end or not quality-filtered and adapter-trimmed (e.g., not processed
  with Trim Galore v0.6.6) — the pipeline expects processed single-end small RNA reads
- Genome FASTA headers do not follow the '>chr[Num]', '>chrMt', '>chrPt' convention
  required by the tncRNA-Toolkit for nuclear/organellar separation
- The goal is to map reads to the native genome without masking tRNA regions — unmasked
  mapping would inflate ambiguous non-tRNA assignments the pipeline explicitly avoids
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: Bowtie1
  role: Short-read aligner used for both the error-tolerant unique alignment to the
    artificial genome index ('-v 2 --best') and the plus-strand realignment ('--norc
    -v 0 -m 50 --best --strata'); bundled as v1.3.0 in the toolkit's util directory
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Sorts SAM alignments and converts them to BAM (v1.10)
- name: bedtools
  role: Masks genuine tRNA gene regions plus 50 nt flanks in the reference genome
    via maskfasta (v2.29.2) before index building
- name: tRNAscan-SE
  role: Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs used
    to define the masked regions and artificial chromosomes
- name: Trim Galore
  role: Quality- and adapter-trims the single-end small RNA reads (v0.6.6) prior to
    alignment
  repo: https://github.com/FelixKrueger/TrimGalore
- name: tncRNA Toolkit
  role: Wraps the whole alignment workflow; builds the bowtie index via 'python3 tncRNAs.py
    -g <genome fasta> -s <species>' and runs tncRNA prediction from the aligned processed
    reads
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:56e2265f1ffce2499513cc4a31f5b18bedbaace531241b32bb519a1d86c2e201
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
  - Bowtie1
  - NCBI SRA Toolkit (v2.10)
  - Trim Galore
  - bedtools
  - biopython
  - pandas
  - samtools
  - tRNAscan-SE
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# small-rna-seq-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Bowtie1-based short-read alignment strategy for mapping quality-filtered plant sRNA-seq reads to an artificial genome (masked tRNA regions plus 50 nt leader/trailer as artificial chromosomes) so tRNA-derived reads can be uniquely and error-tolerantly assigned for tncRNA discovery.

## When to use

Use when quality- and adapter-trimmed single-end Illumina sRNA-seq FASTQ reads must be uniquely aligned to a customized genome index to identify small RNA loci (e.g., tncRNAs) while avoiding ambiguous multi-mapping to repetitive regions.

## When NOT to use

- Reads are paired-end or not quality-filtered and adapter-trimmed (e.g., not processed with Trim Galore v0.6.6) — the pipeline expects processed single-end small RNA reads
- Genome FASTA headers do not follow the '>chr[Num]', '>chrMt', '>chrPt' convention required by the tncRNA-Toolkit for nuclear/organellar separation
- The goal is to map reads to the native genome without masking tRNA regions — unmasked mapping would inflate ambiguous non-tRNA assignments the pipeline explicitly avoids

## Inputs

- Quality-filtered, adapter-trimmed single-end sRNA-seq FASTQ file (.fastq/.fq) from Illumina sequencing
- Reference genome FASTA (e.g., TAIR10.1 for Arabidopsis thaliana, SL4.0 nuclear / SL3.0 organellar for tomato) with headers starting '>chr[Num]', '>chrMt', '>chrPt'
- tRNA gene predictions from tRNAscan-SE (nuclear and organellar, pseudogene- and low-score-filtered)

## Outputs

- BAM files of uniquely mapped reads (SAM sorted and converted with samtools v1.10)
- Plus-strand realignment results for uniquely mapped reads (reads mapping >50 times discarded)
- Mapped unique reads classified as tncRNAs only if they align exclusively to artificial chromosomes (tRNA plus 50 nt leader and trailer)

## How to apply

First predict nuclear tRNAs with tRNAscan-SE (v2.0.6, eukaryotes model; organellar tRNAs with the '-O' option), discard pseudogenes and tRNAs scoring below 50, and mask tRNA regions with 50 bp 5' and 3' flanks using bedtools maskfasta. Append FASTA entries for mature tRNA (CCA-added, intron-removed), 50 bp leader, and 50 bp trailer as artificial chromosomes and build a Bowtie index with bowtie-build (bowtie v1.3.0). Align the processed reads with bowtie '-v 2 --best' to permit error-tolerant unique alignments, sort and convert SAM to BAM with samtools (v1.10), then extract uniquely mapped reads using SAM flags 0 and 16. Realign those reads to the plus strand only with bowtie1 '--norc -v 0 -m 50 --best --strata', discarding reads mapping more than 50 times. The rationale is that classifying only reads mapping exclusively to the artificial chromosomes prevents ambiguous non-tRNA assignments, and limiting mismatches to 2 ('-v 2') avoids the base misidentification that single-mismatch allowance can inflate.

## Related tools

- **Bowtie1** (Short-read aligner used for both the error-tolerant unique alignment to the artificial genome index ('-v 2 --best') and the plus-strand realignment ('--norc -v 0 -m 50 --best --strata'); bundled as v1.3.0 in the toolkit's util directory) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Sorts SAM alignments and converts them to BAM (v1.10))
- **bedtools** (Masks genuine tRNA gene regions plus 50 nt flanks in the reference genome via maskfasta (v2.29.2) before index building)
- **tRNAscan-SE** (Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs used to define the masked regions and artificial chromosomes)
- **Trim Galore** (Quality- and adapter-trims the single-end small RNA reads (v0.6.6) prior to alignment) — https://github.com/FelixKrueger/TrimGalore
- **tncRNA Toolkit** (Wraps the whole alignment workflow; builds the bowtie index via 'python3 tncRNAs.py -g <genome fasta> -s <species>' and runs tncRNA prediction from the aligned processed reads) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -g genome.fa -s species && python3 tncRNAs.py -s species -i processed_sRNA.fastq -o outdir -v 2 -m 50
```

## Evaluation signals

- Output catalog contains only reads mapping exclusively to artificial chromosomes (masked tRNA plus 50 nt leader and trailer), with no ambiguous non-tRNA assignments
- SAM flag check: uniquely mapped reads extracted with flags 0 (forward) and 16 (reverse); multi-mapped reads aligning more than 50 times are discarded via '-m 50'
- Each classified tncRNA carries a subtype label (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, tRNA halves) and an RPM value (reads mapped to progenitor tRNA / PMF, where PMF = total mapped reads/10^6)
- tRNA half classifications follow the 7 nt anticodon-loop cleavage rule (2 nt + 3 nt anticodon + 2 nt)
- Local alignment of each tncRNA to its sequence of origin is recorded via the pairwise2 biopython module

## Limitations

- Allowing just a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives; the pipeline uses '-v 2' to balance this
- Genome FASTA headers must start with '>chr[Num]' (e.g., >chr1, >chr2; >chrMt and >chrPt for organelles) or the automation scripts will not separate nuclear and organellar regions
- The tncRNA Toolkit's default -v is 0 (no mismatch) and values >3 are not recommended; default read count cut-off is 10 and multi-map limit is 50
- Conventional sRNA-seq biases from tRNA modifications persist; the article notes switching to tRNA-seq would be advantageous for tncRNA identification
- Experimental validation is still needed to distinguish bona fide tncRNAs, and tool/pipeline benchmarking with real positive datasets remains lacking

## Evidence
