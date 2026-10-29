---
name: rna-seq-fastq-download-from-geo-ena
description: Use when raw RNA-seq reads for a published study must be fetched from
  public archives — GSE accessions via GEO/SRA or a BioProject via direct ENA run
  downloads — before transcript-level quantification.
when_to_use_negative:
- The transcript abundance matrix (e.g. tpm_abundance_matrix.tsv) already exists and
  quantification is complete — downloading FASTQ again is redundant.
- The target data are the 27 cancer types from the PCAWG project, which CanIsoNet
  covers through PCAWG rather than these four RNA-seq dataset downloads.
- Only genome-level raw read analysis (e.g. genomic alignment) is needed — this skill's
  pipeline is oriented to transcript-level Kallisto quantification compatible with
  isoform switching analysis.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: Kallisto
  role: 'Downstream consumer of the downloaded FASTQs: run in paired-end mode against
    an Ensembl GRCh37.p13 transcript cDNA index to produce the TPM values used in
    CanIsoNet'
- name: CanIsoNet
  role: The database whose downstream isoform switching analysis defines the transcript
    reference and TPM matrix requirements for the downloaded datasets
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:719e01ac424993451e73968672cb8327e6415c89293309e4ec5ba76f3a5e87c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rna-seq-fastq-download-from-geo-ena

## Summary

Retrieves raw paired-end RNA-seq FASTQ files for published disease datasets (OIH, Alzheimer's, Parkinson's, Behcet's) from GEO/SRA accessions (GSE203206, GSE128177, GSE205867) and via direct ENA run downloads for BioProject PRJNA433105, as the entry step of the CanIsoNet transcript-abundance pipeline.

## When to use

Use when raw RNA-seq reads for a published study must be fetched from public archives — GSE accessions via GEO/SRA or a BioProject via direct ENA run downloads — before transcript-level quantification.

## When NOT to use

- The transcript abundance matrix (e.g. tpm_abundance_matrix.tsv) already exists and quantification is complete — downloading FASTQ again is redundant.
- The target data are the 27 cancer types from the PCAWG project, which CanIsoNet covers through PCAWG rather than these four RNA-seq dataset downloads.
- Only genome-level raw read analysis (e.g. genomic alignment) is needed — this skill's pipeline is oriented to transcript-level Kallisto quantification compatible with isoform switching analysis.

## Inputs

- GEO/SRA accession GSE203206 (OIH RNA-seq dataset)
- GEO/SRA accession GSE128177 (Alzheimer's Disease RNA-seq dataset)
- GEO/SRA accession GSE205867 (Parkinson's Disease RNA-seq dataset)
- BioProject PRJNA433105 (Behcet's Disease RNA-seq dataset, via direct ENA run downloads)
- Ensembl human GRCh37.p13 transcript cDNA sequences (for building the Kallisto index)

## Outputs

- Paired-end FASTQ files per sample for the four disease datasets
- Kallisto index built from Ensembl GRCh37.p13 transcript cDNA sequences
- Per-sample Kallisto quant transcript abundance files
- tpm_abundance_matrix.tsv — a TPM matrix with transcripts as rows and samples from all four disease datasets as columns

## How to apply

Resolve each dataset identifier to run-level FASTQ files: datasets GSE203206, GSE128177 and GSE205867 are fetched through their GEO/SRA accessions, while PRJNA433105 is fetched by direct ENA run downloads. Confirm each run is paired-end, since the downstream step runs Kallisto quant in paired-end mode. Build the quantification reference up front — a Kallisto index from Ensembl human GRCh37.p13 transcript cDNA sequences (mouse GRCm38.p6 is the equivalent reference for mouse data) — so isoform definitions match the transcript-level TPMs used downstream in CanIsoNet. Aggregate per-sample TPM values from the Kallisto quant output into a single matrix with transcripts as rows and samples as columns (tpm_abundance_matrix.tsv).

## Related tools

- **Kallisto** (Downstream consumer of the downloaded FASTQs: run in paired-end mode against an Ensembl GRCh37.p13 transcript cDNA index to produce the TPM values used in CanIsoNet)
- **CanIsoNet** (The database whose downstream isoform switching analysis defines the transcript reference and TPM matrix requirements for the downloaded datasets) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
wget "https://www.ebi.ac.uk/ena/portal/api/filereport?accession=PRJNA433105&result=read_run&fields=fastq_ftp" -O ena_runs.tsv
```

## Evaluation signals

- tpm_abundance_matrix.tsv contains one column per sample from all four disease datasets (OIH, Alzheimer's, Parkinson's, Behcet's)
- All TPM entries in tpm_abundance_matrix.tsv are non-negative numeric values
- Every accession (GSE203206, GSE128177, GSE205867, PRJNA433105) resolved to run-level FASTQ files with paired-end read pairs
- The Kallisto index was built from Ensembl GRCh37.p13 human transcript cDNA so isoform definitions match those used downstream
- Row identifiers in the TPM matrix correspond to Ensembl transcript sequences compatible with CanIsoNet's transcript isoform definitions

## Limitations

- The workflow explicitly specifies paired-end Kallisto quant mode; single-end runs from these archives are not covered.
- The transcript reference is fixed to Ensembl GRCh37.p13 (human) / GRCm38.p6 (mouse); other genome builds are not validated against CanIsoNet's isoform definitions.
- Retrieval depends on external archive availability (GEO/SRA and direct ENA run downloads), with no mirror strategy described.

## Evidence

- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [other] GSE203206, GSE128177 and GSE205867 via GEO/SRA accessions and PRJNA433105 via direct ENA run downloads: [evidence span withheld — anchor status: not_found]
- [other] Build a Kallisto index from the Ensembl GRCh37.p13 human transcript cDNA sequences: [evidence span withheld — anchor status: not_found]
- [readme] 27 different cancer types from PCAWG project, and four publicly available RNA-Seq datasets on OIH: "27 different cancer types from PCAWG project, and four publicly available RNA-Seq datasets on OIH"
