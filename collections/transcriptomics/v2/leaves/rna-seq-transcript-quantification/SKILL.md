---
name: rna-seq-transcript-quantification
description: Use when raw RNA-seq FASTQ files from GEO/ENA (disease and control samples)
  must be converted to per-sample isoform-level TPM abundances as input for isoform
  switching / dMDT calling in the CanIsoNet workflow.
when_to_use_negative:
- Input is already a quantified gene-level count/TPM matrix with no isoform resolution
  — dMDT isoform switching requires transcript-level abundances
- The dataset is one of the already-processed conditions (27 PCAWG cancer types plus
  OIH, Alzheimer's, Parkinson's, Behcet's) browsable directly at caniso.net
- The analysis goal is network impact scoring only (the CanIsoNet/STRING step) without
  new RNA-seq quantification
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Kallisto
  role: Pseudoaligns RNA-seq reads to the human reference transcriptome to quantify
    isoform-level TPM abundances for every sample
- name: CanIsoNet
  role: Downstream pipeline (source code under Creative Commons license) that consumes
    the isoform expression values to call dMDT switching events between disease and
    control samples
  repo: github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Isoform-specific interaction network on which quantified isoform switching
    events are later assessed for functional/pathogenic impact
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:cfda10cbfc2a27322a2aa1c4e7f3a0efbc299cae5e84df1843029af8f2dd80df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CanIsoNet
  - Kallisto
  - STRING
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# rna-seq-transcript-quantification

## Summary

Quantify transcript/isoform-level abundances (TPM) for every sample in a disease RNA-seq study with Kallisto against the human reference transcriptome, producing the per-sample isoform expression values that the CanIsoNet dMDT isoform-switching pipeline requires. This is the expression-measurement step that converts raw FASTQ reads into isoform usage statistics comparable across disease and control samples.

## When to use

Use when raw RNA-seq FASTQ files from GEO/ENA (disease and control samples) must be converted to per-sample isoform-level TPM abundances as input for isoform switching / dMDT calling in the CanIsoNet workflow.

## When NOT to use

- Input is already a quantified gene-level count/TPM matrix with no isoform resolution — dMDT isoform switching requires transcript-level abundances
- The dataset is one of the already-processed conditions (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's) browsable directly at caniso.net
- The analysis goal is network impact scoring only (the CanIsoNet/STRING step) without new RNA-seq quantification

## Inputs

- Disease and control RNA-seq FASTQ files from a GEO/ENA accession
- Sample metadata table grouping samples into disease vs. control conditions
- Human reference transcriptome index for Kallisto

## Outputs

- Per-sample Kallisto quantification files with transcript/isoform TPM abundances
- Isoform expression matrix keyed by sample, ready for dMDT isoform-usage comparison
- Per-sample isoform expression values accompanying each isoform switching event in the results table

## How to apply

Download disease and control FASTQ files from the public repository accession (GEO/ENA) and verify the sample metadata grouping before quantification. Quantify transcript/isoform-level abundances for every sample with Kallisto against the human reference transcriptome index, keeping isoform rather than gene-level resolution because downstream dMDT calling compares isoform usage between disease and control groups. Assemble the per-sample quantification tables so each isoform switching event can later be explored with its expression values, matching the per-sample exploration format used across the 31 disease types hosted at caniso.net. The resulting expression values feed the dMDT switching statistics and the isoform-specific STRING interaction-network impact assessment with ClinVar highlighting and PFAM domain IDs.

## Related tools

- **Kallisto** (Pseudoaligns RNA-seq reads to the human reference transcriptome to quantify isoform-level TPM abundances for every sample)
- **CanIsoNet** (Downstream pipeline (source code under Creative Commons license) that consumes the isoform expression values to call dMDT switching events between disease and control samples) — github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Isoform-specific interaction network on which quantified isoform switching events are later assessed for functional/pathogenic impact)

## Examples

```
kallisto quant -i human_transcripts.idx -o sample1_out reads_1.fastq reads_2.fastq
```

## Evaluation signals

- A TPM quantification file exists for every sample in the accession, with non-negative expression values and consistent transcript identifiers against the reference transcriptome
- Quantified transcripts retain isoform (transcript-level) resolution, not collapsed gene-level abundances, since dMDT compares isoform usage
- Sample-to-condition mapping in the quantification matrix matches the verified sample metadata grouping (disease vs. control)
- Per-sample expression values are attachable to each isoform switching event, matching the CanIsoNet per-sample exploration format used for the 31 disease types

## Limitations

- New datasets beyond the 31 hosted disease types are only analysed and uploaded upon request, per the article's availability statement
- Isoform-level quantification accuracy depends on the human reference transcriptome annotation; lowly expressed or poorly annotated isoforms may yield unreliable usage estimates
- The README and article report slightly different annotation counts (11,528 vs. 11,811 isoforms), so schema expectations should be checked against the current database when preparing new results

## Evidence

- [abstract] Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases.: "Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases."
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
- [intro] CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [readme] We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project: "We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project"
- [readme] Exploring the isoform switching event in each sample with their expression values.: "Exploring the isoform switching event in each sample with their expression values."
