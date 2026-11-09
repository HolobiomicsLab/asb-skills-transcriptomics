---
name: scrna-anndata-processing
description: Use when a pre-existing orthomap (gene age class per gene) must be joined
  with single-cell expression data in a Scanpy AnnData object to compute per-cell
  TEI or partial TEI profiles across stages or cell types.
when_to_use_negative:
- No orthomap exists yet for the query species — first extract gene ages with oggmap
  subcommands (qlin, ncbitax, of2orthomap, eggnog/plaza2orthomap) before linking to
  scRNA data
- The expression dataset is bulk RNA-seq or non-single-cell; TEI per cell and partial
  TEI strata assume Scanpy scRNA sequencing objects
- The analysis is already in an R/Seurat workflow and cannot be represented as a Scanpy
  AnnData object; oggmap's gene age-weighted calculation is built on the Python Scanpy
  toolkit
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: oggmap
  role: Provides the orthomap2tei module (get_TEI, get_pstrata) to match gene age
    classes with scRNA data and compute gene age-weighted expression indices
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Holds the single-cell data as an AnnData object into which gene ages and TEI
    values are injected
- name: OrthoFinder
  role: Source of orthogroup results from which the orthomap (gene ages per orthogroup)
    was extracted prior to scRNA processing
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:ed185772872a71649e5d9a6ddaa17bcbd6e7b6feb27fc90e8b097982fef74b7b
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scrna-anndata-processing

## Summary

Loads a single-cell RNA-seq dataset (e.g. GEO-deposited Packer et al. 2019 C. elegans embryo data, ~90,000 cells) into a Scanpy AnnData object and matches gene/transcript IDs with their gene age class from a pre-computed orthomap, so that gene age-weighted expression indices (TEI, partial TEI) can be computed per cell with oggmap.

## When to use

Use when a pre-existing orthomap (gene age class per gene) must be joined with single-cell expression data in a Scanpy AnnData object to compute per-cell TEI or partial TEI profiles across stages or cell types.

## When NOT to use

- No orthomap exists yet for the query species — first extract gene ages with oggmap subcommands (qlin, ncbitax, of2orthomap, eggnog/plaza2orthomap) before linking to scRNA data
- The expression dataset is bulk RNA-seq or non-single-cell; TEI per cell and partial TEI strata assume Scanpy scRNA sequencing objects
- The analysis is already in an R/Seurat workflow and cannot be represented as a Scanpy AnnData object; oggmap's gene age-weighted calculation is built on the Python Scanpy toolkit

## Inputs

- Single-cell RNA-seq count matrix (GEO-deposited, e.g. Packer et al. 2019 C. elegans embryos, ~90,000 cells)
- Pre-computed orthomap mapping gene/transcript IDs to gene age classes (e.g. Sun et al. 2021 C. elegans gene ages)
- Transcript-to-gene ID table (e.g. produced by oggmap gtf2t2g from a GTF file)

## Outputs

- Scanpy AnnData object with gene age class annotations and per-cell TEI in obs
- Partial TEI strata table per gene age class per cell/stage (CSV)
- Stacked partial TEI profile figure (e.g. nematode_partial_tei.png)

## How to apply

Build a Scanpy AnnData object from the raw scRNA count matrix (e.g. the ~90,000-cell C. elegans embryonic scRNA dataset from GEO) and load the pre-existing orthomap from Sun et al. 2021 that assigns each gene a lowest common ancestor (LCA)-derived gene age class (classes 11–13 are the youngest, nematode/sister-species-specific bins). Use oggmap's orthomap2tei module to match gene and transcript IDs and their corresponding gene age class against the AnnData var names; genes whose IDs cannot be matched are dropped, so verify the match rate before proceeding. Compute the gene age-weighted transcriptome evolutionary index (TEI) per cell/stage with oggmap's get_TEI, then compute stacked partial TEI strata with get_pstrata so each age class's contribution sums to the global TEI per stage. Finally calculate mean/relative expression per gene age class, export the partial TEI table to CSV, and render the stacked profile figure (e.g. nematode_partial_tei.png).

## Related tools

- **oggmap** (Provides the orthomap2tei module (get_TEI, get_pstrata) to match gene age classes with scRNA data and compute gene age-weighted expression indices) — https://github.com/kullrich/oggmap
- **Scanpy** (Holds the single-cell data as an AnnData object into which gene ages and TEI values are injected)
- **OrthoFinder** (Source of orthogroup results from which the orthomap (gene ages per orthogroup) was extracted prior to scRNA processing) — https://github.com/davidemms/OrthoFinder

## Examples

```
from oggmap import orthomap2tei as o2t; o2t.get_TEI(adata=adata, gene_id='gene_id', gene_name='ct', gene_age='ct', add_obs=True)
```

## Evaluation signals

- The partial TEI CSV exists and contains the expected gene age classes (e.g. 11–13 for the nematode profile)
- Per stage, the stacked partial TEI contributions sum to the global TEI value for that stage
- Gene/transcript IDs from the orthomap successfully match a non-empty subset of AnnData var names; check match rate before computing TEI
- Every cell/stage has a finite TEI value in obs after get_TEI
- The rendered stacked partial TEI figure reproduces the expected pattern — youngest age classes 11–13 peaking in early embryos (Fig. 1e; validated via llm_judge on the PNG)

## Limitations

- oggmap does not support Python 2.7x or Python <=3.10; it was developed with Python 3.10, so environment setup matters (conda environment.yml or docker container recommended)
- Gene age assignment depends on ID matching — unmatched transcript IDs silently reduce coverage of the TEI index
- The release notes/changelog carry no dataset accessions or versioned dependency statements, so reproducibility requires pinning the orthomap source and scRNA accession manually

## Evidence

- [methods] match gene and transcript IDs and their corresponding gene age class with scRNA data sets.: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [abstract] which can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit: "which can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] calculate and visualize partial transcriptome evolutionary index (TEI) profiles on scRNA data sets.: "calculate and visualize partial transcriptome evolutionary index (TEI) profiles on scRNA data sets."
- [readme] We recommend installing `oggmap` in an independent conda environment to avoid dependent software conflicts.: "We recommend installing `oggmap` in an independent conda environment to avoid dependent software conflicts."
- [methods] `oggmap` is a python package that can: - extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "`oggmap` is a python package that can: - extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
