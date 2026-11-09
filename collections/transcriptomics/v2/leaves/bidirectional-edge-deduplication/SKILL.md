---
name: bidirectional-edge-deduplication
description: Use when counting or intersecting edges from a FAVA (favapy) predicted
  interaction network — favapy emits each protein pair twice (in both directions),
  so raw-row counts inflate edge totals and must be deduplicated first.
when_to_use_negative:
- When the input edge list is already canonicalized undirected pairs (one row per
  unordered pair) — deduplication is a no-op and row counts are already valid
- When analyzing directed or signed relationships where proteinA-proteinB and proteinB-proteinA
  are semantically distinct — collapsing them would lose direction/orientation information
- When edges have not yet been filtered by the confidence cut-off; deduplicate after
  applying the >=70% Pearson correlation threshold
edam_topics:
- http://edamontology.org/topic_3407
tools:
- name: favapy
  role: Produces the FAVA-predicted interaction network whose bidirectional output
    rows (proteinA-proteinB and proteinB-proteinA) must be deduplicated before counting
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Used to canonicalize protein pairs, deduplicate rows, and intersect the filtered
    network with IDG Tdark, STRING v11.5, and hu.MAP 2.0
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:7b9d2fc4866626c34915d7383e1048611bf6b68f258f2523a5b18db5148cf6b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bidirectional-edge-deduplication

## Summary

Canonicalizes and deduplicates favapy's bidirectional output rows (proteinA-proteinB and proteinB-proteinA) into single undirected edges before counting or intersecting FAVA network edges with reference databases such as STRING v11.5 or hu.MAP 2.0.

## When to use

Use when counting or intersecting edges from a FAVA (favapy) predicted interaction network — favapy emits each protein pair twice (in both directions), so raw-row counts inflate edge totals and must be deduplicated first.

## When NOT to use

- When the input edge list is already canonicalized undirected pairs (one row per unordered pair) — deduplication is a no-op and row counts are already valid
- When analyzing directed or signed relationships where proteinA-proteinB and proteinB-proteinA are semantically distinct — collapsing them would lose direction/orientation information
- When edges have not yet been filtered by the confidence cut-off; deduplicate after applying the >=70% Pearson correlation threshold

## Inputs

- FAVA-predicted interaction network (favapy output) with per-pair Pearson correlation confidence scores in [-1, 1]
- IDG list of ~6000 Tdark (understudied) proteins
- STRING v11.5 reference interaction database
- hu.MAP 2.0 protein complex/network reference

## Outputs

- Deduplicated undirected edge list at >=70% confidence
- Count of 4210 Tdark–Tclin interactions (1039 understudied, 611 better-studied proteins)
- Count of 2747 high-scoring Tdark edges absent from STRING v11.5
- Shared-edge/node counts vs hu.MAP 2.0 (1284 edges over 520 shared nodes; 49 overlapping proteins)
- fava_network_validation_report.csv compiling the four comparison counts

## How to apply

Run favapy on the scRNA-seq/proteomics data and filter pairs with the Pearson correlation confidence cut-off (the '-c' option; scores range from 1 for high correlation to -1 for high anti-correlation), keeping edges at >=70% confidence. Deduplicate favapy's bidirectional output rows by canonicalizing each protein pair — e.g. sort the two protein identifiers into a fixed order — so proteinA-proteinB and proteinB-proteinA collapse to one edge, before counting or intersecting. Then intersect the deduplicated network with the IDG Tdark list to count interactions between the 1039 understudied proteins and 611 better-studied Tclin proteins, compare against STRING v11.5 to find edges absent from the reference, and intersect with hu.MAP 2.0 for shared edges and nodes. The rationale is that favapy's bidirectional row format makes naive row-level counting double-count every functional association, which would invalidate validation counts.

## Related tools

- **favapy** (Produces the FAVA-predicted interaction network whose bidirectional output rows (proteinA-proteinB and proteinB-proteinA) must be deduplicated before counting) — https://github.com/mikelkou/fava
- **Python** (Used to canonicalize protein pairs, deduplicate rows, and intersect the filtered network with IDG Tdark, STRING v11.5, and hu.MAP 2.0)

## Examples

```
favapy <path-to-data-file> <path-to-save-output> -c 0.7
```

## Evaluation signals

- After deduplication, the count of >=70%-confidence edges between the 1039 Tdark and 611 Tclin proteins equals exactly 4210
- Edges absent from STRING v11.5 among high-scoring Tdark interactions equal exactly 2747
- Intersection with hu.MAP 2.0 yields 1284 shared edges over 520 shared nodes
- Protein-level overlap between the 1039 Tdark proteins and hu.MAP 2.0's 648 proteins equals 49
- Every deduplicated edge has a unique canonical protein-pair key; no proteinB-proteinA mirror rows remain

## Limitations

- Deduplication must happen before counting/intersection; applying it after aggregating counts yields inflated numbers (roughly double) that will not reproduce the published 4210 interaction total
- Favapy confidence scores span -1 to 1 (anti-correlation to correlation); edges below the chosen cut-off must be excluded first or the deduplicated set will include anti-correlated pairs
- The verified counts (4210, 2747, 1284/520, 49) are specific to the >=70% confidence threshold and the cited STRING v11.5 and hu.MAP 2.0 reference versions

## Evidence

- [other] deduplicating favapy's bidirectional output rows (proteinA-proteinB and proteinB-proteinA) before counting: [evidence span withheld — anchor status: not_found]
- [abstract] FAVA predicted 4210 interactions between 1039 understudied proteins: "has predicted 4210 interactions between 1039 understudied proteins"
- [methods] filter pairs using the Pearson correlation confidence cut-off (the '-c' option; scores range from 1 for high correlation to -1 for high anti-correlation): [evidence span withheld — anchor status: not_found]
- [readme] You can install FAVA using pip: pip install favapy: [evidence span withheld — anchor status: not_found]
