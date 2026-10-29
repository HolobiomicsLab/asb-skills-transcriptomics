---
name: artificial-genome-masked-mapping
description: Use when identifying tRNA-derived small RNAs in plant sRNA-seq FASTQ
  data and ambiguous mapping to paralogous tRNA copies or non-tRNA genomic regions
  would otherwise inflate false-positive tncRNA calls.
when_to_use_negative:
- Input is long-read or paired-end RNA-seq data rather than single-end small RNA reads
  — the tncRNA Toolkit workflow expects processed single-end sRNA FASTQ input
- Reads cannot be uniquely assigned because the target genome lacks well-annotated
  tRNA loci, so the masked artificial chromosome strategy cannot resolve ambiguity
- The goal is general small RNA annotation (miRNAs, piRNAs, rRNAs) rather than tRNA-derived
  fragments — an annotation pipeline like SPORTS1.0 would be more appropriate
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: tncRNA Toolkit
  role: End-to-end workflow that builds the artificial genome bowtie index, aligns
    reads, and classifies tncRNA subtypes
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscan-SE (v2.0.6)
  role: Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes
    used to define the masked tRNA regions
- name: bedtools (v2.29.2)
  role: Masks tRNA regions plus 50 bp 5' and 3' flanks in the reference genome via
    maskfasta
- name: Bowtie1 (v1.3.0)
  role: Aligns reads to the artificial genome index ('-v 2 --best') and filters multi-mappers
    ('--norc -v 0 -m 50 --best --strata')
- name: samtools (v1.10)
  role: Sorts SAM output and converts to BAM; SAM flags 0 and 16 extract plus- and
    minus-strand unique alignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:4c06e34ad4a65a1ff12b6ccd7532d333bab06a7b5dbcec477cf39f4ccc1cb104
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
  - Bowtie1 (v1.3.0)
  - NCBI SRA Toolkit (v2.10)
  - bedtools (v2.29.2)
  - biopython
  - pandas
  - samtools
  - samtools (v1.10)
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# artificial-genome-masked-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A mapping strategy that builds an artificial genome by masking genuine tRNA gene regions with 50 nt 5' and 3' flanks and appending them as artificial chromosomes, so only reads mapping exclusively to tRNA-derived loci are classified as tncRNAs (tRF-5, tRF-3, tRF-1, tRNA halves, tRHs), avoiding ambiguous assignments from non-tRNA regions.

## When to use

Use when identifying tRNA-derived small RNAs in plant sRNA-seq FASTQ data and ambiguous mapping to paralogous tRNA copies or non-tRNA genomic regions would otherwise inflate false-positive tncRNA calls.

## When NOT to use

- Input is long-read or paired-end RNA-seq data rather than single-end small RNA reads — the tncRNA Toolkit workflow expects processed single-end sRNA FASTQ input
- Reads cannot be uniquely assigned because the target genome lacks well-annotated tRNA loci, so the masked artificial chromosome strategy cannot resolve ambiguity
- The goal is general small RNA annotation (miRNAs, piRNAs, rRNAs) rather than tRNA-derived fragments — an annotation pipeline like SPORTS1.0 would be more appropriate

## Inputs

- Single-end Illumina sRNA-seq FASTQ file (quality-filtered and adapter-trimmed, e.g. by Trim Galore v0.6.6)
- Reference genome assembly FASTA (e.g. TAIR10.1 for Arabidopsis thaliana, SL4.0/SL3.0 for tomato)
- Predicted tRNA gene annotations from tRNAscan-SE (pseudogenes and score < 50 removed)

## Outputs

- Bowtie index of the masked genome plus artificial chromosomes (tRNA region + 50 nt leader and trailer)
- BAM/SAM of uniquely mapped reads aligned with bowtie '-v 2 --best'
- Catalog of reads mapping exclusively to artificial chromosomes, each labeled with tncRNA subtype (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, tRNA halves) and RPM value

## How to apply

Predict nuclear tRNAs with tRNAscan-SE (v2.0.6, eukaryotes model) and organellar tRNAs with the '-O' option, then eliminate pseudogenes and tRNAs scoring below 50 from the gene pool. Mask the tRNA regions plus 50 bp upstream and downstream flanks in the reference genome (e.g. TAIR10.1, SL4.0/SL3.0) using bedtools maskfasta, build FASTA entries for CCA-added, intron-removed mature tRNA with its 50 bp leader and trailer, and append them to the masked genome as artificial chromosomes before building the Bowtie index (bowtie-build, v1.3). Align quality-filtered, adapter-trimmed single-end sRNA reads to the artificial genome index with bowtie '-v 2 --best' for error-tolerant unique alignments, extract mapped reads via SAM flags 0 and 16, realign to the plus strand only with '--norc -v 0 -m 50 --best --strata', and discard reads mapping more than 50 times. The rationale is that classifying only reads mapping exclusively to the artificial chromosomes prevents reads from non-tRNA regions from being mislabeled as tncRNAs, a problem that permissive single-mismatch mapping alone would worsen.

## Related tools

- **tncRNA Toolkit** (End-to-end workflow that builds the artificial genome bowtie index, aligns reads, and classifies tncRNA subtypes) — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscan-SE (v2.0.6)** (Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes used to define the masked tRNA regions)
- **bedtools (v2.29.2)** (Masks tRNA regions plus 50 bp 5' and 3' flanks in the reference genome via maskfasta)
- **Bowtie1 (v1.3.0)** (Aligns reads to the artificial genome index ('-v 2 --best') and filters multi-mappers ('--norc -v 0 -m 50 --best --strata'))
- **samtools (v1.10)** (Sorts SAM output and converts to BAM; SAM flags 0 and 16 extract plus- and minus-strand unique alignments)

## Examples

```
python3 tncRNAs.py -s <species name> -i <processed small RNA reads>.fastq -o <output dir>
```

## Evaluation signals

- Output catalog contains only reads mapping exclusively to the artificial chromosomes (masked tRNA + 50 nt flanks); no reads assigned from non-tRNA genomic regions
- Every called tncRNA carries a subtype label (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, tRNA halves) and an RPM value computed against PMF = total mapped reads / 10^6
- tRNA half classifications obey the 7 nt anticodon-loop cleavage rule (2 nt + 3 nt anticodon + 2 nt = 7 nt)
- No tRNA pseudogenes or tRNAscan-SE predictions with score < 50 contributed to the index
- Multi-mapped reads aligning more than 50 times are absent from the final mapped read set

## Limitations

- Allowing just a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives
- Experimental validation and characterization of predicted tncRNAs are still needed to distinguish bona fide tncRNAs
- tRNA-seq instead of conventional RNA-seq would be advantageous to overcome modification biases in tncRNA identification

## Evidence

- [results] To avoid ambiguous reads from non-tRNA regions, we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream: "To avoid ambiguous reads from non-tRNA regions, we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream"
- [methods] Error-tolerant and uniquely mapping reads alignment: "To permit error-tolerant and uniquely mapping reads alignments, processed reads were aligned to the artificial genome index using bowtie [60] with ‘-v2 --best’ options."
- [methods] Discard multi-mapped reads aligned more than 50 times: "multi-mapped reads were discarded if alignment occurred greater than 50 times, using bowtie1 with “--norc -v 0 -m 50” arguments."
- [readme] Index creation by the toolkit: "It will automatically create the bowtie index alongwith needed files in "lib/indexes/< species name >""
