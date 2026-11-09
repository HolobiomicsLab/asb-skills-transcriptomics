---
name: split-pool-barcoding-layout-interpretation
description: Use when one or more seqspec spec.yaml files are available and the goal
  is to interpret a library's ordered sequence elements or to compare library structures
  across assays — e.g. checking which split-pool barcode rounds are shared between
  two modalities.
when_to_use_negative:
- Input is already a processed cell-by-feature count matrix (mtx/h5ad) — use mx inspect/filter
  or cellatlas preprocessing instead, since library structure is no longer the object
  of analysis
- The goal is read preprocessing (alignment, barcode correction, UMI counting) rather
  than interpreting or comparing library structure — seqspec print describes the spec,
  it does not quantify reads
- No seqspec spec.yaml exists for the assay of interest; the Region Ordered Tree can
  only be rendered from a machine-readable specification
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: seqspec
  role: Renders each spec.yaml as a Region Ordered Tree (seqspec print) whose feature
    annotations carry nucleotide lengths, enabling extraction and side-by-side comparison
    of ordered library regions per modality
  repo: https://github.com/IGVF/seqspec
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:6349dda7f78e519839f520204ae55c9444dcd8d004f6c16c7b8b06c4fb4ebe25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - seqspec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# split-pool-barcoding-layout-interpretation

## Summary

Interprets and compares the library structures encoded in seqspec YAML specifications, particularly to reveal that assays from different protocols (e.g. SPLiT-seq RNA-only vs. SHARE-seq RNA+ATAC) can share near-identical split-pool barcoding layouts. seqspec renders each spec as a Region Ordered Tree whose annotated nucleotide lengths make the ordered sequence elements (barcodes, UMIs, linkers, biological features) directly comparable.

## When to use

Use when one or more seqspec spec.yaml files are available and the goal is to interpret a library's ordered sequence elements or to compare library structures across assays — e.g. checking which split-pool barcode rounds are shared between two modalities.

## When NOT to use

- Input is already a processed cell-by-feature count matrix (mtx/h5ad) — use mx inspect/filter or cellatlas preprocessing instead, since library structure is no longer the object of analysis
- The goal is read preprocessing (alignment, barcode correction, UMI counting) rather than interpreting or comparing library structure — seqspec print describes the spec, it does not quantify reads
- No seqspec spec.yaml exists for the assay of interest; the Region Ordered Tree can only be rendered from a machine-readable specification

## Inputs

- seqspec spec.yaml file for each assay (e.g. SPLiT-seq and SHARE-seq specs from the IGVF/seqspec specs directory)
- IGVF/seqspec repository (source of assay specification files)

## Outputs

- Region Ordered Tree visualization per assay (from seqspec print)
- Tab-separated comparison table: one column per assay, one row per library region/modality, with nucleotide lengths and modality annotations

## How to apply

Obtain the assay spec.yaml files from the IGVF/seqspec assay library (git clone https://github.com/IGVF/seqspec.git), then run `seqspec print spec.yaml` on each to render its Region Ordered Tree, where each feature annotation carries its nucleotide length. Parse each tree with seqspec to extract the ordered regions per modality — barcodes, UMIs, linkers/spacers, and biological features — together with their nucleotide lengths. Align the extracted region structures side-by-side, marking the shared split-pool barcoding rounds (e.g. cell barcodes fixed by ligation/polymerase) and the differing modality branches (RNA-only for SPLiT-seq vs. RNA and ATAC for SHARE-seq). Emit the comparison as a tab-separated table with one column per assay and one row per library region/modality, so structural similarity despite different protocols becomes visually explicit.

## Related tools

- **seqspec** (Renders each spec.yaml as a Region Ordered Tree (seqspec print) whose feature annotations carry nucleotide lengths, enabling extraction and side-by-side comparison of ordered library regions per modality) — https://github.com/IGVF/seqspec

## Examples

```
git clone https://github.com/IGVF/seqspec.git; seqspec print seqspec/specs/splitseq/spec.yaml
```

## Evaluation signals

- The output table contains entries for both assays with modality annotations consistent with the seqspec print output
- Shared split-pool barcode layout rows match between the two assay columns
- Each extracted region carries a nucleotide length matching the annotation in the seqspec print Region Ordered Tree
- The differing modality branches (e.g. RNA-only for SPLiT-seq vs. RNA and ATAC for SHARE-seq) are correctly separated from the shared barcode rounds

## Limitations

- Structural comparison depends on the spec.yaml files being present and up-to-date in the IGVF/seqspec assay library; assays without machine-readable specs cannot be compared
- Different modalities between assays mean only the split-pool barcoding portion of the layouts aligns — downstream biological-feature branches differ and must be annotated as assay-specific
- seqspec currently requires custom handling where common library structure is not leveraged; the paper notes assays 'require custom scripts and programs that do not leverage the common structure of sequence elements'

## Evidence

- [abstract] seqspec as machine-readable spec enabling assay comparison: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing and enables tracking and comparison of genomics assays"
- [readme] repo README pointing to seqspec specs: [evidence span withheld — anchor status: external_doc]
- [methods] cloning the IGVF/seqspec repo for spec files: "# download the repo
git clone https://github.com/IGVF/seqspec.git"
