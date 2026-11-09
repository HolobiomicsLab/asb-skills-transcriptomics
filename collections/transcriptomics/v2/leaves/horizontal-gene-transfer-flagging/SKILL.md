---
name: horizontal-gene-transfer-flagging
description: Use when per-orthogroup gene trees and species-membership tables are
  available and you need to screen orthogroups for lineage gaps between the LCA gene
  age class and the focal species, i.e. candidate HGT.
when_to_use_negative:
- You only need the gene age (LCA) assignment itself — use oggmap's of2orthomap orthomap
  extraction rather than the continuity/HGT screen
- Orthogroup gene trees or per-orthogroup species-membership tables are unavailable,
  so intermediate-node completeness cannot be evaluated
- The query species has no LCA-to-focal-species path resolvable in the NCBI taxonomy
  lineage (e.g. unresolved lineage), making node-wise completeness undefined
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: oggmap
  role: Provides the LCA-based gene age class (orthomap) per orthogroup and the query
    lineage tools (qlin, ncbitax) used to define the LCA-to-focal-species path
  repo: https://github.com/kullrich/oggmap
- name: OrthoFinder
  role: Supplies the orthogroups and gene trees whose nodes are walked from the LCA
    to the focal species for completeness evaluation
  repo: https://github.com/davidemms/OrthoFinder
- name: Python
  role: Language used with pandas to load the orthomap and species-membership table,
    walk nodes, and write continuity_scores.tsv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:eac82c01d86be208c04dd7d11627dac69435c50748019c5ee1345afa9071fa6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - oggmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# horizontal-gene-transfer-flagging

## Summary

Computes an optional per-orthogroup continuity score quantifying species completeness along tree nodes from the LCA to the focal species (e.g. Danio rerio), so that intermediate nodes lacking detectable orthologs can be flagged as candidate horizontal gene transfer events.

## When to use

Use when per-orthogroup gene trees and species-membership tables are available and you need to screen orthogroups for lineage gaps between the LCA gene age class and the focal species, i.e. candidate HGT.

## When NOT to use

- You only need the gene age (LCA) assignment itself — use oggmap's of2orthomap orthomap extraction rather than the continuity/HGT screen
- Orthogroup gene trees or per-orthogroup species-membership tables are unavailable, so intermediate-node completeness cannot be evaluated
- The query species has no LCA-to-focal-species path resolvable in the NCBI taxonomy lineage (e.g. unresolved lineage), making node-wise completeness undefined

## Inputs

- Per-orthogroup species-membership table produced by oggmap/of2orthomap for Danio rerio (pandas-readable, e.g. TSV)
- Orthomap (gene ages per orthogroup, LCA-based gene age class) for the query species
- OrthoFinder orthogroup gene trees
- Query species lineage from the NCBI taxonomy database (taxadb.sqlite via oggmap qlin)

## Outputs

- continuity_scores.tsv: per-orthogroup continuity score, each value in [0, 1]
- Per-orthogroup list of intermediate nodes lacking detectable orthologs (candidate horizontal gene transfer)

## How to apply

Load the zebrafish (Danio rerio) orthomap and per-orthogroup species-membership table with pandas, taking the LCA assigned by oggmap's of2orthomap logic as the gene age class and starting node of the path toward the focal species. Obtain the query lineage (e.g. via oggmap's qlin on the NCBI taxadb.sqlite database) so the LCA-to-focal-species node path is well defined. For each orthogroup, walk the orthogroup tree nodes from the LCA toward Danio rerio and evaluate species completeness at each intermediate node as the fraction of expected descendant species with detectable orthologs present in the orthogroup. Aggregate node completeness along the path into a single continuity score per orthogroup in the range [0, 1], and record which intermediate nodes lack detectable orthologs. A low continuity score highlights possible horizontal gene transfer, since intermediate nodes should otherwise retain detectable orthologs.

## Related tools

- **oggmap** (Provides the LCA-based gene age class (orthomap) per orthogroup and the query lineage tools (qlin, ncbitax) used to define the LCA-to-focal-species path) — https://github.com/kullrich/oggmap
- **OrthoFinder** (Supplies the orthogroups and gene trees whose nodes are walked from the LCA to the focal species for completeness evaluation) — https://github.com/davidemms/OrthoFinder
- **Python** (Language used with pandas to load the orthomap and species-membership table, walk nodes, and write continuity_scores.tsv)

## Examples

```
from oggmap import qlin; qlin.get_qlin(q='Danio rerio', dbname='taxadb.sqlite')
```

## Evaluation signals

- continuity_scores.tsv exists and every continuity score lies in the range [0, 1]
- Each orthogroup's starting node matches the LCA gene age class assigned by of2orthomap
- Intermediate nodes flagged as lacking orthologs are consistent with the per-orthogroup species-membership table (zero detectable orthologs at those nodes)
- Low-scoring orthogroups correspond to path positions where intermediate nodes have no detectable orthologs, consistent with the candidate-HGT interpretation
- The node path used matches the Danio rerio lineage returned by oggmap qlin from taxadb.sqlite

## Limitations

- The continuity score is optional in oggmap's workflow, not a core gene-age output, and no dedicated HGT sub-command is exposed among the package sub-commands (cds2aa, gtf2t2g, ncbitax, of2orthomap, orthomcl2orthomap, plaza2orthomap, qlin, broccoli2orthomap)
- Low continuity can only flag candidate HGT; intermediate-node gaps may also arise from incomplete data or orthogroup inference artifacts
- oggmap v0.0.2 requires Python 3.10 (Python 2.7x and Python <=3.10 are not supported), so the scoring script must run in a compatible environment

## Evidence

- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [methods] qlin: "show species lineage information from the `NCBI taxonomy <https://www.ncbi.nlm.nih.gov/taxonomy>`_ database."
- [readme] README orthomap definition: [evidence span withheld — anchor status: external_doc]
- [readme] README zebrafish example: [evidence span withheld — anchor status: external_doc]
- [readme] README qlin invocation: [evidence span withheld — anchor status: external_doc]
