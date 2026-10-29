---
name: network-validation-against-reference-databases
description: Use when you have a predicted protein interaction network with confidence
  scores (e.g. FAVA output filtered at ≥70% Pearson correlation confidence) and must
  quantify its novelty and agreement against STRING or hu.MAP 2.0.
when_to_use_negative:
- The input is not a functional association network with per-edge confidence scores
  (e.g. it is raw scRNA-seq expression data or a proteomics feature table — run FAVA
  inference first).
- The goal is to infer a new network from omics data rather than to validate or benchmark
  an existing predicted network against STRING or hu.MAP 2.0.
- No suitable reference database (STRING, hu.MAP, or an equivalent curated interaction
  set) exists for the organism being analyzed, making the overlap counts uninformative.
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: favapy
  role: Generates the FAVA-predicted interaction network whose bidirectional output
    rows must be deduplicated and filtered with the '-c' confidence cut-off before
    validation counting
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Used to intersect the filtered network with IDG Tdark, STRING v11.5, and hu.MAP
    2.0 lists and compile the validation report
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:a27f1d60aed1b87dda34286e01c505b7bd627a1b034c4ec48ecd5bc330f27b41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# network-validation-against-reference-databases

## Summary

Validation workflow that quantifies how a FAVA-predicted functional association network agrees with (or extends beyond) established protein-network references, using STRING v11.5 and hu.MAP 2.0 as ground-truth databases and reporting overlaps for understudied (IDG Tdark) proteins.

## When to use

Use when you have a predicted protein interaction network with confidence scores (e.g. FAVA output filtered at ≥70% Pearson correlation confidence) and must quantify its novelty and agreement against STRING or hu.MAP 2.0.

## When NOT to use

- The input is not a functional association network with per-edge confidence scores (e.g. it is raw scRNA-seq expression data or a proteomics feature table — run FAVA inference first).
- The goal is to infer a new network from omics data rather than to validate or benchmark an existing predicted network against STRING or hu.MAP 2.0.
- No suitable reference database (STRING, hu.MAP, or an equivalent curated interaction set) exists for the organism being analyzed, making the overlap counts uninformative.

## Inputs

- FAVA-predicted functional association network from favapy with Pearson correlation confidence scores (range -1 to 1)
- IDG list of 6000 Tdark understudied proteins
- STRING v11.5 protein interaction database
- hu.MAP 2.0 protein complex list

## Outputs

- Filtered FAVA network retaining edges at ≥70% confidence
- Count of Tdark-to-better-studied (Tclin) interactions (expected 4210 edges between 1039 Tdark and 611 Tclin proteins)
- Count of high-scoring Tdark edges absent from STRING v11.5 (expected 2747)
- Shared edges and nodes with hu.MAP 2.0 (expected 1284 edges over 520 shared nodes)
- Protein-level overlap between 1039 Tdark proteins and hu.MAP 2.0's 648 proteins (expected 49)
- fava_network_validation_report.csv compiling all four comparison counts

## How to apply

Run FAVA (favapy) to infer the combined scRNA-seq + bulk proteomics network, then filter edges with the '-c' Pearson correlation confidence cut-off (scores range from 1 for high correlation to -1 for high anti-correlation), keeping only edges at ≥70% confidence. Deduplicate favapy's bidirectional output rows (proteinA-proteinB and proteinB-proteinA) before any counting. Intersect the filtered network with the IDG list of 6000 Tdark understudied proteins to count Tdark-to-better-studied (Tclin) interactions, then compare high-scoring Tdark edges against STRING v11.5 to identify edges absent from the reference database. Separately intersect the ≥70% confidence network with hu.MAP 2.0 to compute shared edges and nodes, and compute the protein-level overlap between Tdark proteins and hu.MAP 2.0 proteins. Compile all four comparison counts into fava_network_validation_report.csv; the rationale is that FAVA networks, being inferred from omics rather than literature, should better cover understudied proteins than biased reference databases.

## Related tools

- **favapy** (Generates the FAVA-predicted interaction network whose bidirectional output rows must be deduplicated and filtered with the '-c' confidence cut-off before validation counting) — https://github.com/mikelkou/fava
- **Python** (Used to intersect the filtered network with IDG Tdark, STRING v11.5, and hu.MAP 2.0 lists and compile the validation report)

## Examples

```
favapy <path-to-data-file> <path-to-save-output> -c 0.7
```

## Evaluation signals

- After deduplicating bidirectional favapy rows, the Tdark–Tclin edge count at ≥70% confidence equals 4210.
- Comparison against STRING v11.5 yields exactly 2747 Tdark edges absent from the reference database.
- Intersection with hu.MAP 2.0 yields 1284 shared edges over 520 shared nodes.
- Protein-level overlap between the 1039 Tdark proteins and hu.MAP 2.0's 648 proteins equals 49.
- All four counts are present in fava_network_validation_report.csv.

## Limitations

- favapy outputs both proteinA-proteinB and proteinB-proteinA rows for each pair, so naive counting inflates edge counts unless bidirectional rows are deduplicated first.
- FAVA networks are inferred from omics data and should be free of literature bias, but validation against STRING and hu.MAP 2.0 mixes edges supported by curated evidence with purely predicted ones.
- Coverage is asymmetric: existing high-quality networks are heavily biased by data availability, so agreement with reference databases will be lower for understudied (Tdark) proteins.

## Evidence

- [abstract] FAVA predicted 4210 interactions between 1039 understudied proteins: "has predicted 4210 interactions between 1039 understudied proteins"
- [intro] Existing networks are biased by data availability, motivating validation focused on understudied proteins: "most existing high-quality networks are heavily biased by data availability, in the sense that well-studied proteins have many more interactions than understudied proteins"
- [methods] FAVA runs from the command line to produce the network that is then validated: "Run FAVA from the command line as follows:"
- [readme] Favapy is installed via pip as the Python package used to run FAVA: "You can install FAVA using pip:"
