---
name: annadata-object-manipulation
description: Use when a scRNA-seq analysis pipeline requires an AnnData object preprocessed
  via the scanpy standard workflow with log-cpm normalisation and high-resolution
  Leiden cluster labels as input to downstream tools.
when_to_use_negative:
- The data is already a feature table, marker matrix, or aggregated summary rather
  than a cell-level count matrix that can be normalised to log-cpm
- The analysis goal is de novo cell-type clustering rather than statistical testing/merging
  of pre-computed over-clustered Leiden labels
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: AnnData
  role: Container object holding the log-cpm normalised counts and cluster labels
    that all Cytocipher functions read from and write to
- name: scanpy
  role: Used for the standard preprocessing workflow (log-cpm normalisation, Leiden
    clustering) that produces the expected AnnData input
- name: Cytocipher
  role: scverse-compatible package whose cc.tl/cc.pl functions consume and augment
    the AnnData object (get_markers, code_enrich, merge_clusters)
  repo: https://github.com/BradBalderson/Cytocipher
- name: Jupyter
  role: Notebook environment for inspecting AnnData objects and function documentation
    (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:79f8306a0862cdd4d3a176b3f2c194b380692ff10fb3e3168f793915b6b33bec
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# annadata-object-manipulation

## Summary

Prepare and manipulate an AnnData object carrying log-cpm normalised single-cell RNA-seq counts and tentative Leiden cluster labels so that Cytocipher's marker identification, code scoring, and cluster merging functions can operate on it.

## When to use

Use when a scRNA-seq analysis pipeline requires an AnnData object preprocessed via the scanpy standard workflow with log-cpm normalisation and high-resolution Leiden cluster labels as input to downstream tools.

## When NOT to use

- The data is already a feature table, marker matrix, or aggregated summary rather than a cell-level count matrix that can be normalised to log-cpm
- The analysis goal is de novo cell-type clustering rather than statistical testing/merging of pre-computed over-clustered Leiden labels

## Inputs

- Raw single-cell RNA-seq count matrix loadable into an AnnData object (.h5ad or equivalent)
- Leiden cluster labels stored in the AnnData obs slot (e.g. key 'leiden')

## Outputs

- AnnData object with log-cpm normalised expression and tentative high-resolution Leiden cluster labels
- AnnData augmented by Cytocipher with per-cell cluster enrichment scores (cell-by-cluster matrix) and merged cluster labels (e.g. 'leiden_merged')

## How to apply

Load the raw count matrix into an AnnData object (data) and process it similarly to the scanpy standard PBMC3k workflow to produce log-cpm normalised values. Perform Leiden clustering with a deliberately high resolution so the data is over-clustered, since Cytocipher is designed to merge the non-significantly different clusters downstream. Attach the tentative cluster labels as a categorical column (e.g. 'leiden') in the AnnData obs slot, because functions like cc.tl.get_markers(data, 'leiden') and cc.tl.code_enrich(data, 'leiden') reference this key by name. Verify the object with jupyter-based introspection (e.g. ?cc.tl.get_markers) before running enrichment, and extract results such as the cell-by-cluster enrichment score matrix back out of the AnnData structure after scoring.

## Related tools

- **AnnData** (Container object holding the log-cpm normalised counts and cluster labels that all Cytocipher functions read from and write to)
- **scanpy** (Used for the standard preprocessing workflow (log-cpm normalisation, Leiden clustering) that produces the expected AnnData input)
- **Cytocipher** (scverse-compatible package whose cc.tl/cc.pl functions consume and augment the AnnData object (get_markers, code_enrich, merge_clusters)) — https://github.com/BradBalderson/Cytocipher
- **Jupyter** (Notebook environment for inspecting AnnData objects and function documentation (e.g. ?cc.tl.get_markers))

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden')
```

## Evaluation signals

- The AnnData object contains log-cpm normalised values consistent with the scanpy standard PBMC3k workflow before any Cytocipher call
- The obs slot holds tentative Leiden labels under the key passed to Cytocipher functions (e.g. 'leiden'), with high resolution/over-clustering
- After cc.tl.code_enrich, a cell-by-cluster enrichment score matrix is retrievable from the AnnData object
- After cc.tl.merge_clusters, merged labels (e.g. 'leiden_merged') are present and cc.pl.enrich_heatmap(data, 'leiden_merged') renders without error
- Diagnostics on a non-significant pair (e.g. clusters '3' and '9') yield p-values near 1, as in the documented example p=0.913 and p=0.813

## Limitations

- Input must follow the scanpy standard workflow; non-normalised or differently scaled AnnData objects are not the expected input
- Leiden resolution should be high (over-clustered); if clusters are already coarse, Cytocipher's merging has little to test
- Small clusters give sparse score comparisons — the documented example pair ('3','9') compares only 3 scores per group, which may limit statistical power

## Evidence

- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html) to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering).: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering."
- [methods] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [abstract] we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
