---
name: orthologous-group-lca-gene-age-interpretation
description: Use when a target species' genes need evolutionary age classes derived
  from precomputed or newly computed orthologous groups (OrthoFinder, eggNOG v6, PLAZA
  v5), e.g. to build an orthomap or count species gene-age coverage.
when_to_use_negative:
- The orthologous groups must first be computed from protein/CDS FASTA — this skill
  consumes existing OrthoFinder/eggNOG/PLAZA orthogroup results, not raw sequence
  files.
- The goal is orthology inference itself (finding orthogroups/orthologs) rather than
  interpreting existing orthogroups as gene ages — use OrthoFinder directly instead.
- Taxonomic lineages are needed for species other than the query target and without
  orthogroup context — use NCBI taxonomy queries (ncbitax/qlin) alone.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0085
tools:
- name: oggmap
  role: Python package that extracts and assigns the lowest common ancestor (LCA)
    from orthologous groups as gene age class for a target species; sub-commands of2orthomap,
    plaza2orthomap, orthomcl2orthomap extract the orthomap, qlin/ncbitax resolve lineages
    from NCBI taxonomy
  repo: https://github.com/kullrich/oggmap
- name: OrthoFinder
  role: Source of orthologous groups from which oggmap extracts the orthomap via of2orthomap
  repo: https://github.com/davidemms/OrthoFinder
- name: eggNOG
  role: Precomputed homologous groups (eggnog_6.0) from which orthomaps can be extracted
- name: PLAZA
  role: Precomputed gene family data (plaza2orthomap) used as an orthogroup source
    for plant species (dicots and monocots)
- name: Scanpy
  role: Downstream toolkit for calculating gene age-weighted expression data from
    scRNA sequencing objects using the extracted orthomap
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:39f7a6145e894d291fdb4c9bf5d599687de06aeda6f8d89a8277e5a6f7ea075e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - PLAZA
  - Python
  - Scanpy
  - eggNOG
  - oggmap
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# orthologous-group-lca-gene-age-interpretation

## Summary

Interprets orthologous groups as gene ages by assigning the lowest common ancestor (LCA) of each orthogroup as a gene age class for a target species, using oggmap to extract an orthomap from OrthoFinder, eggNOG, or PLAZA orthology results. The resulting gene ages per orthogroup enable downstream analyses such as gene age-weighted expression (TEI) on scRNA data.

## When to use

Use when a target species' genes need evolutionary age classes derived from precomputed or newly computed orthologous groups (OrthoFinder, eggNOG v6, PLAZA v5), e.g. to build an orthomap or count species gene-age coverage.

## When NOT to use

- The orthologous groups must first be computed from protein/CDS FASTA — this skill consumes existing OrthoFinder/eggNOG/PLAZA orthogroup results, not raw sequence files.
- The goal is orthology inference itself (finding orthogroups/orthologs) rather than interpreting existing orthogroups as gene ages — use OrthoFinder directly instead.
- Taxonomic lineages are needed for species other than the query target and without orthogroup context — use NCBI taxonomy queries (ncbitax/qlin) alone.

## Inputs

- OrthoFinder results (orthogroups) for a query species, e.g. from ensembl release-113 longest-isoform CDS with diamond_ultra_sens
- eggNOG 6.0 homologous groups (http://eggnog6.embl.de/)
- PLAZA gene family data (dicots and monocots, PLAZA v5)
- Local NCBI taxonomy database (taxadb.sqlite, ~150MB, updated via oggmap ncbitax)
- GTF files for gene/transcript ID matching (optional, for scRNA linkage)

## Outputs

- Orthomap: gene ages per orthogroup (LCA-derived gene age class assignments for the target species)
- Query species taxonomic lineage / topology from NCBI taxonomy (via qlin)
- Gene age class assignments matched to gene and transcript IDs, usable in Scanpy scRNA objects
- Per-source species coverage counts, e.g. 317 species (Ensembl release-110), 1322 species (eggNOG v6), 98 dicots + 52 monocots (PLAZA v5)

## How to apply

First obtain orthologous groups for the query species — either run OrthoFinder on translated longest-isoform CDS (e.g. ensembl release-113 FASTA, diamond_ultra_sens) or load precomputed homologous groups from eggNOG 6.0 or PLAZA gene family data. Use the matching oggmap sub-command to extract the orthomap: of2orthomap for OrthoFinder output, plaza2orthomap for PLAZA gene family data, or orthomcl2orthomap for orthomcl output. For each orthologous group, oggmap assigns the lowest common ancestor (LCA) from the NCBI taxonomy database (local taxadb.sqlite, updated via ncbitax) as the gene age class for the target species. Confirm the query species lineage first with qlin against the NCBI taxonomy. The resulting orthomap (gene ages per orthogroup) can then be matched with gene/transcript IDs from GTF files (gtf2t2g) and linked to scRNA objects to compute gene age-weighted expression and the transcriptome evolutionary index (TEI).

## Related tools

- **oggmap** (Python package that extracts and assigns the lowest common ancestor (LCA) from orthologous groups as gene age class for a target species; sub-commands of2orthomap, plaza2orthomap, orthomcl2orthomap extract the orthomap, qlin/ncbitax resolve lineages from NCBI taxonomy) — https://github.com/kullrich/oggmap
- **OrthoFinder** (Source of orthologous groups from which oggmap extracts the orthomap via of2orthomap) — https://github.com/davidemms/OrthoFinder
- **eggNOG** (Precomputed homologous groups (eggnog_6.0) from which orthomaps can be extracted)
- **PLAZA** (Precomputed gene family data (plaza2orthomap) used as an orthogroup source for plant species (dicots and monocots))
- **Scanpy** (Downstream toolkit for calculating gene age-weighted expression data from scRNA sequencing objects using the extracted orthomap)

## Examples

```
oggmap ncbitax -u -outdir taxadb -type taxa -dbname taxadb.sqlite; oggmap qlin -q "Danio rerio" -dbname taxadb.sqlite; oggmap of2orthomap -q "Danio rerio" -dbname taxadb.sqlite
```

## Evaluation signals

- Per-source species coverage counts match reported figures: Ensembl release-110: 317 species; eggNOG v6: 1322 species; PLAZA v5: 98 dicots and 52 monocots.
- Summed per-source eukaryotic species coverage exceeds 1000 species, consistent with the abstract claim that oggmap provides extracted gene ages for more than thousand eukaryotic species.
- Each distinct species enumerated possesses at least one LCA-derived gene age class assignment in its source orthomap file.
- The query species lineage returned by qlin (e.g. Danio rerio, taxID 7955) is consistent with the NCBI taxonomy database stored in taxadb.sqlite.

## Limitations

- oggmap v0.0.2 was developed using Python 3.10; Python 2.7x and Python <=3.10 (older) are not supported.
- OrthoFinder results used for orthomap extraction are version- and release-dependent (e.g. ensembl release-105 vs release-113 CDS examples; precomputed coverage figures cite Ensembl release-110, eggNOG v6, PLAZA v5).
- The local NCBI taxonomy database must be downloaded/updated (~150MB) before LCA lineage assignment; staleness of taxadb.sqlite affects LCA calls.
- Article discussion/supplementary sections provide no dataset accessions or data availability statements beyond the cited repositories and Zenodo archive.

## Evidence

- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "`oggmap` is a python package that can: - extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [abstract] oggmap provides extracted gene ages for more than thousand eukaryotic species: "oggmap provides extracted gene ages for more than thousand eukaryotic species"
- [readme] The following code extracts the orthomap for Danio rerio based on pre-calculated OrthoFinder results and ensembl release-113: "The following code extracts the `orthomap` for `Danio rerio` based on pre-calculated 
OrthoFinder results and ensembl release-113"
- [methods] plaza2orthomap extract orthomap from PLAZA gene family data for query species: "plaza2orthomap      extract orthomap from PLAZA gene family data for query species"
- [methods] show species lineage information from the NCBI taxonomy database.: "show species lineage information from the `NCBI taxonomy <https://www.ncbi.nlm.nih.gov/taxonomy>`_ database."
