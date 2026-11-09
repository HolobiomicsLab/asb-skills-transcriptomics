---
name: orthogroup-lca-annotation
description: Use when you have OrthoFinder (or eggNOG/PLAZA/orthomcl) orthogroup results
  for a query species (e.g. Danio rerio) and need per-orthogroup gene ages (LCA classes)
  or continuity scores of species representation.
when_to_use_negative:
- Your orthogroups come from raw OrthoFinder/eggNOG output without a defined query
  species — the LCA assignment is target-species specific and needs qlin/of2orthomap
  context
- You only need TEI or gene age-weighted expression on scRNA objects (use the downstream
  Scanpy-based TEI step, not this annotation skill)
- Input is already a computed orthomap (gene ages per orthogroup) — skip directly
  to the continuity/TEI analysis
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: oggmap
  role: Python package that extracts the orthomap and assigns the LCA of orthologous
    groups as gene age class for the query species; subcommands ncbitax, qlin, and
    of2orthomap drive the workflow
  repo: https://github.com/kullrich/oggmap
- name: OrthoFinder
  role: Source of orthogroups and gene trees whose LCA is converted into gene age
    classes
  repo: https://github.com/davidemms/OrthoFinder
- name: Python
  role: Runtime for oggmap (v0.0.2, Python 3.10) and for pandas-based loading of the
    orthomap and species-membership table to compute continuity scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:ae73f2055b3b6269d30615d55522d5dcb0d68ec6139d6527136405a11b7bbd69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - oggmap
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# orthogroup-lca-annotation

## Summary

Annotates each orthologous group with its lowest common ancestor (LCA) as the gene age class for a query species using oggmap's of2orthomap logic on OrthoFinder results, and optionally derives a per-orthogroup continuity score of species completeness along the LCA-to-focal-species path. This grounds evolutionary age assignment for downstream scRNA analyses such as the transcriptome evolutionary index (TEI).

## When to use

Use when you have OrthoFinder (or eggNOG/PLAZA/orthomcl) orthogroup results for a query species (e.g. Danio rerio) and need per-orthogroup gene ages (LCA classes) or continuity scores of species representation.

## When NOT to use

- Your orthogroups come from raw OrthoFinder/eggNOG output without a defined query species — the LCA assignment is target-species specific and needs qlin/of2orthomap context
- You only need TEI or gene age-weighted expression on scRNA objects (use the downstream Scanpy-based TEI step, not this annotation skill)
- Input is already a computed orthomap (gene ages per orthogroup) — skip directly to the continuity/TEI analysis

## Inputs

- OrthoFinder results directory (orthogroups and rooted gene trees, e.g. from diamond_ultra_sens runs on translated longest-isoform CDS from Ensembl release-105/113)
- Query species name or taxID (e.g. Danio rerio, taxID 7955)
- Local NCBI taxonomy database (taxadb.sqlite)
- Per-orthogroup species-membership table from the orthomap step (TSV)

## Outputs

- Orthomap: gene ages (LCA classes) per orthogroup for the query species
- Query species lineage / topology (newick) from qlin
- continuity_scores.tsv: per-orthogroup continuity scores in [0, 1] with intermediate nodes lacking detectable orthologs flagged
- Candidate horizontal gene transfer list (orthogroups with low continuity scores)

## How to apply

First build a local NCBI taxonomy database with oggmap ncbitax (-u -outdir taxadb -type taxa -dbname taxadb.sqlite) and retrieve the query species lineage with qlin (e.g. q='Danio rerio', taxID 7955). Then run of2orthomap on the OrthoFinder output to assign, for each orthogroup, the LCA of its member species as the gene age class for the target species. For the optional continuity analysis, load the orthomap and the per-orthogroup species-membership table with pandas, walk the orthogroup tree nodes from the LCA toward the focal species, and compute the fraction of expected descendant species with detectable orthologs at each intermediate node; aggregate these node completeness values into a single continuity score per orthogroup. A low continuity score highlights possible horizontal gene transfer, since intermediate nodes along the lineage lack detectable orthologs. Write per-orthogroup scores to continuity_scores.tsv and verify every score lies in [0, 1].

## Related tools

- **oggmap** (Python package that extracts the orthomap and assigns the LCA of orthologous groups as gene age class for the query species; subcommands ncbitax, qlin, and of2orthomap drive the workflow) — https://github.com/kullrich/oggmap
- **OrthoFinder** (Source of orthogroups and gene trees whose LCA is converted into gene age classes) — https://github.com/davidemms/OrthoFinder
- **Python** (Runtime for oggmap (v0.0.2, Python 3.10) and for pandas-based loading of the orthomap and species-membership table to compute continuity scores)

## Examples

```
oggmap ncbitax -u -outdir taxadb -type taxa -dbname taxadb.sqlite; oggmap qlin -q "Danio rerio" -dbname taxadb.sqlite
```

## Evaluation signals

- continuity_scores.tsv exists and every continuity score lies in the range [0, 1]
- Each orthogroup's assigned LCA class matches the NCBI-taxonomy-derived lineage (qlin output) for the query species
- Orthogroups with low continuity scores have at least one intermediate node along the LCA-to-focal-species path lacking detectable orthologs
- The orthomap covers the orthogroups present in the OrthoFinder results for the query species (Danio rerio taxID 7955 in the paper's example)
- taxadb.sqlite was built successfully by ncbitax before lineage queries were made

## Limitations

- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10
- The NCBI taxonomy database download is ~150MB and must be kept up to date via ncbitax
- A low continuity score can indicate missing data (incomplete orthogroup representation) as well as horizontal gene transfer, so it flags possible HGT rather than proving it
- No versioned dependencies or dataset accessions are documented for some releases, so reproducibility depends on archived OrthoFinder inputs (e.g. the Zenodo-archived Ensembl release-113 run)

## Evidence

- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "`oggmap` is a python package that can: - extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [methods] of2orthomap         extract orthomap from OrthoFinder output for query species <of2orthomap -h>: "of2orthomap         extract orthomap from OrthoFinder output for query species <of2orthomap -h>"
- [readme] a python package to extract orthologous maps (short: `orthomap` or in other words the evolutionary age of a given orthologous group) from OrthoFinder or eggNOG results.: "[`oggmap`](https://github.com/kullrich/oggmap) is a python package to extract orthologous maps
(short: `orthomap` or in other words the evolutionary age of a given orthologous group) from OrthoFinder"
- [abstract] Here, we propose a Python package called oggmap, which allows for the easy extraction of an orthomap (gene ages per orthogroup): "Here, we propose a Python package called oggmap, which allows for the easy extraction of an orthomap (gene ages per orthogroup) for any given query species from OrthoFinder and other gene family data"
