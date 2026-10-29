---
name: eqtl-trained-logistic-model-training
description: Use when GrID-Net Granger scores for peak-gene pairs need distance-aware
  calibration before linking variants to genes, and an eQTL dataset (e.g., SHARE-seq
  skin) is available to define positive pairs.
when_to_use_negative:
- Peak-gene links are meant to be purely correlational or distance-independent — nearest-gene
  assignment is adequate and no distal interactions are of interest
- No eQTL or perturbation-derived positive training set is available for the tissue/assay,
  since the logistic model requires labeled regulatory pairs to fit
- Candidate pairs were not filtered by the required thresholds (genes in >= 1% of
  cells, peaks in >= 0.1% of cells, peaks within the distance window), in which case
  the training distribution is invalid
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: GrID-Net
  role: Supplies the Granger causal peak-gene scores that serve as the primary predictor
    feature in the logistic regression
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: 'Upstream: learns joint RNA+ATAC cell representations used to build the cell
    DAG from which GrID-Net scores are derived'
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:4a03128a47eefacfb7708ac86e67fc5fe2c8d9395c5a4e2dc147c8260de22079
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - PyTorch
  - Schema
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# eQTL-trained logistic model training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a logistic regression that combines GrID-Net Granger causal scores with log genomic distance to model distance-dependent regulatory effects, using significant SHARE-seq mouse skin eQTL peak-gene pairs (P < 1e-3) as positive training labels; the fitted model is then used to re-rank candidate peak-gene pairs when linking fine-mapped SCZ variants to genes.

## When to use

Use when GrID-Net Granger scores for peak-gene pairs need distance-aware calibration before linking variants to genes, and an eQTL dataset (e.g., SHARE-seq skin) is available to define positive pairs.

## When NOT to use

- Peak-gene links are meant to be purely correlational or distance-independent — nearest-gene assignment is adequate and no distal interactions are of interest
- No eQTL or perturbation-derived positive training set is available for the tissue/assay, since the logistic model requires labeled regulatory pairs to fit
- Candidate pairs were not filtered by the required thresholds (genes in >= 1% of cells, peaks in >= 0.1% of cells, peaks within the distance window), in which case the training distribution is invalid

## Inputs

- GrID-Net results DataFrame with Granger causality test statistics per candidate peak-gene pair (from gridnet.run_gridnet)
- SHARE-seq mouse skin eQTL peak-gene pairs with significance thresholds (positives: P < 1e-3)
- Genomic distances between each peak center and gene TSS for candidate pairs
- Fine-mapped SCZ variants (Trubetskoy et al., FINEMAP PP >= 0.05) overlapping 10x Multiome corticogenesis ATAC peaks

## Outputs

- Fitted logistic regression model over [GrID-Net score, log genomic distance]
- Distance-weighted scores for each variant-overlapping peak-gene pair
- variant_gene_links.csv: table linking each variant to up to 3 highest-scoring genes (top 50% of scores only)

## How to apply

After running gridnet.run_gridnet on candidate peak-gene pairs, fit a logistic regression predicting known regulatory pairs — significant SHARE-seq mouse skin eQTL peak-gene associations with P < 1e-3 — from two features: the raw GrID-Net Granger score and the log genomic distance between the peak and the gene TSS. This captures distance-dependent regulatory effects that raw Granger scores alone miss, so that distal but strongly causal links are not under-weighted relative to naive nearest-gene assignment. Apply the fitted model to score each variant-overlapping peak's genes, retain only peak-gene pairs in the top 50% of all scores, and link each fine-mapped variant (FINEMAP PP >= 0.05, overlapping a 10x Multiome ATAC peak) to up to 3 of its highest-scoring genes. The rationale is that eQTL-validated pairs mimic perturbation-based validation of the causal hypotheses GrID-Net infers.

## Related tools

- **GrID-Net** (Supplies the Granger causal peak-gene scores that serve as the primary predictor feature in the logistic regression) — https://github.com/alexw16/gridnet
- **Schema** (Upstream: learns joint RNA+ATAC cell representations used to build the cell DAG from which GrID-Net scores are derived) — https://github.com/rs239/schema

## Examples

```
from sklearn.linear_model import LogisticRegression; import numpy as np; y, s, d = eqtl_labels, gridnet_scores, np.log10(distances); clf = LogisticRegression().fit(np.c_[s, d], y); scored = clf.predict_proba(np.c_[s_var, d_var])[:, 1]
```

## Evaluation signals

- The linked gene set contains 132 candidate SCZ genes, matching the published result
- Enrichment for genes implicated in the original SCZ GWAS is reproduced with FDR = 0.037
- Variant coverage check: 30/78 (38%) of PP>50% variants linked vs 22/78 (28%) for baseline nearest-gene assignment (a ~36% relative increase)
- Agreement with the paper's shared-variant links: 9/12 variants assigned the same genes
- Only peak-gene pairs in the top 50% of distance-weighted scores are retained; each variant links to at most 3 genes

## Limitations

- GrID-Net associations are causal hypotheses that require perturbation validation; eQTL cross-referencing only mimics such validation
- The eQTL training set comes from mouse skin (SHARE-seq), so the learned distance-dependence may transfer imperfectly to human corticogenesis; corticogenesis data may also explain only a subset of SCZ phenotypes
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding; no Hi-C/CTCF topology information is incorporated

## Evidence
