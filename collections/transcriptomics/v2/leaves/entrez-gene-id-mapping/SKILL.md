---
name: entrez-gene-id-mapping
description: Use when network nodes, ACE tables, or path outputs from SEMgraph/SEMtree
  are keyed by Entrez IDs (e.g., ATG16L1 = 55054, CCR5 = 1234) and must be labeled
  with gene symbols for reporting or benchmark counting.
when_to_use_negative:
- Input graph nodes are already gene symbols (e.g., user-supplied label attributes)
  and no Entrez-to-symbol conversion is needed
- Mapping non-human organisms, where org.Hs.eg.db is the wrong annotation resource
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0602
tools:
- name: org.Hs.eg.db
  role: Bioconductor annotation package queried with mapIds to convert Entrez Gene
    IDs to SYMBOL labels on network nodes
- name: SEMgraph
  role: Provides the igraph networks (KEGG interactome, Steiner tree, path subgraphs)
    whose Entrez-keyed nodes are annotated with symbols
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies Entrez-ID-keyed datasets (e.g., GSE172114 COVID-19 expression data
    and KEGG reference network) whose node names require symbol mapping
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: igraph
  role: Holds the graph object whose V(graph)$name and V(graph)$label attributes are
    read and written during ID conversion
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:7a33d22bc6bb2d75ad258a6e19849412efc0278682480a01c4bb2b749bdb3122
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMpath
  - SEMrun
  - SEMtree()
  - igraph
  - org.Hs.eg.db
  - transformData
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# entrez-gene-id-mapping

## Summary

Converts Entrez Gene IDs used as node names in SEMgraph/SEMtree igraph networks into HGNC gene symbols with Bioconductor's org.Hs.eg.db, so perturbed subnetworks (e.g., the GSE172114 COVID-19 Steiner tree paths) can be reported and validated in standard gene-symbol space.

## When to use

Use when network nodes, ACE tables, or path outputs from SEMgraph/SEMtree are keyed by Entrez IDs (e.g., ATG16L1 = 55054, CCR5 = 1234) and must be labeled with gene symbols for reporting or benchmark counting.

## When NOT to use

- Input graph nodes are already gene symbols (e.g., user-supplied label attributes) and no Entrez-to-symbol conversion is needed
- Mapping non-human organisms, where org.Hs.eg.db is the wrong annotation resource

## Inputs

- igraph network or SEMgraph/SEMtree result table with node names as Entrez Gene IDs (e.g., 55054 for ATG16L1, 1234 for CCR5)
- Bioconductor org.Hs.eg.db annotation package

## Outputs

- Node label attribute V(graph)$label with HGNC gene symbols mapped from ENTREZID keys
- Gene-symbol lists for counting perturbed vs unperturbed genes and computing precision/recall/F1 against reference gene sets

## How to apply

Build the named vector of labels by calling mapIds(org.Hs.eg.db, V(graph)$name, column = 'SYMBOL', keytype = 'ENTREZID') and assign it to the graph's node attribute, e.g. V(alsData$graph)$label <- mapIds(...). Run the conversion after subnetwork extraction (Steiner tree, directed path, or ACE output) so that perturbation statistics remain attached to the correct Entrez-keyed nodes while labels are added for interpretation. Use the symbols to count perturbed versus unperturbed genes (e.g., verify that exactly 2 of the 14 nodes on the ATG16L1→CCR5 path are non-significant at p >= 0.05 in SEMrun's group model) and to match against reference gene sets when computing precision/recall/F1. Rationale: SEMdata reference networks such as KEGG (5007 nodes / 44755 edges) index nodes by Entrez IDs, and the article explicitly suggests org.Hs.eg.db from Bioconductor for gene ID conversion.

## Related tools

- **org.Hs.eg.db** (Bioconductor annotation package queried with mapIds to convert Entrez Gene IDs to SYMBOL labels on network nodes)
- **SEMgraph** (Provides the igraph networks (KEGG interactome, Steiner tree, path subgraphs) whose Entrez-keyed nodes are annotated with symbols) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies Entrez-ID-keyed datasets (e.g., GSE172114 COVID-19 expression data and KEGG reference network) whose node names require symbol mapping) — https://github.com/fernandoPalluzzi/SEMdata
- **igraph** (Holds the graph object whose V(graph)$name and V(graph)$label attributes are read and written during ID conversion)

## Examples

```
V(graph)$label <- mapIds(org.Hs.eg.db, keys = V(graph)$name, column = "SYMBOL", keytype = "ENTREZID")
```

## Evaluation signals

- Key genes resolve to the expected symbols: Entrez 55054 maps to ATG16L1 and Entrez 1234 maps to CCR5
- Every node in the mapped subnetwork receives a SYMBOL label (or a documented NA for genes absent from org.Hs.eg.db)
- Perturbed/unperturbed gene counts computed on symbols match the Entrez-keyed results (14 nodes, 13 edges, exactly 2 unperturbed on the ATG16L1→CCR5 path)
- Symbol-mapped gene lists recover the benchmark counts, e.g., the ST gene F1 = 0.11 against the 92 reference COVID-19 genes

## Limitations

- mapIds returns NA for Entrez IDs without a current SYMBOL annotation, leaving some nodes unlabeled
- Symbol mapping is one-way labeling; downstream SEM fitting still operates on the Entrez-keyed network topology
- org.Hs.eg.db annotations drift over time, so symbol assignments may differ from those frozen in SEMdata benchmark data

## Evidence

- [methods] Gene ID conversion with mapIds (Entrez to gene symbols): "V(alsData$graph)$label <- mapIds(org.Hs.eg.db, V(alsData$graph)$name, column = 'SYMBOL', keytype = 'ENTREZID')"
- [methods] Package rationale for ID conversion: "the use of the R package **org.Hs.eg.db** in the Bioconductor project, for gene ID conversion"
- [readme] README: SEMdata networks are keyed by Entrez-style node identifiers: [evidence span withheld — anchor status: external_doc]
