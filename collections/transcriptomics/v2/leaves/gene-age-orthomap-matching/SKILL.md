---
name: gene-age-orthomap-matching
description: Use when an AnnData/scRNA object must be annotated with per-gene evolutionary
  ages (an orthomap) before computing a transcriptome evolutionary index (TEI), e.g.
  reproducing hourglass or young-gene-peak patterns across embryo stages.
when_to_use_negative:
- The scRNA object and gene age table are already merged (age classes already present
  in AnnData var/obs) — re-matching would duplicate annotation effort
- No orthogroup or gene family resource (OrthoFinder, eggNOG, PLAZA) exists for the
  query species, so no orthomap can be extracted or sourced
- The analysis does not involve gene age-weighted or evolutionary indices (e.g. plain
  differential expression or cell-type annotation), where orthomap matching adds no
  value
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: oggmap
  role: Provides the orthomap2tei module that matches gene/transcript IDs and their
    gene age class with scRNA datasets, and gtf2t2g/of2orthomap to build the age table
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: AnnData container holding the scRNA data whose var_names the orthomap age
    classes are matched against
- name: OrthoFinder
  role: Source of orthogroups from which oggmap extracts the orthomap (gene ages per
    orthogroup) for the query species
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:95cd0219a09934ee9c4184ea0f2531054ba2adb09f15c4b9ba35fcea02e93fa2
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-age-orthomap-matching

## Summary

Match gene and transcript identifiers with their orthogroup-derived evolutionary age class (an orthomap from oggmap/OrthoFinder or eggNOG) against the var_names of a Scanpy AnnData single-cell object, so that gene age-weighted indices such as TEI or partial TEI can be computed per cell or per developmental stage.

## When to use

Use when an AnnData/scRNA object must be annotated with per-gene evolutionary ages (an orthomap) before computing a transcriptome evolutionary index (TEI), e.g. reproducing hourglass or young-gene-peak patterns across embryo stages.

## When NOT to use

- The scRNA object and gene age table are already merged (age classes already present in AnnData var/obs) — re-matching would duplicate annotation effort
- No orthogroup or gene family resource (OrthoFinder, eggNOG, PLAZA) exists for the query species, so no orthomap can be extracted or sourced
- The analysis does not involve gene age-weighted or evolutionary indices (e.g. plain differential expression or cell-type annotation), where orthomap matching adds no value

## Inputs

- Scanpy AnnData object (h5ad) with scRNA counts, e.g. ~70,000-cell zebrafish (Packer-style GEO-deposited) or ~90,000-cell C. elegans embryonic dataset
- Orthomap gene age table mapping gene identifiers to phylostratum/age class (oggmap of2orthomap / eggNOG 6.0 / PLAZA / published orthomap)
- Genome GTF annotation (e.g. Danio rerio GRCz11 GTF) for building a transcript-to-gene table via gtf2t2g when IDs are transcript-level

## Outputs

- AnnData object with orthomap age classes matched to var_names and per-cell TEI stored in obs (saved as e.g. zebrafish_tei.h5ad)
- Aligned gene-to-age-class lookup table usable by orthomap2tei get_tei and get_pstrata
- Partial TEI table per gene age class per stage (CSV) with age classes 11–13 represented for nematode data

## How to apply

First obtain the orthomap — a table of gene age classes per gene for the query species — either via oggmap's of2orthomap on OrthoFinder results, eggNOG 6.0 or PLAZA gene family data, or a pre-existing published orthomap (e.g. the Sun et al. 2021 C. elegans gene age table). If the scRNA object stores transcript-level IDs, build a transcript-to-gene table with `oggmap gtf2t2g` on the matching genome annotation (e.g. Danio rerio GRCz11 GTF) and collapse/map the AnnData var_names to gene-level IDs. Then use the oggmap.orthomap2tei module to match gene and transcript IDs and their corresponding gene age class with the scRNA data, keeping genes without an assigned age class out of the weighted calculation rather than imputing ages. The aligned orthomap feeds downstream `get_tei` (per-cell TEI stored in AnnData obs) and `get_pstrata` (partial TEI per age class, summing to the global TEI) computations.

## Related tools

- **oggmap** (Provides the orthomap2tei module that matches gene/transcript IDs and their gene age class with scRNA datasets, and gtf2t2g/of2orthomap to build the age table) — https://github.com/kullrich/oggmap
- **Scanpy** (AnnData container holding the scRNA data whose var_names the orthomap age classes are matched against)
- **OrthoFinder** (Source of orthogroups from which oggmap extracts the orthomap (gene ages per orthogroup) for the query species) — https://github.com/davidemms/OrthoFinder

## Examples

```
from oggmap import orthomap2tei; adata.obs['tei'] = orthomap2tei.get_tei(adata=adata, gene_age=orthomap)
```

## Evaluation signals

- Every AnnData gene with a non-null age class in the orthomap is matched; unannotated genes are verifiably excluded from the weighted TEI calculation rather than silently included
- After gtf2t2g mapping, transcript-level var_names resolve to gene-level IDs that overlap the orthomap's gene identifiers (count matched genes and report coverage)
- Downstream sanity check: per-cell TEI stored in AnnData obs is finite and within a plausible age-class range; partial TEI strata per age class sum to the global TEI per cell/stage
- Task-level check: in the C. elegans re-analysis the partial TEI table contains age classes 11–13 and those young classes peak in early-embryo stages; in the zebrafish re-analysis the minimum stage-wise TEI equals the 12-hpf stage

## Limitations

- oggmap v0.0.2 requires Python >= 3.10 — Python 2.7x and Python <= 3.10 are not supported
- Genes without an assigned orthogroup/age class are kept out of the weighted calculation, so the TEI covers only the annotated fraction of the transcriptome
- Transcript-level gene IDs in the scRNA object must be collapsed to gene level (e.g. via gtf2t2g on the matching GTF such as GRCz11) or the orthomap match will fail

## Evidence

- [methods] match gene and transcript IDs and their corresponding gene age class with scRNA data sets.: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [methods] extract and assign LCA gene age class: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [readme] orthomap definition from README: [evidence span withheld — anchor status: external_doc]
- [methods] transcript-to-gene extraction from GTF: "gtf2t2g             extract transcript to gene table from GTF"
- [readme] oggmap connects orthomap results to weighted scRNA expression: [evidence span withheld — anchor status: external_doc]
