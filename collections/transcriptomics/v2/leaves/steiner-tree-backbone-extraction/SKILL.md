---
name: steiner-tree-backbone-extraction
description: Use when you have gene differential-expression P-values (seed genes)
  plus a reference interactome and need a minimal connected subnetwork capturing perturbed
  condition-specific signal for downstream enrichment scoring.
when_to_use_negative:
- When the network is already DAG-structured and you need directed causal paths or
  ACE estimation rather than an undirected minimal connecting subtree (use SEMace/SEMpath
  instead)
- When seeds are a small curated gene list with no quantitative P-values available
  for edge weighting
- When the goal is classification/scoring of samples rather than perturbation extraction
  — other SEMtree() methods may be more appropriate
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: SEMtree()
  role: Runs Kou's Steiner tree algorithm (type='ST', eweight='pvalue') to extract
    the perturbed backbone connecting seed genes
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: Provides weightGraph (Fisher r-to-z edge weighting) and transformData (nonparanormal
    transform) preprocessing for the Steiner tree input
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies the GSE172114 COVID-19 RNA-seq dataset, KEGG interactome (5007 nodes
    / 44755 edges), and reference networks used as inputs
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Gene ID conversion (e.g. Entrez to gene symbols) when mapping tree genes to
    reference sets and GO terms
- name: igraph
  role: Graph representation of the reference interactome and the extracted Steiner
    tree backbone
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:afd0e6e41a7ab1bce5552b2a79537476b112505f4b5629dfc8c2bb948223f5e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - igraph
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Steiner tree backbone extraction

## Summary

Extracts a minimal perturbed backbone subnetwork via Kou's Steiner tree algorithm, connecting all seed (disease) genes at minimum edge cost on a group-difference-weighted KEGG interactome, as implemented by SEMtree() with type='ST' in the SEMgraph R package.

## When to use

Use when you have gene differential-expression P-values (seed genes) plus a reference interactome and need a minimal connected subnetwork capturing perturbed condition-specific signal for downstream enrichment scoring.

## When NOT to use

- When the network is already DAG-structured and you need directed causal paths or ACE estimation rather than an undirected minimal connecting subtree (use SEMace/SEMpath instead)
- When seeds are a small curated gene list with no quantitative P-values available for edge weighting
- When the goal is classification/scoring of samples rather than perturbation extraction — other SEMtree() methods may be more appropriate

## Inputs

- Group-labeled RNA-seq expression matrix (e.g. GSE172114 COVID-19 dataset, 46 critical vs 23 non-critical cases)
- Per-gene differential expression P-values between patient and control groups used as seeds
- Reference interactome as igraph object (e.g. SEMgraph::kegg, 5007 nodes / 44755 edges)
- r2z-weighted graph from SEMgraph::weightGraph
- Reference disease gene list (e.g. 92 COVID-19 genes) and GO term set (e.g. 1099 reference GO terms)

## Outputs

- Steiner tree (perturbed backbone) as an igraph subnetwork connecting all seed genes at minimum cost
- Per-method precision, recall, and F1-score at gene level (reproduced gene F1 = 0.11 for ST) and GO level (535 GO terms recovered)
- st_metrics_table.csv rows matching the ST rows of the benchmark Table 3

## How to apply

First weight the reference network (e.g. SEMgraph::kegg, 5007 nodes / 44755 edges) with SEMgraph::weightGraph using the Fisher 'r-to-z' method (method='r2z') on nonparanormal-transformed expression data (SEMgraph::transformData) so edge weights encode group differences between patient and control samples. Then call SEMtree() with type='ST' and eweight='pvalue', supplying per-gene DE P-values as seeds, which runs Kou's Steiner tree algorithm to connect all seed nodes at minimum cost, yielding the perturbed backbone. The resulting tree can optionally be clustered (e.g. TAHC, edge-betweenness clustering with minimum cluster size 5) for community-level analysis. Finally, score the tree against a reference gene list and mapped GO terms by computing precision, recall, and F1-score to quantify recovery of disease-relevant biology.

## Related tools

- **SEMtree()** (Runs Kou's Steiner tree algorithm (type='ST', eweight='pvalue') to extract the perturbed backbone connecting seed genes) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (Provides weightGraph (Fisher r-to-z edge weighting) and transformData (nonparanormal transform) preprocessing for the Steiner tree input) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies the GSE172114 COVID-19 RNA-seq dataset, KEGG interactome (5007 nodes / 44755 edges), and reference networks used as inputs) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Gene ID conversion (e.g. Entrez to gene symbols) when mapping tree genes to reference sets and GO terms)
- **igraph** (Graph representation of the reference interactome and the extracted Steiner tree backbone)

## Examples

```
st <- SEMtree(W, data, seed = gene_pvalues, type = 'ST', eweight = 'pvalue')
```

## Evaluation signals

- Reproduced gene-level F1-score against the 92 reference COVID-19 genes must equal 0.11 for the ST method
- Number of recovered GO terms mapped from tree genes must equal 535, matching the ST rows of Table 3
- Extracted subgraph must be a connected tree containing all seed (differential expression) genes
- Input graph must have the expected dimension: 5007 nodes and 44755 edges for the KEGG interactome before weighting
- ST performance should be compared against sibling methods (e.g. STr2z gene F1 = 0.12) to confirm ST's rank among active subnetwork detection methods

## Limitations

- ST shows slightly lower gene F1 (0.11) than the r2z variant STr2z (0.12) in the COVID-19 benchmark, so method choice affects sensitivity
- The nonparanormal transform is required to relax the normality assumption; skipping it may distort correlation-based edge weights
- The KEGG interactome and GO/Disease Ontology reference networks in SEMdata are freeze-dated benchmark versions (KEGG update Nov 2021); use loadPathways() or graphite for current versions

## Evidence

- [methods] Steiner tree (ST) backbone extraction via Kou's algorithm: "# Perturbed backbone as a Steiner tree (Kou's algorithm)"
- [methods] Fisher r-to-z weighting of group differences: "the fastest of them is based on the Fisher's "r-to-z" method, to test the group difference between correlation coefficients"
- [methods] Nonparanormal transform precedes SEM analysis: "we apply a nonparanormal transform to relax the normality assumption."
- [results] Benchmark result: ST methods best at identifying COVID-19 genes: [evidence span withheld — anchor status: not_found]
- [readme] Semdata provides the KEGG network dimensions and data inputs: [evidence span withheld — anchor status: external_doc]
