---
name: protein-interaction-network-confidence-filtering
description: Use when a FAVA (favapy) predicted protein association network must be
  reduced to high-confidence edges before counting or validating interactions, e.g.
  against STRING, hu.MAP, or the IDG Tdark list.
when_to_use_negative:
- When the goal is to generate the network itself from raw scRNA-seq/proteomics matrices
  rather than threshold and validate an already-predicted network
- When a confidence threshold other than the article's ≥70% Pearson correlation cut-off
  is required, since the counts (4210 interactions, 2747 absent from STRING, 1284
  shared hu.MAP edges) are specific to that cutoff
- When the reference comparison target is not a curated protein interaction database
  (STRING, hu.MAP) or the IDG dark-protein list
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: FAVA (favapy)
  role: Produces the predicted functional association network and exposes the pairs_after_cutoff
    / '-c' confidence cut-off used to retain ≥70% confidence edges
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Used to intersect the filtered network with IDG Tdark, STRING v11.5, and hu.MAP
    2.0, deduplicate bidirectional pairs, and compile the validation report
- name: STRING v11.5
  role: Reference interaction database used to identify high-scoring FAVA Tdark edges
    absent from existing curated networks (2747 edges)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:b328dcfedb03f9a0a7358bd7f3e1aaf305c9b0b66ae4a8e5a8a3eabc28408529
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - FAVA (favapy)
  - Python
  - STRING v11.5
  - favapy
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# protein-interaction-network-confidence-filtering

## Summary

Filter the FAVA-predicted functional association network (built from Human Protein Atlas scRNA-seq and PRIDE proteomics data) at a ≥70% Pearson correlation confidence cut-off using favapy's pairs_after_cutoff, then validate the retained edges against STRING v11.5 and hu.MAP 2.0 for the IDG Tdark understudied-protein analysis.

## When to use

Use when a FAVA (favapy) predicted protein association network must be reduced to high-confidence edges before counting or validating interactions, e.g. against STRING, hu.MAP, or the IDG Tdark list.

## When NOT to use

- When the goal is to generate the network itself from raw scRNA-seq/proteomics matrices rather than threshold and validate an already-predicted network
- When a confidence threshold other than the article's ≥70% Pearson correlation cut-off is required, since the counts (4210 interactions, 2747 absent from STRING, 1284 shared hu.MAP edges) are specific to that cutoff
- When the reference comparison target is not a curated protein interaction database (STRING, hu.MAP) or the IDG dark-protein list

## Inputs

- FAVA-predicted interaction network (favapy output of protein pairs with Pearson correlation confidence scores ranging from 1 to -1)
- IDG list of ~6000 Tdark understudied proteins
- STRING v11.5 reference protein interaction database
- hu.MAP 2.0 protein complex/network reference

## Outputs

- Filtered edge list at ≥70% confidence (deduplicated protein pairs)
- Counts of Tdark-to-better-studied interactions, edges absent from STRING v11.5, and shared edges/nodes with hu.MAP 2.0
- fava_network_validation_report.csv compiling the four comparison counts

## How to apply

Run favapy to produce the predicted interaction network, then apply the confidence cut-off via the '-c' option, where correlation scores range from 1 (high correlation) to -1 (high anti-correlation), keeping edges at ≥70% confidence (the article's pairs_after_cutoff step). Because favapy emits bidirectional rows (proteinA-proteinB and proteinB-proteinA), deduplicate pairs before counting. Intersect the filtered network with the IDG Tdark list to count Tdark-to-better-studied interactions, then compare the retained edges to STRING v11.5 and hu.MAP 2.0 to quantify novel edges and shared nodes. The rationale is that computing correlations in FAVA's compressed latent space — rather than on the sparse, redundant raw omics matrices — yields higher-quality associations that can then be thresholded into a literature-unbiased network.

## Related tools

- **FAVA (favapy)** (Produces the predicted functional association network and exposes the pairs_after_cutoff / '-c' confidence cut-off used to retain ≥70% confidence edges) — https://github.com/mikelkou/fava
- **Python** (Used to intersect the filtered network with IDG Tdark, STRING v11.5, and hu.MAP 2.0, deduplicate bidirectional pairs, and compile the validation report)
- **STRING v11.5** (Reference interaction database used to identify high-scoring FAVA Tdark edges absent from existing curated networks (2747 edges))

## Examples

```
favapy <path-to-data-file> <path-to-save-output> -c 0.70
```

## Evaluation signals

- After deduplicating favapy's bidirectional proteinA-proteinB / proteinB-proteinA rows, the ≥70%-confidence Tdark-vs-better-studied interaction count equals 4210
- Comparison against STRING v11.5 yields exactly 2747 high-confidence edges absent from the reference database
- Intersection with hu.MAP 2.0 yields 1284 shared edges over 520 shared nodes
- Protein-level overlap between the 1039 Tdark proteins and hu.MAP 2.0's 648 proteins equals 49
- All four comparison counts are present in fava_network_validation_report.csv

## Limitations

- Counts are threshold-specific: the 4210/2747/1284/49 validation numbers hold only at the ≥70% confidence cut-off and change if '-c' is altered
- favapy outputs bidirectional pair rows that inflate counts unless deduplicated before any edge statistics are computed
- FAVA networks reflect correlations in omics data and are literature-unbiased, so agreement with curated databases like STRING is partial by design; missing edges are not necessarily false positives

## Evidence

- [abstract] FAVA predicted 4210 interactions between 1039 understudied proteins: "has predicted 4210 interactions between 1039 understudied proteins"
- [intro] latent-space correlation rationale: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] CLI entry point: "Run FAVA from the command line as follows:"
- [abstract] scverse input format: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
