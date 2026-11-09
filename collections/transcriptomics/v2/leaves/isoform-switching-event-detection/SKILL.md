---
name: isoform-switching-event-detection
description: Use when transcript-level TPM abundances (e.g. Kallisto-quantified FASTQ
  from GEO/ENA) must be converted into per-sample dominant-transcript calls and disease-vs-control
  isoform switching event tables, as done for the 31 CanIsoNet disease types including
  Alzheimer's GSE203206.
when_to_use_negative:
- Input lacks transcript-level TPM abundances (e.g. only gene-level counts or protein-level
  data), since MDT calling requires per-transcript TPM per sample.
- The analysis goal is differential gene expression or alternative splicing quantification
  rather than dominance/switching of the highest-TPM isoform between disease and control
  samples.
- The organism is outside the human (and mouse) scope of the 3did/STRING/ClinVar-integrated
  CanIsoNet networks and no compatible transcriptome reference is available.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CanIsoNet
  role: Source of the dMDT calling procedure and isoform switching event framework;
    pipeline code at kahramanlab/CanIsoNet_Web defines the per-sample event table
    and database schema for 31 disease types
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Protein-protein interaction network (v10.0) onto which disrupted Pfam domain-domain
    interactions are overlaid to assess functional impact of each switching event
- name: Kallisto
  role: Quantifies transcript/isoform-level TPM abundances for every sample against
    the human reference transcriptome before dMDT calling
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:14cdd91ddf5c6e99a110f85d7bc3e4428cedc2743058f7c76ec636698300edbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  - STRING
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# isoform-switching-event-detection

## Summary

Detect disease-specific Most Dominant Transcript (dMDT) isoform switching events by calling the highest-TPM transcript per gene per sample from RNA-seq quantification, then comparing dominant transcripts between disease and control samples. This is the core procedure behind the CanIsoNet database of isoform switching events and their pathogenic impact on protein interactions.

## When to use

Use when transcript-level TPM abundances (e.g. Kallisto-quantified FASTQ from GEO/ENA) must be converted into per-sample dominant-transcript calls and disease-vs-control isoform switching event tables, as done for the 31 CanIsoNet disease types including Alzheimer's GSE203206.

## When NOT to use

- Input lacks transcript-level TPM abundances (e.g. only gene-level counts or protein-level data), since MDT calling requires per-transcript TPM per sample.
- The analysis goal is differential gene expression or alternative splicing quantification rather than dominance/switching of the highest-TPM isoform between disease and control samples.
- The organism is outside the human (and mouse) scope of the 3did/STRING/ClinVar-integrated CanIsoNet networks and no compatible transcriptome reference is available.

## Inputs

- Transcript-level TPM expression matrix per sample (e.g. from Kallisto quantification of disease and control FASTQ files from a GEO/ENA accession)
- Sample metadata grouping disease vs control samples
- 3did (v 2020_05) Pfam domain-domain interaction (DDI) pairs
- STRING (v10.0) protein-protein interaction data mapped to gene symbols
- ClinVar-annotated gene list

## Outputs

- Per-sample isoform switching event table with dMDTs and expression values (e.g. alzheimer_dmdt_events.csv)
- Annotated edge-list of disrupted isoform interactions with PFAM domain IDs per disease (e.g. disease_isoform_interaction_networks.tsv)
- Per-event results table with gene, participating isoforms, switching statistics, and interaction impact scores matching the CanIsoNet database schema

## How to apply

Quantify transcript/isoform abundances for every disease and control sample with Kallisto against the human reference transcriptome, or load an existing TPM matrix (e.g. GSE203206 for Alzheimer's) with pandas. For each gene and each sample, determine the Most Dominant Transcript (MDT) as the isoform with the highest TPM value using the CanIsoNet dMDT calling procedure. Compare dominant transcripts between disease and control samples to call disease-specific dMDTs and derive per-sample isoform switching events, then assemble the event table listing dMDTs with their expression values (e.g. alzheimer_dmdt_events.csv, mirroring the CanIsoNet Alzheimer's disease-specific page, Fig. 1A). For functional impact, intersect dMDT genes whose isoforms lose domains with 3did (v 2020_05) Pfam domain-domain interaction pairs and overlay the disrupted interactions onto the STRING (v10.0) PPI network per disease, annotating each disrupted edge with its PFAM domain IDs and highlighting ClinVar-annotated genes. Use the pipeline code released with CanIsoNet (github.com/kahramanlab/CanIsoNet_Web, CC-BY-4.0) so outputs conform to the schema of the 31 disease types hosted at caniso.net.

## Related tools

- **CanIsoNet** (Source of the dMDT calling procedure and isoform switching event framework; pipeline code at kahramanlab/CanIsoNet_Web defines the per-sample event table and database schema for 31 disease types) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Protein-protein interaction network (v10.0) onto which disrupted Pfam domain-domain interactions are overlaid to assess functional impact of each switching event)
- **Kallisto** (Quantifies transcript/isoform-level TPM abundances for every sample against the human reference transcriptome before dMDT calling)

## Examples

```
import pandas as pd; tpm = pd.read_csv('GSE203206_tpm_matrix.tsv', sep='\t', index_col=0); mdt = tpm.idxmax(axis=1)
```

## Evaluation signals

- The per-sample event table (e.g. alzheimer_dmdt_events.csv) exists, is well-formed, and every listed isoform switching event is supported by a per-sample dominant-transcript call consistent with the dMDTs on the Alzheimer's disease-specific CanIsoNet page (Fig. 1A).
- Results table contains the required fields: gene, participating isoforms, switching statistics, and impact scores, conforming to the CanIsoNet database schema.
- Exported network TSV contains PFAM domain ID annotations for disrupted interactions (field_present / contains_substring checks).
- Known event frequencies reproduce: the HSPH1-001 (858 AA) vs HSPH1-002 (814 AA) switching event appears among the top 10 most frequent dMDTs, observed in 21% of analysed Alzheimer's samples (GSE203206).
- Event and isoform counts are in the range annotated by CanIsoNet (11,811 isoforms and 14,357 unique isoform switching events across 31 disease types per the article; 11,528 isoforms per the README).

## Limitations

- Coverage is limited to the 31 pre-computed disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's RNA-seq datasets); new datasets require the full download/quantify/call pipeline and explicit request to the authors for upload ('Upon request, new datasets can be analysed and uploaded to the CanIsoNet database.').
- Functional impact predictions depend on specific reference versions (3did v 2020_05 and STRING v10.0) and will change with newer releases.
- The article and README report slightly different isoform annotation counts (11,811 vs 11,528 isoforms), so reproduced annotation totals may not exactly match either source.

## Evidence

- [abstract] Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases.: "Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases."
- [intro] 31 disease types covering 27 different cancer types from PCAWG project: "We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project"
- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [readme] It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases.: "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
- [readme] 4) Exploring the isoform switching event in each sample with their expression values.: "4) Exploring the isoform switching event in each sample with their expression values."
- [readme] CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network.: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network."
