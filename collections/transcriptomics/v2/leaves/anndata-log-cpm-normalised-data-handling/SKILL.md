---
name: anndata-log-cpm-normalised-data-handling
description: Use when loading a scRNA-seq dataset (e.g. .h5ad AnnData) in scanpy/Jupyter
  before running Cytocipher, and you must confirm the matrix is log-cpm normalised
  and carries tentative Leiden cluster labels, preferably at high resolution.
when_to_use_negative:
- Input is raw counts not yet log-cpm normalised, or lacks any tentative cluster labels
  — Cytocipher expects log-cpm data with an existing over-clustered Leiden column
- The clustering is already at low resolution with few clusters and little over-clustering
  — the README advises high resolution so that there is a lot of over-clustering for
  merging
- The data are not single-cell RNA-seq AnnData/scanpy-compatible objects (Cytocipher
  is scverse-compatible and expects AnnData)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Cytocipher
  role: consumes the log-cpm normalised AnnData with 'leiden' labels for marker identification,
    code scoring, and cluster merging
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: reference standard workflow used to preprocess the AnnData into log-cpm normalised
    form with Leiden cluster labels
- name: AnnData
  role: container format of the input object passed as 'data' to Cytocipher functions
- name: Jupyter
  role: notebook environment where Cytocipher functions are invoked and documentation
    inspected (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:b45593a2ffd6a23f1a03e6edbbb6caba3e28e5846998ab94fdfba2ece8acdfd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# anndata-log-cpm-normalised-data-handling

## Summary

Prepares and validates an AnnData object processed via the scanpy standard workflow into log-cpm normalised form with tentative (over-clustered) Leiden labels, which is the required input format for Cytocipher's marker identification, code scoring, and cluster-merge testing.

## When to use

Use when loading a scRNA-seq dataset (e.g. .h5ad AnnData) in scanpy/Jupyter before running Cytocipher, and you must confirm the matrix is log-cpm normalised and carries tentative Leiden cluster labels, preferably at high resolution.

## When NOT to use

- Input is raw counts not yet log-cpm normalised, or lacks any tentative cluster labels — Cytocipher expects log-cpm data with an existing over-clustered Leiden column
- The clustering is already at low resolution with few clusters and little over-clustering — the README advises high resolution so that there is a lot of over-clustering for merging
- The data are not single-cell RNA-seq AnnData/scanpy-compatible objects (Cytocipher is scverse-compatible and expects AnnData)

## Inputs

- AnnData object (data) processed similarly to the scanpy standard workflow
- Log-cpm normalised expression matrix
- Tentative high-resolution Leiden cluster label column (e.g. 'leiden') in AnnData.obs

## Outputs

- Validated AnnData ready for cc.tl.get_markers / cc.tl.code_enrich / cc.tl.merge_clusters
- Cell-by-cluster enrichment scores added to the AnnData by Cytocipher

## How to apply

Load the AnnData object in a Jupyter notebook (import cytocipher as cc after !pip install cytocipher) and check it has been processed similarly to the scanpy standard workflow to produce log-cpm normalised counts. Ensure a tentative clustering column exists (e.g. 'leiden' from Leiden clustering); the Cytocipher README notes a high Leiden resolution is better so there is a lot of over-clustering, since Cytocipher merges non-significantly different clusters. Pass this column key as the second argument to cc.tl.get_markers, cc.tl.code_enrich, and cc.tl.merge_clusters (e.g. cc.tl.get_markers(data, 'leiden')). In the E18.5 hypothalamus benchmark, the Yaghmaeian Salmani et al. 2022 AnnData was already log-cpm normalised with tentative high-resolution Leiden labels before marker identification and code enrichment scoring.

## Related tools

- **Cytocipher** (consumes the log-cpm normalised AnnData with 'leiden' labels for marker identification, code scoring, and cluster merging) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (reference standard workflow used to preprocess the AnnData into log-cpm normalised form with Leiden cluster labels)
- **AnnData** (container format of the input object passed as 'data' to Cytocipher functions)
- **Jupyter** (notebook environment where Cytocipher functions are invoked and documentation inspected (e.g. ?cc.tl.get_markers))

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden')
```

## Evaluation signals

- AnnData matrix values are log-cpm normalised (processed 'similarly to the scanpy standard workflow') before calling any cc.tl functions
- data.obs contains the tentative clustering column passed as the label key (e.g. 'leiden'), and it is over-clustered/high-resolution rather than pre-merged
- cc.tl.get_markers and cc.tl.code_enrich run without errors on (data, 'leiden'), confirming the expected AnnData schema
- cc.pl.sig_cluster_diagnostics returns violin plots with printed p-values for chosen cluster pairs (e.g. plot_pair=('3','9')), showing scores were computed per cell
- Cluster merging (cc.tl.merge_clusters) produces a 'leiden_merged' key whose enrichment heatmap (cc.pl.enrich_heatmap(data, 'leiden_merged')) can be plotted

## Limitations

- Low-resolution Leiden labels give little over-clustering, weakening Cytocipher's cluster merging — the README advises 'It's better if the Leiden resolution is high, so that there is alot of over-clustering'
- Cytocipher merges non-significantly different clusters, so the input labels are expected to be tentative rather than final
- No changelog found for the repository, so version-specific behaviour of preprocessing expectations may be undocumented (manuscript software version deposited at Zenodo doi 10.5281/zenodo.8089546)

## Evidence

- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering).: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html) to produce log-cpm normalised data"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [methods] cc.tl.get_markers(data, 'leiden'): "cc.tl.get_markers(data, 'leiden')"
- [methods] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high: "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high"
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
