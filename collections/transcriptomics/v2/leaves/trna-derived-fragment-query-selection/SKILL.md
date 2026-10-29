---
name: trna-derived-fragment-query-selection
description: Use when you have a cataloged plant tncRNA pool and need to choose conserved/common
  tRF-5 and tRF-3 sequences (18–25 nt) as queries for miRNA-like target prediction
  workflows.
when_to_use_negative:
- Input is raw sRNA-seq FASTQ reads rather than an already-identified and classified
  tncRNA pool — use the tncRNA identification/classification pipeline first
- You need tncRNA subtype classification (tRF-1, tRNA halves) or quantification rather
  than target-prediction query selection
- The organism lacks psRNATarget-supported cDNA/transposon reference sequences or
  the goal is not miRNA-like target prediction
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: psRNATarget
  role: Predicts putative mRNA targets of the selected common tncRNA (tRF-5/tRF-3,
    18–25 nt) query sequences against Arabidopsis cDNA and transposon sequences
- name: g:Profiler
  role: Performs pathway enrichment analysis on the protein target list extracted
    from the psRNATarget output
- name: Cytoscape
  role: Visualizes enriched terms using the EnrichmentMap and AutoAnnotate apps
- name: tncRNA Toolkit
  role: Produces the upstream tncRNA pool (subtype-labeled sequences with RPM values)
    from which common tRF-5/tRF-3 queries are selected
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:836f060fdeeca1d8763e18b7e917209e0e741f7d8e58882936c2d55e9cb68103
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
  - Cytoscape
  - NCBI SRA Toolkit (v2.10)
  - biopython
  - g:Profiler
  - pandas
  - psRNATarget
  - samtools
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# trna-derived-fragment-query-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects common plant tncRNAs (tRF-5 and tRF-3 series, 18–25 nt) from an identified tncRNA pool as query sequences for downstream psRNATarget target prediction and g:Profiler pathway enrichment.

## When to use

Use when you have a cataloged plant tncRNA pool and need to choose conserved/common tRF-5 and tRF-3 sequences (18–25 nt) as queries for miRNA-like target prediction workflows.

## When NOT to use

- Input is raw sRNA-seq FASTQ reads rather than an already-identified and classified tncRNA pool — use the tncRNA identification/classification pipeline first
- You need tncRNA subtype classification (tRF-1, tRNA halves) or quantification rather than target-prediction query selection
- The organism lacks psRNATarget-supported cDNA/transposon reference sequences or the goal is not miRNA-like target prediction

## Inputs

- Identified tncRNA catalog (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, tRNA half subtypes) with sequences, subtype labels, and RPM values
- Arabidopsis thaliana cDNA and transposon reference sequences (e.g. TAIR10.1-derived)

## Outputs

- Filtered FASTA of common tncRNA query sequences (tRF-5 and tRF-3 series, 18–25 nt)
- psRNATarget predicted target list (mRNA targets of the common tncRNA queries)

## How to apply

From the identified tncRNA pool (tRF-5s, tRF-3s, tRF-1, tRNA halves), restrict the candidate set to the common tncRNAs belonging to the tRF-5 and tRF-3 series and filter by length to 18–25 nt. Use these sequences as the query input to psRNATarget (2017 release, default parameters) against Arabidopsis cDNA and transposon sequences to predict putative mRNA targets. The rationale is that tRF-5 and tRF-3 fragments of this length share the size and 5'-seed characteristics needed for the psRNATarget miRNA-like target prediction model, whereas other subtypes (tRF-1, tRNA halves) are not used as queries in this workflow. Extract the predicted protein targets from the psRNATarget output and pass that list to g:Profiler for pathway enrichment, with enrichment results visualized in Cytoscape (EnrichmentMap, AutoAnnotate).

## Related tools

- **psRNATarget** (Predicts putative mRNA targets of the selected common tncRNA (tRF-5/tRF-3, 18–25 nt) query sequences against Arabidopsis cDNA and transposon sequences)
- **g:Profiler** (Performs pathway enrichment analysis on the protein target list extracted from the psRNATarget output)
- **Cytoscape** (Visualizes enriched terms using the EnrichmentMap and AutoAnnotate apps)
- **tncRNA Toolkit** (Produces the upstream tncRNA pool (subtype-labeled sequences with RPM values) from which common tRF-5/tRF-3 queries are selected) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
psRNATarget -i common_tRFs_18-25nt.fa -d ath_cDNA -p plant -a O -e -4 -v 4.5 -m 0 -q 0
```

## Evaluation signals

- All selected queries are 18–25 nt in length
- All selected queries are exclusively tRF-5 or tRF-3 series; no tRF-1 or tRNA-half sequences enter the query set
- psRNATarget run used the 2017 release with default parameters
- Resulting enrichment analysis shows targets majorly enriched in metabolic and developmental processes, consistent with Supplementary Fig. 13
- Predicted protein target list is extractable from the psRNATarget output and is accepted by g:Profiler without format errors

## Limitations

- Only common tncRNAs of the tRF-5 and tRF-3 series (18–25 nt) are used as queries; other tncRNA subtypes are excluded from this target-prediction workflow
- Predicted tncRNA targets are computational predictions and require experimental validation
- Pathway enrichment depends on the quality of the psRNATarget prediction; the article notes that further validation of putative targets will be needed to understand regulatory mechanisms

## Evidence

- [supplementary] Predicted targets of conserved tRF-5 sequences enriched in growth/development, transport, metabolism (Supplementary Fig. 13): [evidence span withheld — anchor status: not_found]
