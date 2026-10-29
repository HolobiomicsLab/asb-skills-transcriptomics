---
name: ari-clone-label-agreement-evaluation
description: Use when a per-cell clone labeling from a CNA-inference pipeline must
  be validated against an existing clone annotation, and a single agreement statistic
  (ARI) is needed to quantify concordance.
when_to_use_negative:
- No independent reference clone annotation exists for the sample — ARI cannot be
  computed without a second labeling to compare against
- The goal is de novo clonal reconstruction rather than validation of an existing
  reconstruction against a benchmark
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Produces the CNA probability tensor whose flattened, K-means-clustered labels
    (XClone_ID, via the XClustering analysis function) form the test labeling for
    the ARI comparison
  repo: https://github.com/single-cell-genetics/XClone
- name: CopyKAT
  role: Source of the reference Clone_ID annotation against which XClone_ID concordance
    is measured
- name: xcltk
  role: Preprocessing toolkit generating the RDR and BAF matrices that feed XClone
    prior to the clustering evaluation
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:9553e605dde9121f6cdf72eabf5241d3f2e7748b05b383201654408a2d09649e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - CopyKAT
  - XClone
  - xcltk
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ari-clone-label-agreement-evaluation

## Summary

Evaluates whether a computationally reconstructed subclonal structure (e.g., K-means labels on XClone's flattened CNA probability matrix, stored as XClone_ID) agrees with an independent reference clone annotation (CopyKAT-derived Clone_ID) by computing the adjusted Rand index, with 0.965 as the benchmark concordance for the TNBC1 sample.

## When to use

Use when a per-cell clone labeling from a CNA-inference pipeline must be validated against an existing clone annotation, and a single agreement statistic (ARI) is needed to quantify concordance.

## When NOT to use

- No independent reference clone annotation exists for the sample — ARI cannot be computed without a second labeling to compare against
- The goal is de novo clonal reconstruction rather than validation of an existing reconstruction against a benchmark

## Inputs

- Per-cell XClone_ID labels from K-means clustering (XClustering) on XClone's flattened CNA probability matrix
- Per-cell reference CopyKAT-derived Clone_ID annotation
- Cell barcodes shared between the XClone output AnnData and the reference annotation

## Outputs

- Adjusted Rand index (ARI) quantifying concordance between XClone_ID and CopyKAT Clone_ID
- Per-cell CSV table tnbc1_xclone_clone_labels.csv with columns: cell barcode, XClone_ID, CopyKAT Clone_ID, plus the ARI value

## How to apply

After running the XClone RDR, BAF, and Combine modules on the TNBC1 10x Genomics AnnData inputs, flatten the per-cell x per-gene CNA-state probability tensor and assign per-cell labels with K-means via XClone's XClustering analysis function, storing them as XClone_ID. Load the original CopyKAT-derived Clone_ID annotation as the reference. Compute the adjusted Rand index between the two label vectors; for the TNBC1 sample the target agreement is ARI 0.965, i.e., effectively perfect concordance. Persist the per-cell table (cell barcode, XClone_ID, CopyKAT Clone_ID) together with the ARI value to tnbc1_xclone_clone_labels.csv so the evaluation is auditable and reproducible.

## Related tools

- **XClone** (Produces the CNA probability tensor whose flattened, K-means-clustered labels (XClone_ID, via the XClustering analysis function) form the test labeling for the ARI comparison) — https://github.com/single-cell-genetics/XClone
- **CopyKAT** (Source of the reference Clone_ID annotation against which XClone_ID concordance is measured)
- **xcltk** (Preprocessing toolkit generating the RDR and BAF matrices that feed XClone prior to the clustering evaluation) — https://github.com/hxj5/xcltk

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(adata.obs['XClone_ID'], adata.obs['Clone_ID']); adata.obs[['XClone_ID','Clone_ID']].to_csv('tnbc1_xclone_clone_labels.csv')
```

## Evaluation signals

- ARI between XClone_ID and CopyKAT Clone_ID reaches the benchmark value 0.965 on the TNBC1 sample
- Every cell barcode in tnbc1_xclone_clone_labels.csv has both an XClone_ID and a CopyKAT Clone_ID (no unmatched cells)
- Number of K-means clusters on the flattened CNA probability tensor is consistent with the number of distinct CopyKAT clone labels
- Output CSV contains exactly the expected columns: cell barcode, XClone_ID, CopyKAT Clone_ID, plus the recorded ARI value

## Limitations

- ARI is reference-dependent: agreement is only meaningful against a trusted annotation, and CopyKAT itself does not consider allelic information and cannot detect copy-neutral LOH, so it may mislabel clone boundaries
- K-means clustering of the flattened CNA probability tensor can be sensitive to initialization and to the number of clusters chosen, affecting reproducibility of the ARI

## Evidence

- [other] K-means clustering on XClone's flattened CNA probability matrix yielded clonal reconstruction concordant with the original clone annotation at ARI 0.965: [evidence span withheld — anchor status: not_found]
- [methods] XClone clones via XClustering on the CNA states: "- [analysis] minor update in Clone function `XClustering`"
- [intro] CopyKAT ignores allelic information and cannot detect copy-neutral LOH: "both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [readme] XClone reconstructs clonal substructure from scRNA-seq by integrating RDR and BAF signals: [evidence span withheld — anchor status: external_doc]
