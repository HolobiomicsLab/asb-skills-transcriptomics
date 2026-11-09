---
name: isoform-dominant-transcript-calling
description: Use when you have a gene-by-transcript TPM expression matrix from a case/control
  RNA-seq dataset and need to identify per-sample dominant transcript isoforms and
  derive disease-specific isoform switching events.
when_to_use_negative:
- Input is raw RNA-seq reads or unquantified FASTQ — TPM values must already be computed
  per transcript before MDT calling
- The goal is differential gene-level expression rather than transcript isoform dominance/switching
  within genes
- The disease type of interest is not covered by CanIsoNet's 31 disease types (27
  PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's), so no reference
  dMDT page exists for validation
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CanIsoNet
  role: 'Reference database and dMDT calling procedure: supplies the disease-specific
    dMDT pages (e.g. Alzheimer''s Fig. 1A) used to define and validate per-sample
    dominant transcript calls and isoform switching events'
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:5bcac112210adb34773475156e60ec5e732ef6d5068f4e20a9f3a82dd654ac3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# isoform-dominant-transcript-calling

## Summary

Calls per-sample and disease-specific Most Dominant Transcripts (MDTs/dMDTs) from RNA-seq TPM expression matrices to detect isoform switching events, following the CanIsoNet dMDT procedure. This reproduces the transcript-level dominance logic behind CanIsoNet's disease-specific isoform switching tables, e.g. the Alzheimer's disease HSPH1-001/HSPH1-002 event.

## When to use

Use when you have a gene-by-transcript TPM expression matrix from a case/control RNA-seq dataset and need to identify per-sample dominant transcript isoforms and derive disease-specific isoform switching events.

## When NOT to use

- Input is raw RNA-seq reads or unquantified FASTQ — TPM values must already be computed per transcript before MDT calling
- The goal is differential gene-level expression rather than transcript isoform dominance/switching within genes
- The disease type of interest is not covered by CanIsoNet's 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's), so no reference dMDT page exists for validation

## Inputs

- TPM transcript-level expression matrix (gene/transcript by sample) from a publicly available RNA-Seq dataset, e.g. GSE203206 Alzheimer's disease
- Disease/control sample labels for dMDT specificity calling
- Reference CanIsoNet disease-specific dMDT page (e.g. Alzheimer's disease Fig. 1A) for validation

## Outputs

- alzheimer_dmdt_events.csv: isoform switching event table listing disease-specific dMDTs per sample with their expression values
- Per-sample dominant-transcript (MDT) calls per gene
- Ranked list of frequent isoform switching events (e.g. top 10 most frequent dMDTs with per-sample frequency such as 21% of Alzheimer's samples)

## How to apply

Load the TPM expression matrix of the disease dataset (e.g. GSE203206 for Alzheimer's) with pandas. For each gene in each sample, apply the CanIsoNet dMDT calling procedure: assign the Most Dominant Transcript as the transcript isoform with the highest TPM value. Compare dominant transcripts between disease and control samples to call disease-specific dMDTs, and derive the isoform switching event per sample (e.g. canonical HSPH1-001, 858 AA, switching to short HSPH1-002, 814 AA). Assemble a per-sample isoform switching event table with expression values, mirroring the Alzheimer's disease-specific CanIsoNet page (Fig. 1A), and write it to CSV. Cross-check called events against the CanIsoNet web server (caniso.net), which provides 'disease-type specific dMDTs' and per-sample isoform switching events with expression values.

## Related tools

- **CanIsoNet** (Reference database and dMDT calling procedure: supplies the disease-specific dMDT pages (e.g. Alzheimer's Fig. 1A) used to define and validate per-sample dominant transcript calls and isoform switching events) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
import pandas as pd; tpm = pd.read_csv('gse203206_tpm.csv', index_col=0); mdt = tpm.T.groupby(level=0).idxmax()
```

## Evaluation signals

- alzheimer_dmdt_events.csv exists and is a well-formed table (parses without error, consistent columns)
- Every listed isoform switching event is supported by a per-sample dominant-transcript call consistent with the dMDTs shown on the Alzheimer's disease-specific CanIsoNet page (Fig. 1A)
- Frequently observed events match reported frequencies, e.g. the HSPH1-001 (858 AA) / HSPH1-002 (814 AA) switch appears in 21% of analysed Alzheimer's samples and ranks among the top 10 most frequent dMDTs
- Each per-sample MDT call is the transcript with the maximum TPM for its gene in that sample (verifiable by re-deriving argmax over the TPM matrix)

## Limitations

- dMDT calling depends on relative TPM dominance, so lowly expressed genes yield unstable dominant-transcript assignments; no explicit TPM threshold is specified in the article or README
- Validation is anchored to the four non-cancer RNA-seq datasets (OIH, Alzheimer's, Parkinson's, Behcet's) and PCAWG cancer types covered by CanIsoNet; events in other disease types have no reference dMDT page for cross-checking
- CanIsoNet annotates 14,357 unique isoform switching events across 31 diseases (README reports 11,528 isoforms vs 11,811 in the article), so isoform coverage may not include every transcript in the input matrix

## Evidence

- [other] For each gene and each sample, determine the Most Dominant Transcript (MDT) as the transcript isoform with the highest TPM value using the CanIsoNet dMDT calling procedure.: [evidence span withheld — anchor status: not_found]
- [other] From the Alzheimer's disease RNA-seq data, a switching event between the canonical transcript HSPH1-001 (858 AA) and the short transcript HSPH1-002 (814 AA) is among the top 10 most frequent dMDTs, observed in 21% of the analysed Alzheimer's samples.: [evidence span withheld — anchor status: not_found]
- [readme] README: CanIsoNet provides the ability to explore disease-type specific dMDTs and per-sample isoform switching events with expression values.: "3) Exploring the disease-type specific dMDTs. <br> 4) Exploring the isoform switching event in each sample with their expression values."
- [intro] Article intro: CanIsoNet enables per-sample exploration of isoform switching events with expression values.: [evidence span withheld — anchor status: not_found]
- [readme] README availability statement grounding the reference database used for validation.: "CanIsoNet is freely available at [caniso.net](https://www.caniso.net/)"
