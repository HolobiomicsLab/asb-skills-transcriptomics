---
name: orthomap-species-membership-mapping
description: Use when you have an orthomap plus per-orthogroup species-membership
  table from OrthoFinder/oggmap and need lineage-continuity scores per orthogroup
  to spot candidate horizontal gene transfer along the LCA-to-focal-species path.
when_to_use_negative:
- The goal is only to assign gene ages per orthogroup (the orthomap itself) — use
  oggmap's of2orthomap directly; this skill concerns post-hoc continuity scoring,
  not orthomap extraction.
- The downstream task is TEI or partial-TEI computation on scRNA data (Scanpy objects);
  continuity scoring plays no role there and adds no gene-age-weighted expression
  output.
- Orthogroups come from resources other than OrthoFinder/orthomcl/eggNOG/PLAZA outputs
  (e.g. ad-hoc clustering) with no rooted gene trees to walk.
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: oggmap
  role: provides the orthomap, LCA gene age class assignment (of2orthomap) and NCBI-taxonomy
    lineage utilities (qlin, ncbitax) used as the path definition
  repo: https://github.com/kullrich/oggmap
- name: OrthoFinder
  role: source of orthogroups and rooted gene trees whose nodes are walked to compute
    species completeness per node
  repo: https://github.com/davidemms/OrthoFinder
- name: Python
  role: implementation language for the pandas-based walk and score aggregation (oggmap
    v0.0.2 requires Python 3.10)
  repo: https://github.com/kullrich/oggmap
- name: NCBI taxonomy database
  role: provides species lineage information defining the LCA-to-focal-species path
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:7764d15ac929e1cfb4e0952c089829a5dc4a43bb42eabfb942a9e9db67d209a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - NCBI taxonomy database
  - OrthoFinder
  - Python
  - oggmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# orthomap-species-membership-mapping

## Summary

Reconstructs a per-orthogroup species completeness (continuity) score by walking orthogroup tree nodes from the LCA gene-age node assigned by oggmap's of2orthomap toward the focal species (e.g. Danio rerio), quantifying how completely species are represented along the lineage and flagging intermediate nodes without detectable orthologs as possible horizontal gene transfer.

## When to use

Use when you have an orthomap plus per-orthogroup species-membership table from OrthoFinder/oggmap and need lineage-continuity scores per orthogroup to spot candidate horizontal gene transfer along the LCA-to-focal-species path.

## When NOT to use

- The goal is only to assign gene ages per orthogroup (the orthomap itself) — use oggmap's of2orthomap directly; this skill concerns post-hoc continuity scoring, not orthomap extraction.
- The downstream task is TEI or partial-TEI computation on scRNA data (Scanpy objects); continuity scoring plays no role there and adds no gene-age-weighted expression output.
- Orthogroups come from resources other than OrthoFinder/orthomcl/eggNOG/PLAZA outputs (e.g. ad-hoc clustering) with no rooted gene trees to walk.

## Inputs

- Orthomap (gene ages / LCA gene age class per orthogroup) extracted from OrthoFinder results with oggmap of2orthomap
- Per-orthogroup species-membership table (orthogroup to species with detectable orthologs) from OrthoFinder output
- Query species lineage/topology from the local NCBI taxonomy database (taxadb.sqlite) obtained with oggmap qlin
- Orthogroup gene trees (rooted per-gene trees inferred by OrthoFinder)

## Outputs

- continuity_scores.tsv — per-orthogroup continuity scores in [0, 1]
- Per-orthogroup list of intermediate lineage nodes lacking detectable orthologs (candidate horizontal gene transfer)

## How to apply

Load the query-species orthomap (e.g. the Danio rerio orthomap extracted from OrthoFinder results via of2orthomap) and the per-orthogroup species-membership table with pandas. For each orthogroup, take the LCA-based gene age class assigned by of2orthomap as the starting node of the path toward the focal species, using lineage information obtained from the NCBI taxonomy database (via oggmap's qlin/ncbitax subcommands). At each intermediate node on the path, compute species completeness as the fraction of expected descendant species that have detectable orthologs present in the orthogroup. Aggregate node completeness along the path into a single continuity score in the range [0, 1] and record which intermediate nodes lack detectable orthologs. Rationale: a low continuity score indicates gaps in species representation along the lineage, which the article treats as a signal of possible horizontal gene transfer rather than vertical inheritance.

## Related tools

- **oggmap** (provides the orthomap, LCA gene age class assignment (of2orthomap) and NCBI-taxonomy lineage utilities (qlin, ncbitax) used as the path definition) — https://github.com/kullrich/oggmap
- **OrthoFinder** (source of orthogroups and rooted gene trees whose nodes are walked to compute species completeness per node) — https://github.com/davidemms/OrthoFinder
- **Python** (implementation language for the pandas-based walk and score aggregation (oggmap v0.0.2 requires Python 3.10)) — https://github.com/kullrich/oggmap
- **NCBI taxonomy database** (provides species lineage information defining the LCA-to-focal-species path)

## Examples

```
from oggmap import qlin; qlin.get_qlin(q='Danio rerio', dbname='taxadb.sqlite')
```

## Evaluation signals

- continuity_scores.tsv exists and every score lies in the range [0, 1]
- Each orthogroup's path starts at the LCA node that matches the gene age class assigned by of2orthomap for that orthogroup
- The number of scored orthogroups equals the number of rows in the input orthomap
- Intermediate nodes recorded as lacking orthologs correspond exactly to nodes where the completeness fraction is 0 (or below threshold) in the species-membership table
- Spot-check: an orthogroup with all descendant species represented at every node scores 1.0, and injecting a missing species monotonically lowers its score

## Limitations

- A low continuity score highlights possible horizontal gene transfer but is only a candidate signal — annotation gaps or incomplete taxon sampling produce the same pattern.
- oggmap v0.0.2 was developed with Python 3.10; Python 2.7x and versions <=3.10 are not supported, constraining the implementation environment.
- Species completeness depends on the set of species included in the OrthoFinder run; expected descendant species are defined by that input dataset, not by all known species.

## Evidence

- [methods] LCA-based continuity scoring rationale: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [abstract] orthomap extraction scope: "which allows for the easy extraction of an orthomap (gene ages per orthogroup) for any given query species"
- [methods] lineage information source: "show species lineage information from the `NCBI taxonomy <https://www.ncbi.nlm.nih.gov/taxonomy>`_ database."
- [readme] orthogroup trees provided by OrthoFinder: [evidence span withheld — anchor status: external_doc]
- [methods] implementation environment constraint: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."
