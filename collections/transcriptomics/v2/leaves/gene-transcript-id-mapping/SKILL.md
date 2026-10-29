---
name: gene-transcript-id-mapping
description: Use when scRNA AnnData var_names are transcript-level or otherwise inconsistent
  with the gene IDs in an orthomap/OrthoFinder-derived gene age table, and IDs must
  be collapsed or matched before computing gene age-weighted TEI.
when_to_use_negative:
- The scRNA object already has gene-level IDs that directly overlap the orthomap gene
  identifiers — go straight to the orthomap2tei matching step
- No reference GTF or gene-to-transcript annotation exists for the species, so gtf2t2g
  cannot be run
- The analysis does not link an orthomap to scRNA data (e.g. plain orthogroup extraction
  with no expression matrix), so ID matching to an AnnData object is unnecessary
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: oggmap
  role: Provides gtf2t2g to extract the transcript-to-gene table from a GTF, and the
    orthomap2tei helper to match gene/transcript IDs and gene age classes with scRNA
    datasets
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Handles the AnnData scRNA object whose var_names are remapped from transcript-level
    to gene-level IDs prior to TEI calculation
- name: OrthoFinder
  role: Upstream producer of the orthogroup results whose gene identifiers define
    the target ID space the scRNA matrix must be mapped to
  repo: https://github.com/kullrich/oggmap
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:4f75d929e454716e8783b57f45d6afe04115212b92e3d5cf33815b1435082e5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - Scanpy
  - oggmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-transcript-id-mapping

## Summary

Reconciles transcript-level and gene-level identifiers between an orthomap (gene ages per orthogroup) and single-cell RNA data, using oggmap's gtf2t2g to extract a transcript-to-gene table from a GTF (e.g. Danio rerio GRCz11) so gene age classes can be matched to AnnData var_names before TEI calculation.

## When to use

Use when scRNA AnnData var_names are transcript-level or otherwise inconsistent with the gene IDs in an orthomap/OrthoFinder-derived gene age table, and IDs must be collapsed or matched before computing gene age-weighted TEI.

## When NOT to use

- The scRNA object already has gene-level IDs that directly overlap the orthomap gene identifiers — go straight to the orthomap2tei matching step
- No reference GTF or gene-to-transcript annotation exists for the species, so gtf2t2g cannot be run
- The analysis does not link an orthomap to scRNA data (e.g. plain orthogroup extraction with no expression matrix), so ID matching to an AnnData object is unnecessary

## Inputs

- Species GTF annotation file (e.g. Danio rerio GRCz11 GTF from Ensembl release-105/113)
- scRNA count object (AnnData) with transcript-level or otherwise mismatched var_names
- Orthomap gene age table (gene IDs per orthogroup with age classes, from of2orthomap/eggnog2orthomap or OrthoFinder results)

## Outputs

- Transcript-to-gene mapping table produced by gtf2t2g
- AnnData with gene-level var_names (transcripts collapsed to genes)
- Gene age class annotations matched to var_names via oggmap.orthomap2tei, ready for get_tei

## How to apply

Run oggmap's gtf2t2g sub-command on the species GTF annotation (e.g. Danio rerio GRCz11 from Ensembl) to build a transcript-to-gene table, then use it to collapse or remap transcript-level IDs in the AnnData object to gene-level IDs. Next, match the orthomap gene IDs and their corresponding gene age classes to the AnnData var_names with the oggmap.orthomap2tei helper, excluding unannotated genes from any downstream weighted (TEI) calculation. Because scRNA count matrices and OrthoFinder/eggNOG orthogroup tables come from different ID universes (Ensembl transcript IDs vs. orthogroup gene lists), the mapping step must be done before aggregation; duplicate transcript entries collapsed onto one gene should have their counts combined so the matrix dimensions stay consistent with the matched age-class vector. Verify the match by checking that the majority of var_names carry an age class and that no unmapped identifiers propagate into the TEI vector stored in obs.

## Related tools

- **oggmap** (Provides gtf2t2g to extract the transcript-to-gene table from a GTF, and the orthomap2tei helper to match gene/transcript IDs and gene age classes with scRNA datasets) — https://github.com/kullrich/oggmap
- **Scanpy** (Handles the AnnData scRNA object whose var_names are remapped from transcript-level to gene-level IDs prior to TEI calculation)
- **OrthoFinder** (Upstream producer of the orthogroup results whose gene identifiers define the target ID space the scRNA matrix must be mapped to) — https://github.com/kullrich/oggmap

## Examples

```
oggmap gtf2t2g -h
```

## Evaluation signals

- The gtf2t2g output table contains one row per transcript with a valid gene-level assignment from the GRCz11 GTF
- All AnnData var_names are gene-level after remapping, with no residual transcript IDs left before get_tei
- A large fraction of var_names receive a gene age class from the orthomap; unannotated genes are dropped from the TEI-weighted calculation rather than propagated as errors
- Per-cell TEI values stored in AnnData obs are finite (no NaN from unmatched IDs) and aggregate to a stage-wise summary consistent with published patterns
- Collapsed count matrices retain the same number of cells, with only the var dimension reduced to gene-level

## Limitations

- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10, constraining the environment for the mapping step
- Only orthogroup sources with dedicated parsers (OrthoFinder, eggNOG 6.0, PLAZA, OrthoMCL, Broccoli) can be matched; gene age tables from other resources need custom ID conversion
- Genes present in the scRNA matrix but absent from the orthomap remain unannotated and are excluded from weighted TEI calculations, which can bias the index if coverage is low

## Evidence

- [methods] gtf2t2g extracts transcript-to-gene table from GTF: "gtf2t2g             extract transcript to gene table from GTF <gtf2t2g -h>"
- [methods] IDs matched to scRNA datasets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [methods] gene/transcript information from GTF: "extract gene and transcript information from GTF files"
- [methods] longest isoform handling via cds2aa: "cds2aa              translate CDS to AA and optional retain longest isoform <cds2aa -h>"
- [methods] zebrafish GRCz11 reference sequences: "wget https://ftp.ensembl.org/pub/release-105/fasta/danio_rerio/cds/Danio_rerio.GRCz11.cds.all.fa.gz"
