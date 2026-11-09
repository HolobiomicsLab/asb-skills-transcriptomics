---
name: te-vs-gene-clustering-agreement-ari-nmi
description: Use when you have independently clustered the same cells from a TE count
  matrix and a gene expression matrix and need to measure concordance, or when validating
  that a TE quantification method (e.g., MATES) reproduces known cell states like
  the mouse 2CLC population.
when_to_use_negative:
- You only have a subfamily-level TE count matrix and want locus-resolution concordance
  — aggregating at subfamily level masks locus-specific signal
- TE and gene measurements come from different cells or different barcodes, making
  label-vector comparison meaningless
- The biological population of interest is not expected to be marked by TE expression
  (MATES' own validation relies on TE-marked states such as 2CLCs)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATES
  role: Produces the locus-specific TE count matrix on which the TE-based clustering
    is performed
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Computes per-locus counts URi + ceil(αi × MRi) that feed the TE clustering
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:58603159cab0f7eee3b7db15797e4de426f5898f96526c435dd9f737599e2d38
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - TE_quantifier
  - TE_quantifier_Intronic
  - bam_processor
  - data_processor
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# TE-vs-gene clustering agreement via ARI and NMI

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well single-cell clusters built from a MATES locus-specific TE count matrix agree with clusters built from standard gene expression by computing adjusted Rand index (ARI) and normalized mutual information (NMI), validating that TE-level signal recapitulates biology (e.g., the 2CLC population, reported medians ARI 0.397 / NMI 0.496).

## When to use

Use when you have independently clustered the same cells from a TE count matrix and a gene expression matrix and need to measure concordance, or when validating that a TE quantification method (e.g., MATES) reproduces known cell states like the mouse 2CLC population.

## When NOT to use

- You only have a subfamily-level TE count matrix and want locus-resolution concordance — aggregating at subfamily level masks locus-specific signal
- TE and gene measurements come from different cells or different barcodes, making label-vector comparison meaningless
- The biological population of interest is not expected to be marked by TE expression (MATES' own validation relies on TE-marked states such as 2CLCs)

## Inputs

- MATES locus-level TE count matrix (cells x TE loci, from TE_quantifier)
- Gene expression matrix from the same cells (e.g., STARsolo 10x count matrix)
- Cluster label vectors from Louvain/Leiden on each matrix, across multiple resolutions

## Outputs

- Per-resolution ARI and NMI values between TE-based and gene-based clusterings
- Comparison of locus-level vs subfamily-level ARI (e.g., Fig. 2h box plot)
- Identified cell population (2CLC) with TE markers MERVL-int and MT2_Mm

## How to apply

Cluster cells separately on the MATES TE count matrix and on the gene expression matrix using Louvain/Leiden over multiple resolutions. For each resolution, compute the adjusted Rand index and normalized mutual information between the TE-based and gene-based cluster label vectors. Compare the locus-level ARI/NMI against the subfamily-level values (box plot across resolutions), since locus-specific assignment is the paper's claim to improve over subfamily-level TE aggregation. Accept the analysis when the locus-level median matches the reported ARI 0.397 (NMI 0.496) and the TE-driven clusters recover the expected population — cells scoring positive for Zscan4c/Zscan4d with enriched MT2_Mm and MERVL-int flanking the Zscan4 loci (binomial test).

## Related tools

- **MATES** (Produces the locus-specific TE count matrix on which the TE-based clustering is performed) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Computes per-locus counts URi + ceil(αi × MRi) that feed the TE clustering) — https://github.com/mcgilldinglab/MATES

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score; ari = adjusted_rand_score(te_labels, gene_labels); nmi = normalized_mutual_info_score(te_labels, gene_labels)
```

## Evaluation signals

- Locus-level ARI median across clustering resolutions approximates the reported 0.397
- NMI median approximates the reported 0.496
- The TE-driven cluster co-expresses Zscan4c/Zscan4d
- MT2_Mm and MERVL-int reads are enriched flanking Zscan4c/Zscan4d loci (binomial test)
- Locus-level ARI exceeds subfamily-level ARI in the resolution sweep (Fig. 2h comparison)

## Limitations

- MATES requires more computational resources such as GPUs than scTE or SoloTE before clustering can be attempted
- Multi-mapping rate estimates can be inaccurate for TEs longer than the 2001 bp coverage vector, weakening locus-level counts for long elements like L1HS
- Identical TE copies (e.g., HERVK isoforms) cannot be distinguished when flanking regions lack uniquely mappable sequence, affecting locus-level cluster markers
- Agreement metrics depend on clustering resolution choice; the paper reports medians across resolutions rather than a single value

## Evidence

- [other] Reported ARI/NMI targets for locus-level vs gene clustering: [evidence span withheld — anchor status: not_found]
- [other] Clustering procedure on MATES TE matrix: [evidence span withheld — anchor status: not_found]
- [readme] Rationale for locus-level quantification improving clustering: [evidence span withheld — anchor status: external_doc]
