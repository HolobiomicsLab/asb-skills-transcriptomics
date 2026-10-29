---
name: per-cell-tei-computation
description: Use when an AnnData scRNA-seq object and a gene-age orthomap (e.g. from
  OrthoFinder/oggmap) are available and you need gene age-weighted expression summaries
  such as per-cell or per-stage TEI to test developmental hourglass patterns.
when_to_use_negative:
- The input is already a gene age-weighted expression matrix or a computed TEI vector
  — re-running get_tei would double-weight the data
- No orthomap (gene ages per orthogroup) exists for the query species and generating
  one (OrthoFinder/oggmap extraction) has not been done — compute the orthomap first
- The goal is orthogroup/gene-age extraction itself (cds2aa, of2orthomap, gtf2t2g)
  rather than linking ages to expression data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: oggmap
  role: Provides orthomap2tei helper functions (get_tei) to match gene age classes
    to AnnData var_names and compute per-cell transcriptome evolutionary index; gtf2t2g
    builds the transcript-to-gene table
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Python toolkit for handling the scRNA AnnData object in which TEI is calculated
    and stored, and for downstream visualization
- name: Python
  role: Runtime environment for the oggmap/Scanpy workflow (oggmap v0.0.2 developed
    with Python 3.10)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:9bd5b61cf9ad27600fe104c90f446792240573b3babb840afdef1a3e73966ed3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# per-cell-tei-computation

## Summary

Compute a per-cell transcriptome evolutionary index (TEI) by weighting gene expression in an scRNA-seq object (AnnData/Scanpy) with gene age classes from an oggmap orthomap, then aggregate stage/cell-type-level TEI profiles to detect evolutionary hourglass patterns.

## When to use

Use when an AnnData scRNA-seq object and a gene-age orthomap (e.g. from OrthoFinder/oggmap) are available and you need gene age-weighted expression summaries such as per-cell or per-stage TEI to test developmental hourglass patterns.

## When NOT to use

- The input is already a gene age-weighted expression matrix or a computed TEI vector — re-running get_tei would double-weight the data
- No orthomap (gene ages per orthogroup) exists for the query species and generating one (OrthoFinder/oggmap extraction) has not been done — compute the orthomap first
- The goal is orthogroup/gene-age extraction itself (cds2aa, of2orthomap, gtf2t2g) rather than linking ages to expression data

## Inputs

- GEO-deposited scRNA-seq AnnData object (~70,000 zebrafish cells) with raw or normalized counts
- oggmap orthomap gene age table (gene ages per orthogroup, e.g. from task_001 OrthoFinder output)
- Species GTF annotation (Danio rerio GRCz11) for building the transcript-to-gene table via gtf2t2g

## Outputs

- AnnData object (e.g. zebrafish_tei.h5ad) with per-cell TEI stored in obs
- Stage-level summary table of mean TEI per developmental stage
- Stage-wise TEI plot (e.g. matching Fig. 1d hourglass profile)

## How to apply

Build a transcript-to-gene table from the species GTF with oggmap gtf2t2g (e.g. Danio rerio GRCz11) and use it to map transcript-level IDs in the AnnData var_names to gene-level IDs. Match the orthomap gene age classes to the AnnData var_names with the oggmap.orthomap2tei helper, excluding unannotated genes from the weighted calculation. Compute the per-cell TEI vector with orthomap2tei get_tei and store it in the AnnData obs, saving the annotated object (e.g. zebrafish_tei.h5ad). Then aggregate mean TEI per developmental stage or cell type and plot the stage-wise profile (as in the zebrafish ~70,000-cell analysis, where the minimum TEI occurs at the 12 hpf stage, reproducing the hourglass). Rationale: TEI weights expression by evolutionary age of each gene's orthogroup, so low TEI indicates expression dominated by evolutionarily young genes, the signature of the hourglass model at mid-development.

## Related tools

- **oggmap** (Provides orthomap2tei helper functions (get_tei) to match gene age classes to AnnData var_names and compute per-cell transcriptome evolutionary index; gtf2t2g builds the transcript-to-gene table) — https://github.com/kullrich/oggmap
- **Scanpy** (Python toolkit for handling the scRNA AnnData object in which TEI is calculated and stored, and for downstream visualization)
- **Python** (Runtime environment for the oggmap/Scanpy workflow (oggmap v0.0.2 developed with Python 3.10))

## Examples

```
from oggmap import orthomap2tei; adata.obs['tei'] = orthomap2tei.get_tei(adata=adata, gene_id_to_age=orthomap)
```

## Evaluation signals

- Stage-level summary table of mean TEI exists and the developmental stage with the minimum TEI equals the 12-hpf stage (zebrafish hourglass validation)
- Per-cell TEI vector is present in AnnData obs and object saves successfully as zebrafish_tei.h5ad
- Gene IDs in the AnnData var_names are gene-level (post gtf2t2g mapping), and orthomap age classes match those var_names with unannotated genes excluded from the weighted calculation
- Stage-wise TEI plot reproduces the hourglass profile matching Fig. 1d

## Limitations

- Unannotated genes (without orthomap age classes) are kept out of the weighted TEI calculation, so TEI covers only age-annotated genes
- oggmap v0.0.2 was developed with Python 3.10; Python 2.7x and versions <=3.10 are not supported, which constrains the environment
- No versioned dependencies stated for the 0.0.1 release in the discussion/changelog, so reproducibility of exact package versions must be pinned manually

## Evidence

- [abstract] calculate gene age-weighted expression data from scRNA sequencing objects: "which can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] match IDs and compute TEI: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets. calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [methods] gtf2t2g subcommand: "gtf2t2g             extract transcript to gene table from GTF <gtf2t2g -h>"
- [readme] zebrafish example figure: [evidence span withheld — anchor status: external_doc]
- [abstract] gene ages for eukaryotic species: "oggmap provides extracted gene ages for more than thousand eukaryotic species"
