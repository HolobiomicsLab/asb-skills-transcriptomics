---
name: dark-protein-network-coverage-analysis
description: Use when a FAVA (favapy) interaction network must be validated for coverage
  of IDG Tdark understudied proteins, or when reporting how many novel dark-protein
  edges a de novo network adds beyond STRING or hu.MAP 2.0.
when_to_use_negative:
- The goal is to curate or benchmark a literature-derived PPI database (e.g. STRING
  itself) rather than assess coverage of dark proteins in a de novo omics-inferred
  network
- The input is a raw scRNA-seq or proteomics feature table that still needs network
  inference — run FAVA (favapy) first before this validation workflow
- Confidence scores are not Pearson-correlation-derived FAVA outputs (e.g. STRING
  combined scores), making the ≥70% '-c' cut-off semantics inapplicable
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: FAVA (favapy)
  role: Infers the functional association network from scRNA-seq/proteomics data and
    provides the '-c' confidence cut-off used to filter protein pairs before Tdark
    coverage counting
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure FAVA uses within the scverse ecosystem to consume the
    high-dimensional omics matrix
- name: Python
  role: Used for the fava.py module and to intersect, deduplicate, and count network
    edges against IDG, STRING v11.5, and hu.MAP 2.0
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:2544a2b5b0709f98d204a98c5f42e66e02c01461fb4b8d9b967c6da743917f74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# dark-protein-network-coverage-analysis

## Summary

Quantify how well a FAVA-predicted functional association network covers understudied (IDG Tdark) proteins by counting high-confidence Tdark–Tclin interactions and validating overlap against STRING v11.5 and hu.MAP 2.0. This addresses the literature bias where well-studied proteins accumulate far more interactions than dark proteins.

## When to use

Use when a FAVA (favapy) interaction network must be validated for coverage of IDG Tdark understudied proteins, or when reporting how many novel dark-protein edges a de novo network adds beyond STRING or hu.MAP 2.0.

## When NOT to use

- The goal is to curate or benchmark a literature-derived PPI database (e.g. STRING itself) rather than assess coverage of dark proteins in a de novo omics-inferred network
- The input is a raw scRNA-seq or proteomics feature table that still needs network inference — run FAVA (favapy) first before this validation workflow
- Confidence scores are not Pearson-correlation-derived FAVA outputs (e.g. STRING combined scores), making the ≥70% '-c' cut-off semantics inapplicable

## Inputs

- FAVA-predicted interaction network output from favapy (protein pairs with confidence scores in [-1, 1])
- IDG list of ~6000 Tdark (understudied) proteins
- STRING v11.5 protein interaction database (for novelty comparison)
- hu.MAP 2.0 protein complex/network dataset (for agreement analysis)

## Outputs

- Filtered ≥70%-confidence interaction set restricted to Tdark–Tclin protein pairs (4210 edges)
- Count of Tdark interactions absent from STRING v11.5 (2747 edges)
- Shared-edge and shared-node counts versus hu.MAP 2.0 (1284 edges over 520 nodes; 49 Tdark/hu.MAP protein overlap)
- fava_network_validation_report.csv compiling the four comparison counts

## How to apply

Run favapy on the scRNA-seq/proteomics data (Human Protein Atlas + PRIDE inputs via AnnData) to infer the combined functional association network, then filter protein pairs with the '-c' Pearson correlation confidence cut-off set to ≥70% (scores range from 1 for high correlation to -1 for high anti-correlation). Intersect the filtered network with the IDG list of ~6000 Tdark proteins, deduplicating favapy's bidirectional output rows (proteinA-proteinB and proteinB-proteinA) before counting; the expected result is 4210 interactions linking 1039 Tdark proteins to 611 better-studied (Tclin) proteins. Compare the high-scoring Tdark edges against STRING v11.5 to count edges absent from the reference database (expected 2747), then intersect with hu.MAP 2.0 to compute shared edges and nodes (expected 1284 edges over 520 shared nodes). Finally, compute the protein-level overlap between the 1039 Tdark proteins and hu.MAP 2.0's 648 proteins (expected 49 overlapping proteins) and compile all four counts into a validation report CSV.

## Related tools

- **FAVA (favapy)** (Infers the functional association network from scRNA-seq/proteomics data and provides the '-c' confidence cut-off used to filter protein pairs before Tdark coverage counting) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure FAVA uses within the scverse ecosystem to consume the high-dimensional omics matrix)
- **Python** (Used for the fava.py module and to intersect, deduplicate, and count network edges against IDG, STRING v11.5, and hu.MAP 2.0)

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- After deduplicating favapy's bidirectional rows (proteinA-proteinB and proteinB-proteinA), the ≥70%-confidence Tdark–Tclin edge count equals exactly 4210
- The count of high-scoring Tdark interactions absent from STRING v11.5 equals 2747
- Intersection with hu.MAP 2.0 yields 1284 shared edges over 520 shared nodes
- The protein-level overlap between the 1039 Tdark proteins and hu.MAP 2.0's 648 proteins equals 49
- fava_network_validation_report.csv exists and contains all four comparison counts

## Limitations

- favapy outputs bidirectional duplicate rows (proteinA-proteinB and proteinB-proteinA) that inflate counts unless deduplicated before counting
- Confidence scores span 1 (high correlation) to -1 (high anti-correlation); the ≥70% cut-off discards anti-correlated pairs, and other thresholds will change coverage counts
- Existing high-quality networks are heavily biased by data availability, so overlap with STRING/hu.MAP reflects both genuine novelty and coverage differences of the reference databases

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] well-studied proteins have many more interactions than understudied proteins: "most existing high-quality networks are heavily biased by data availability, in the sense that well-studied proteins have many more interactions than understudied proteins"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Run FAVA from the command line as follows: "Run FAVA from the command line as follows:"
- [readme] FAVA: Functional Associations using Variational Autoencoders: "FAVA: Functional Associations using Variational Autoencoders"
