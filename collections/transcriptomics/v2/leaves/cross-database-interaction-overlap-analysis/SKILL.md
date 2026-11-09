---
name: cross-database-interaction-overlap-analysis
description: Use when you have a FAVA (favapy) predicted protein-pair network with
  Pearson-correlation confidence scores and need to quantify how many of its high-confidence
  edges are novel vs. supported by STRING, hu.MAP 2.0, or the IDG Tdark catalogue.
when_to_use_negative:
- You need functional (directional/causal) regulatory edges — FAVA infers functional
  associations from co-variation in latent space, not pathway mechanisms
- Your network edges lack a correlation-based confidence score (e.g. edges from STRING
  or hu.MAP themselves), so the ≥70% cut-off cannot be applied
- The comparison reference is not STRING v11.5 or hu.MAP 2.0 and its edge identifiers
  are not mappable to the FAVA protein identifiers
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0121
tools:
- name: FAVA (favapy)
  role: Produces the predicted interaction network whose high-confidence edges are
    filtered at the '-c' confidence cut-off and then intersected with STRING v11.5,
    hu.MAP 2.0 and the IDG Tdark list
  repo: https://github.com/mikelkou/fava
- name: favapy
  role: Command-line entry point that generates the protein-pair output whose bidirectional
    duplicate rows must be deduplicated before overlap counting
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Used to filter pairs, deduplicate unordered pairs, perform the set intersections
    against STRING and hu.MAP 2.0, and compile the validation report CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:971447925c1f6b511a3eac196415352a50d351d1144d5b4ec03245197a470fa1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-database-interaction-overlap-analysis

## Summary

Validates a FAVA-predicted functional association network by intersecting its ≥70%-confidence edges against reference resources (STRING v11.5, hu.MAP 2.0) and the IDG Tdark understudied-protein set, reporting shared and absent edges/nodes as quantitative overlap counts.

## When to use

Use when you have a FAVA (favapy) predicted protein-pair network with Pearson-correlation confidence scores and need to quantify how many of its high-confidence edges are novel vs. supported by STRING, hu.MAP 2.0, or the IDG Tdark catalogue.

## When NOT to use

- You need functional (directional/causal) regulatory edges — FAVA infers functional associations from co-variation in latent space, not pathway mechanisms
- Your network edges lack a correlation-based confidence score (e.g. edges from STRING or hu.MAP themselves), so the ≥70% cut-off cannot be applied
- The comparison reference is not STRING v11.5 or hu.MAP 2.0 and its edge identifiers are not mappable to the FAVA protein identifiers

## Inputs

- FAVA-predicted interaction network from favapy (protein pairs with Pearson correlation confidence scores, bidirectional rows)
- IDG catalogue of 6000 Tdark (understudied) proteins
- STRING v11.5 protein interaction database
- hu.MAP 2.0 protein complex/network resource (648 proteins in the relevant overlap)

## Outputs

- Count of ≥70%-confidence FAVA interactions between Tdark and Tclin proteins (expected: 4210 edges over 1039 Tdark and 611 Tclin proteins)
- Count of high-confidence Tdark edges absent from STRING v11.5 (expected: 2747)
- Shared edges and nodes between the ≥70% FAVA network and hu.MAP 2.0 (expected: 1284 edges over 520 shared nodes)
- Protein-level overlap between the 1039 FAVA Tdark proteins and hu.MAP 2.0's 648 proteins (expected: 49 proteins)
- fava_network_validation_report.csv compiling all four comparison counts

## How to apply

First run the FAVA network inference and filter pairs with the Pearson-correlation confidence cut-off (the '-c' option; scores range from 1 for high correlation to -1 for high anti-correlation), keeping edges at ≥70% confidence. Because favapy emits bidirectional rows (proteinA–proteinB and proteinB–proteinA), deduplicate unordered protein pairs before any counting. Intersect the filtered network with the IDG list of 6000 Tdark proteins to count interactions linking understudied proteins to better-studied (Tclin) proteins, then compare the high-scoring Tdark edges against STRING v11.5 to identify edges absent from that reference database. Finally intersect the ≥70% network with hu.MAP 2.0 to compute shared edges and nodes plus the protein-level overlap, and compile all comparison counts into a single validation report (fava_network_validation_report.csv).

## Related tools

- **FAVA (favapy)** (Produces the predicted interaction network whose high-confidence edges are filtered at the '-c' confidence cut-off and then intersected with STRING v11.5, hu.MAP 2.0 and the IDG Tdark list) — https://github.com/mikelkou/fava
- **favapy** (Command-line entry point that generates the protein-pair output whose bidirectional duplicate rows must be deduplicated before overlap counting) — https://github.com/mikelkou/fava
- **Python** (Used to filter pairs, deduplicate unordered pairs, perform the set intersections against STRING and hu.MAP 2.0, and compile the validation report CSV)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- After deduplicating favapy's bidirectional rows, the count of ≥70%-confidence Tdark–Tclin interactions equals exactly 4210
- The number of high-confidence Tdark edges absent from STRING v11.5 equals 2747
- The FAVA/hu.MAP 2.0 intersection yields 1284 shared edges over 520 shared nodes
- The protein-level overlap between the 1039 Tdark proteins and hu.MAP 2.0's 648 proteins equals 49 proteins
- fava_network_validation_report.csv contains all four comparison counts as its rows

## Limitations

- favapy outputs bidirectional rows (proteinA–proteinB and proteinB–proteinA); if not deduplicated, all edge counts are inflated by roughly a factor of two
- Confidence scores range from 1 (high correlation) to -1 (high anti-correlation); edges with negative scores represent anti-correlation and should not be counted as positive associations
- The overlap counts are specific to the ≥70% confidence cut-off and the compared reference versions (STRING v11.5, hu.MAP 2.0); other thresholds or database versions will yield different counts

## Evidence

- [abstract] FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets"
- [abstract] FAVA predicted 4210 interactions between 1039 understudied proteins: "has predicted 4210 interactions between 1039 understudied proteins"
- [methods] Run FAVA from the command line: "Run FAVA from the command line as follows:"
- [intro] Calculating correlations in latent space: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
