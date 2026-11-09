---
name: tpm-expression-data-handling
description: Use when a transcript-level TPM matrix from an RNA-seq disease dataset
  (e.g. GSE203206 Alzheimer's) must be reduced to per-sample dominant isoforms to
  call disease-specific dMDT isoform switching events.
when_to_use_negative:
- Input is raw FASTQ or unquantified reads — transcript-level TPM values must already
  exist before MDT calling
- Goal is splice-junction/PSI-based isoform switching detection rather than dominant-transcript
  switching derived from TPM maxima
- Analysis targets one of the 27 PCAWG cancer types rather than the non-cancer RNA-seq
  datasets (OIH, Alzheimer's, Parkinson's, Behçet's) covered by the dMDT workflow
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CanIsoNet
  role: Provides the dMDT calling procedure and the disease-specific reference pages
    (e.g. Alzheimer's Fig. 1A) used to define and validate dominant-transcript calls
    and isoform switching events
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:67d9201d90f3a85a3d0909b56696144372cf66c8729c97c76fd9f128fedeb4c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tpm-expression-data-handling

## Summary

Handles transcript-per-million (TPM) expression matrices to identify per-sample Most Dominant Transcripts (MDTs) and derive disease-specific dMDT isoform switching events, following the CanIsoNet dMDT calling procedure on datasets such as Alzheimer's disease RNA-seq (GSE203206).

## When to use

Use when a transcript-level TPM matrix from an RNA-seq disease dataset (e.g. GSE203206 Alzheimer's) must be reduced to per-sample dominant isoforms to call disease-specific dMDT isoform switching events.

## When NOT to use

- Input is raw FASTQ or unquantified reads — transcript-level TPM values must already exist before MDT calling
- Goal is splice-junction/PSI-based isoform switching detection rather than dominant-transcript switching derived from TPM maxima
- Analysis targets one of the 27 PCAWG cancer types rather than the non-cancer RNA-seq datasets (OIH, Alzheimer's, Parkinson's, Behçet's) covered by the dMDT workflow

## Inputs

- Transcript-level TPM expression matrix for an RNA-seq disease dataset (e.g. GSE203206 Alzheimer's disease), loadable with pandas
- Sample disease/control labels for distinguishing disease-specific dMDTs
- Transcript-to-gene mapping implied by the TPM matrix rows

## Outputs

- alzheimer_dmdt_events.csv — isoform switching event table listing disease-specific dMDTs per sample with their expression values
- Per-sample dominant-transcript (MDT) calls per gene

## How to apply

Load the TPM expression matrix (e.g. the Alzheimer's disease dataset GSE203206) with pandas and index it by transcript. For each gene and each sample, call the Most Dominant Transcript (MDT) as the transcript isoform with the highest TPM value using the CanIsoNet dMDT calling procedure. Compare dominant transcripts between disease and control samples to identify disease-specific dMDTs, and derive the per-sample isoform switching events. Assemble the isoform switching event table listing dMDTs per sample with their expression values, mirroring the Alzheimer's disease-specific CanIsoNet page (Fig. 1A), and write it to alzheimer_dmdt_events.csv for validation.

## Related tools

- **CanIsoNet** (Provides the dMDT calling procedure and the disease-specific reference pages (e.g. Alzheimer's Fig. 1A) used to define and validate dominant-transcript calls and isoform switching events) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
import pandas as pd; tpm = pd.read_csv('GSE203206_TPM.tsv', sep='\t', index_col=0); mdt = tpm.idxmax(axis=0)
```

## Evaluation signals

- alzheimer_dmdt_events.csv exists and parses as a well-formed table
- Every listed isoform switching event is supported by a per-sample dominant-transcript call consistent with the dMDTs shown on the Alzheimer's disease-specific CanIsoNet page (Fig. 1A)
- The HSPH1-001 (858 AA) ↔ HSPH1-002 (814 AA) switching event appears among the top 10 most frequent dMDTs, observed in 21% of analysed Alzheimer's samples
- Each gene/sample pair in the MDT table corresponds to the transcript with the maximum TPM value in that sample
- Each sample carries at most one dominant transcript per gene, so derived switching events alternate between two annotated isoforms

## Limitations

- MDT calling depends entirely on TPM maxima, so lowly expressed genes and near-tie isoform abundances can yield unstable dominant-transcript calls
- Switching events are restricted to genes with at least two annotated isoforms; the database annotates 11,528 isoforms and 14,357 unique switching events across 31 diseases
- Event frequencies (e.g. the 21% HSPH1 observation) are dataset-specific and depend on the number of analysed Alzheimer's samples

## Evidence

- [readme] Exploring the isoform switching event in each sample with their expression values: "4) Exploring the isoform switching event in each sample with their expression values."
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
