---
name: phylogenetic-node-completeness-scoring
description: Use when OrthoFinder rooted gene trees and an of2orthomap-derived orthomap
  (with LCA gene age classes) are available for a focal species such as Danio rerio,
  and you need per-orthogroup completeness along the lineage to flag low-continuity,
  HGT-suspect orthogroups.
when_to_use_negative:
- OrthoFinder was run without gene tree inference (orthogroup membership only) — there
  are no internal tree nodes to walk, so continuity cannot be computed
- The goal is simply assigning gene ages (LCA classes) per gene or calculating TEI
  from scRNA objects — those are handled by of2orthomap and the Scanpy-based TEI workflow,
  not node-completeness scoring
- The orthogroup is single-species or the focal species gene is absent from the tree
  — no LCA-to-focal path exists to score
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0194
tools:
- name: oggmap
  role: supplies the of2orthomap-derived orthomap with LCA gene age classes that define
    the starting node, and qlin/ncbitax for the query species lineage used to orient
    the path
  repo: https://github.com/kullrich/oggmap
- name: OrthoFinder
  role: source of the orthogroups and rooted gene trees whose internal nodes are walked
    for completeness evaluation
  repo: https://github.com/davidemms/OrthoFinder
- name: Python
  role: implementation environment; pandas loads the orthomap/membership tables and
    Bio.Phylo parses and traverses the newick gene trees
  repo: https://github.com/kullrich/oggmap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:8fcdf8afcdd746adb1c4a4beb5ea396dbbdf8d15ea140fc8130d545b63c95a67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - oggmap
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# phylogenetic-node-completeness-scoring

## Summary

Computes an optional per-orthogroup continuity score from oggmap/OrthoFinder output, quantifying species completeness at each tree node along the LCA-to-focal-species path so that intermediate nodes lacking detectable orthologs (candidate horizontal gene transfer) are exposed.

## When to use

Use when OrthoFinder rooted gene trees and an of2orthomap-derived orthomap (with LCA gene age classes) are available for a focal species such as Danio rerio, and you need per-orthogroup completeness along the lineage to flag low-continuity, HGT-suspect orthogroups.

## When NOT to use

- OrthoFinder was run without gene tree inference (orthogroup membership only) — there are no internal tree nodes to walk, so continuity cannot be computed
- The goal is simply assigning gene ages (LCA classes) per gene or calculating TEI from scRNA objects — those are handled by of2orthomap and the Scanpy-based TEI workflow, not node-completeness scoring
- The orthogroup is single-species or the focal species gene is absent from the tree — no LCA-to-focal path exists to score

## Inputs

- OrthoFinder results directory with per-orthogroup rooted gene trees (Gene_Trees) and orthogroup membership files
- Orthomap for the query species produced by oggmap of2orthomap, containing per-gene LCA gene age classes
- Per-orthogroup species-membership table listing which species have detectable orthologs
- Query species lineage/topology from NCBI taxonomy (taxadb.sqlite) via oggmap qlin or qlin.get_lineage_topo
- Focal species name or taxID (e.g. 'Danio rerio', taxID 7955)

## Outputs

- continuity_scores.tsv — one continuity score per orthogroup, each in the range [0, 1]
- List of intermediate tree nodes along the LCA-to-focal-species path that lack detectable orthologs (candidate horizontal gene transfer)

## How to apply

Start from the orthomap and per-orthogroup species-membership table produced by of2orthomap; the LCA-based gene age class assigned by oggmap is the starting node of the path toward the focal species. For each orthogroup, load its rooted gene tree from the OrthoFinder results (e.g. with Bio.Phylo) and walk the nodes from the LCA toward the focal species. At each intermediate node, evaluate species completeness as the fraction of expected descendant species with detectable orthologs present in the orthogroup. Aggregate node completeness along the LCA-to-focal-species path into a single continuity score per orthogroup, recording which intermediate nodes lack detectable orthologs as candidate horizontal gene transfer events. The query lineage topology itself comes from the NCBI taxonomy database via oggmap's qlin sub-command, so scores are interpreted against the correct species tree path.

## Related tools

- **oggmap** (supplies the of2orthomap-derived orthomap with LCA gene age classes that define the starting node, and qlin/ncbitax for the query species lineage used to orient the path) — https://github.com/kullrich/oggmap
- **OrthoFinder** (source of the orthogroups and rooted gene trees whose internal nodes are walked for completeness evaluation) — https://github.com/davidemms/OrthoFinder
- **Python** (implementation environment; pandas loads the orthomap/membership tables and Bio.Phylo parses and traverses the newick gene trees) — https://github.com/kullrich/oggmap

## Examples

```
from Bio import Phylo; tree = Phylo.read('og0000001_tree.nwk','newick'); path = tree.get_path('Danio rerio'); score = sum(completeness(c) for c in path)/len(path)
```

## Evaluation signals

- Every continuity score written to continuity_scores.tsv lies in the range [0, 1]
- For each orthogroup, the path's starting node matches the LCA gene age class recorded in the of2orthomap orthomap
- The node at the focal-species end of the path contains the focal species' gene(s), giving a terminal completeness consistent with orthogroup membership
- Intermediate nodes flagged as lacking orthologs are independently absent from the OrthoFinder orthogroup membership table
- Orthogroups whose path is fully covered by ortholog-containing species receive a continuity score of 1.0

## Limitations

- The continuity score is an optional, secondary computation layered on oggmap's core orthomap extraction — it is not part of the TEI calculation
- A low score indicates intermediate nodes lack detectable orthologs, but this may reflect incomplete taxon sampling, gene loss, or annotation gaps rather than true horizontal gene transfer
- oggmap v0.0.2 was developed with and supports only Python 3.10 (no Python 2.7x or <=3.10), constraining the scripting environment

## Evidence

- [full_text] For each orthogroup, a continuity score can be optionally calculated representing the species completeness along tree nodes from the LCA to the focal species: [evidence span withheld — anchor status: not_found]
- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [methods] show species lineage information from the `NCBI taxonomy <https://www.ncbi.nlm.nih.gov/taxonomy>`_ database.: "show species lineage information from the `NCBI taxonomy <https://www.ncbi.nlm.nih.gov/taxonomy>`_ database."
- [readme] You can get the query species topology as a tree.: [evidence span withheld — anchor status: external_doc]
- [readme] based on pre-calculated OrthoFinder results: [evidence span withheld — anchor status: external_doc]
