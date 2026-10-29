---
name: bowtie-index-generation
description: Use when building the reference index for tncRNA (tRF/tRNA-half) detection
  from small RNA-seq, where reads must be classified only if they map exclusively
  to tRNA-derived artificial chromosomes rather than non-tRNA genomic loci.
when_to_use_negative:
- Input reads are not single-end small RNA-seq being classified as tncRNAs (tRF-5s,
  tRF-3s, tRF-1, tRNA halves) — e.g. bulk RNA-seq or miRNA profiling pipelines that
  map to the unmodified reference genome
- A tncRNA detection tool that maps directly against mature tRNA sequences (e.g. tDRmapper,
  MINTmap, tRF2Cancer) is already in use and ambiguous multi-mapping to non-tRNA regions
  is not a concern
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0659
tools:
- name: bedtools
  role: maskfasta script used to mask filtered tRNA regions plus 50 bp 3' and 5' flanks
    in the reference genome before appending artificial chromosomes
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Bowtie
  role: bowtie-build (bowtie v1.3 / Bowtie1 v1.3.0) builds the index for the artificial
    genome so processed reads can be aligned with '-v2 --best' and '--norc -v 0 -m
    50'
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscan-SE
  role: predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes
    that define the masked regions and artificial chromosomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:09fd0c57759ad3052379e063f9b15e4a2f460dcaf0bc031af9e21eebec913cc4
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
  - Bowtie
  - bedtools
  - tRNAscan-SE
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# bowtie-index-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a Bowtie1 index for an artificial genome in which tRNA gene regions (plus 50 nt 5' leader and 3' trailer flanks) are masked in the reference genome and the masked tRNA sequences are appended as artificial chromosomes, enabling unambiguous mapping of tRNA-derived small RNA reads in the tncRNA Toolkit pipeline.

## When to use

Use when building the reference index for tncRNA (tRF/tRNA-half) detection from small RNA-seq, where reads must be classified only if they map exclusively to tRNA-derived artificial chromosomes rather than non-tRNA genomic loci.

## When NOT to use

- Input reads are not single-end small RNA-seq being classified as tncRNAs (tRF-5s, tRF-3s, tRF-1, tRNA halves) — e.g. bulk RNA-seq or miRNA profiling pipelines that map to the unmodified reference genome
- A tncRNA detection tool that maps directly against mature tRNA sequences (e.g. tDRmapper, MINTmap, tRF2Cancer) is already in use and ambiguous multi-mapping to non-tRNA regions is not a concern

## Inputs

- Reference genome FASTA (headers starting '>chr[Num]', organellar as '>chrMt'/'chrPt')
- tRNAscan-SE predicted tRNA set (nuclear and organellar)
- Filtered tRNA FASTA files: mature tRNA (intron-removed, CCA-added), 50 bp 5' leader flank, 50 bp 3' trailer flank

## Outputs

- Artificial genome FASTA (masked reference chromosomes plus tRNA-derived artificial chromosomes)
- Bowtie1 index files (artificial_genome.*.ebwt) under lib/indexes/<species name>

## How to apply

First filter the tRNAscan-SE (v2.0.6) predictions by removing pseudogenes and tRNAs with Covels score below 50 (eukaryote model for nuclear tRNAs, '-O' option for organellar tRNAs). Prepare three FASTA files from the filtered tRNAs: mature tRNA (intron removed, CCA added at the 3' terminus), the 50 bp 5' leader flank, and the 50 bp 3' trailer flank. Mask the filtered tRNA regions plus 50 bp flanks on both sides in the reference genome using the maskfasta script of bedtools (v2.29), then concatenate the three tRNA FASTA files onto the masked genome as additional chromosomes to create the artificial genome. FASTA headers must begin with '>chr[Num]' (e.g. >chr1, >chr2), with organellar sequences as >chrMt and >chrPt, so the toolkit scripts can separate nuclear and organellar regions. Finally run bowtie-build (bowtie v1.3) on the artificial genome, writing index files under lib/indexes/<species name>.

## Related tools

- **bedtools** (maskfasta script used to mask filtered tRNA regions plus 50 bp 3' and 5' flanks in the reference genome before appending artificial chromosomes) — https://github.com/skbinfo/tncRNA-Toolkit
- **Bowtie** (bowtie-build (bowtie v1.3 / Bowtie1 v1.3.0) builds the index for the artificial genome so processed reads can be aligned with '-v2 --best' and '--norc -v 0 -m 50') — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscan-SE** (predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes that define the masked regions and artificial chromosomes)

## Examples

```
python3 tncRNAs.py -g <genome fasta> -s <species name>
```

## Evaluation signals

- Bowtie index files for the artificial genome exist (artificial_genome.*.ebwt) under lib/indexes/<species name>
- The artificial genome FASTA contains the masked reference chromosomes plus the additional tRNA-derived chromosomes
- All FASTA headers in the artificial genome begin with '>chr[Num]' (nuclear) and '>chrMt'/'>chrPt' (organellar)
- Downstream bowtie alignment with '--norc -v 0 -m 50' succeeds and tncRNAs are called only from reads mapping exclusively to the artificial chromosomes
- Genome index build via 'python3 tncRNAs.py -g <genome fasta> -s <species name>' completes without errors and creates the needed files automatically

## Limitations

- Genome FASTA headers must start with '>chr[Num]' (and '>chrMt'/'chrPt' for organellar) or the toolkit's automation for separating nuclear and organellar regions breaks
- The masking/appending strategy depends on the tRNA filter thresholds (pseudogene removal, Covels score >= 50); low-quality or pseudogene tRNAs in the pool would create spurious artificial chromosomes
- Allowing a single mismatch during mapping to overcome sequencing errors may lead to base misidentification and inflation of false negatives

## Evidence

- [methods] Masking tRNA regions in reference genome: [evidence span withheld — anchor status: not_found]
- [other] bowtie-build index location: [evidence span withheld — anchor status: not_found]
