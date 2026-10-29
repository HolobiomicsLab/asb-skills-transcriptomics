---
name: cluster-rank-selection-via-cpcc-dispersion
description: Use when running NMF (or similar consensus clustering) on a candidate-regulator
  feature matrix and you must justify the chosen cluster count via consensus-based
  stability indices like CPCC and dispersion.
when_to_use_negative:
- Your input is not a numeric feature matrix of candidate regulators — e.g. raw TCGA/MiOncoCirc
  files or RNAct interactome data that have not yet been reduced to MATRIX.txt via
  the GET_* scripts
- You only need to score candidate RBPs individually (e.g. C value or Zp alone) without
  an unsupervised grouping step
- You lack multiple NMF runs per rank, so consensus-based CPCC/dispersion indices
  cannot be computed reliably
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CRIT
  role: Pipeline whose 'cluster identify' component applies NMF to the features matrix
    and ranks clusters using CPCC and dispersion indices
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: NMF
  role: R package implementing the non-negative matrix factorization used to cluster
    candidate RBPs at varying ranks
  repo: https://cran.r-project.org/web/packages/NMF/NMF.pdf
- name: R
  role: Language in which NMF.R and the Fisher's exact cluster-enrichment tests are
    implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:0d83feb4e884ff5b0139454995904e47d4023abb889b816c52e990f655056cc8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - NMF
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cluster-rank-selection-via-cpcc-dispersion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the number of NMF clusters (factorization rank R) for the CRIT feature matrix by picking the rank at local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices. In the CRIT paper this yielded R=36 clusters of 1344 candidate RBPs, from which cluster 23 was found enriched for gold-standard circRNA regulators.

## When to use

Use when running NMF (or similar consensus clustering) on a candidate-regulator feature matrix and you must justify the chosen cluster count via consensus-based stability indices like CPCC and dispersion.

## When NOT to use

- Your input is not a numeric feature matrix of candidate regulators — e.g. raw TCGA/MiOncoCirc files or RNAct interactome data that have not yet been reduced to MATRIX.txt via the GET_* scripts
- You only need to score candidate RBPs individually (e.g. C value or Zp alone) without an unsupervised grouping step
- You lack multiple NMF runs per rank, so consensus-based CPCC/dispersion indices cannot be computed reliably

## Inputs

- MATRIX.txt: merged candidate RBP feature matrix (ρm, Zp, Zs, Zg, C value columns)
- RBP_list.RData: list of 1344 candidate RBPs (Ensembl gene id, Hgnc symbol, Entrez gene id, UniProt SwissProt identifiers)
- gold_regulators.xlsx: list of gold-standard circRNA regulators for downstream enrichment testing

## Outputs

- NMFgroup_XXclusterYYrun.txt: NMF cluster assignments for each cluster count XX and run YY
- Selected factorization rank R (expected 36 in the CRIT analysis)
- Cluster 23 membership: 90 RBPs, 17 of which are gold-standard circRNA regulators (Fisher's exact p = 0.00012)

## How to apply

Merge the CRIT features (ρm, Zp, Zs, Zg, C value) into MATRIX.txt for the 1344 candidate RBPs listed in RBP_list.RData. Run the NMF algorithm repeatedly (script NMF.R from BioinformaticsSTU/CRIT) across a sweep of cluster counts and multiple runs per count, writing each assignment to NMFgroup_XXclusterYYrun.txt (XX = number of clusters, YY = run number). For each rank, compute consensus-based stability metrics — the cophenetic correlation coefficient (CPCC) and the dispersion index (DC) — and select the rank at their local maxima; in the published analysis this selected R=36. With rank 36 fixed, identify the gold-regulator-enriched cluster (expected: cluster 23 with 90 RBPs, 17 gold regulators, Fisher's exact p = 0.00012).

## Related tools

- **CRIT** (Pipeline whose 'cluster identify' component applies NMF to the features matrix and ranks clusters using CPCC and dispersion indices) — https://github.com/BioinformaticsSTU/CRIT
- **NMF** (R package implementing the non-negative matrix factorization used to cluster candidate RBPs at varying ranks) — https://cran.r-project.org/web/packages/NMF/NMF.pdf
- **R** (Language in which NMF.R and the Fisher's exact cluster-enrichment tests are implemented)

## Examples

```
Rscript NMF.R  # after sourcing CRIT_settings.R with rank sweep; writes NMFgroup_36cluster1run.txt .. NMFgroup_36cluster36run.txt
```

## Evaluation signals

- The chosen rank is at a local maximum of both the cophenetic correlation coefficient (CPCC) and the dispersion (DC) index curves (expected R=36 in the published analysis)
- All cluster assignments are written as NMFgroup_XXclusterYYrun.txt files following the XXclusterYYrun naming convention
- The gold-regulator-enriched cluster (cluster 23 at R=36) contains exactly 90 RBPs of which 17 are gold-standard circRNA regulators
- Fisher's exact p-values match reported values: 0.00012 (gold regulators), 0.00201 (Li et al. 103-RBP circmCherry screen, 13/90 overlap), 0.00112 (19 curated established regulators, 6/19 overlap)

## Limitations

- The rank choice depends on CPCC/DC local maxima, which are consensus statistics requiring many NMF runs per rank and may be unstable for small numbers of runs
- Enrichment of the identified cluster depends on the supplied gold_regulators.xlsx and the 1344-RBP list; substituting a different gene list changes the expected cluster membership and p-values
- The CRIT framework and this clustering strategy were developed and validated in cancer datasets; generalization to other diseases (e.g. neurodegenerative disease) is untested

## Evidence
