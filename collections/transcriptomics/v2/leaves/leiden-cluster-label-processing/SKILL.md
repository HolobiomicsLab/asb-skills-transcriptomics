---
name: leiden-cluster-label-processing
description: Use when scRNA-seq count data must be converted to log-cpm normalised
  AnnData with tentative Leiden cluster labels as the required input format for Cytocipher
  code-scoring and cluster merging.
when_to_use_negative:
- Data is already log-cpm normalised AnnData with cluster labels — skip straight to
  cc.tl.get_markers
- Input is a cell-by-cluster enrichment score matrix or feature table rather than
  raw per-cell expression counts
- The analysis goal requires a single fixed clustering rather than over-clustering
  followed by significance-based merging
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Standard workflow used to produce log-cpm normalised data from raw counts
- name: AnnData
  role: Container format holding the normalised counts and tentative Leiden cluster
    labels consumed by Cytocipher
- name: cytocipher
  role: Downstream package (cc.tl.get_markers, cc.tl.code_enrich) that consumes the
    over-clustered labels
  repo: https://github.com/BradBalderson/Cytocipher
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:57c8debedbbff0c1a1a654c280baac3f8fba2548eb02d084d9122a9fcc2c1916
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# leiden-cluster-label-processing

## Summary

Prepare an AnnData object for Cytocipher by processing scRNA-seq data through the scanpy standard workflow to log-cpm normalised counts and attaching tentative Leiden cluster labels, deliberately at high resolution so the data is over-clustered before significance-based merging.

## When to use

Use when scRNA-seq count data must be converted to log-cpm normalised AnnData with tentative Leiden cluster labels as the required input format for Cytocipher code-scoring and cluster merging.

## When NOT to use

- Data is already log-cpm normalised AnnData with cluster labels — skip straight to cc.tl.get_markers
- Input is a cell-by-cluster enrichment score matrix or feature table rather than raw per-cell expression counts
- The analysis goal requires a single fixed clustering rather than over-clustering followed by significance-based merging

## Inputs

- Raw single-cell RNA-seq count matrix loadable into an AnnData object
- Leiden clustering parameters (resolution set high to induce over-clustering)

## Outputs

- AnnData object with log-cpm normalised counts and tentative Leiden cluster labels (e.g. data.obs['leiden'])
- Over-clustered cell partition ready for cc.tl.get_markers and cc.tl.code_enrich

## How to apply

Process raw counts through the scanpy standard workflow (as in the PBMC3k tutorial) to produce log-cpm normalised data stored in an AnnData object, then run Leiden clustering to attach tentative cluster labels (e.g. under a 'leiden' key). Deliberately use a high Leiden resolution so that the data is substantially over-clustered, because Cytocipher's design expects non-significantly different clusters to be merged downstream. This over-clustered AnnData is then the input to cc.tl.get_markers and cc.tl.code_enrich, which score each cell against every cluster's marker gene sets. The winsorisation function f is applied during marker identification to bound expression values.

## Related tools

- **scanpy** (Standard workflow used to produce log-cpm normalised data from raw counts)
- **AnnData** (Container format holding the normalised counts and tentative Leiden cluster labels consumed by Cytocipher)
- **cytocipher** (Downstream package (cc.tl.get_markers, cc.tl.code_enrich) that consumes the over-clustered labels) — https://github.com/BradBalderson/Cytocipher

## Examples

```
import scanpy as sc; data = sc.read_h5ad('raw.h5ad'); sc.pp.normalize_total(data, target_sum=1e6); sc.pp.log1p(data); sc.tl.leiden(data, resolution=1.5, key_added='leiden')
```

## Evaluation signals

- AnnData contains log-cpm normalised values (not raw counts) in the expression layer expected by Cytocipher
- data.obs holds a 'leiden' key with tentative cluster labels usable by cc.tl.get_markers(data, 'leiden')
- Clustering is deliberately over-clustered: number of Leiden clusters is high (per README guidance that 'It's better if the Leiden resolution is high')
- Downstream diagnostics run cleanly: cc.pl.sig_cluster_diagnostics on a non-significant pair yields p-values near 1 (e.g. p=0.913 and p=0.813 in the documented example)

## Limitations

- The README specifies the scanpy PBMC3k-style standard workflow as the expected preprocessing; deviations from log-cpm normalisation may not match Cytocipher's assumptions
- Marker identification applies the winsorisation function f to bound expression values, so extreme expression behaviour is clipped rather than modelled
- No discussion-section content is available to confirm behaviour on very small clusters or sparse label assignments

## Evidence

- [readme] AnnData processed with scanpy standard workflow and high-resolution Leiden: [evidence span withheld — anchor status: not_found]
- [readme] Over-clustering rationale: "It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [methods] Workflow functions consuming the prepared labels: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] Validation of downstream diagnostics on prepared data: "the following visualises the scores as violin plots of the enrichment scores & prints the p-values determined by comparing the scores"
