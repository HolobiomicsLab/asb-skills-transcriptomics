---
name: differential-expression-pvalue-seeding
description: Use when you have case/control RNA-seq data (e.g. GSE172114 COVID-19,
  46 critical vs 23 non-critical cases), have computed per-gene DE P-values, and need
  to extract a perturbed subnetwork from a reference interactome such as the KEGG
  graph (5007 nodes / 44755 edges).
when_to_use_negative:
- You already have a curated list of disease seed genes and want a fixed-gene Steiner
  tree rather than data-driven P-value seeding
- Your data are single-group or lack a binary group vector, so no differential-expression
  P-values or r2z edge weights can be computed
- The goal is full SEM model fitting (SEMrun) rather than perturbed backbone extraction
  from seed genes
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
tools:
- name: SEMtree()
  role: Extracts the perturbed backbone as a Steiner tree (type='ST', Kou's algorithm)
    with seed=gene P-values and eweight='pvalue'
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: Provides transformData (nonparanormal transform) and weightGraph (Fisher r-to-z
    edge weighting) to prepare the weighted input graph
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies the GSE172114 COVID-19 RNA-seq dataset and the frozen KEGG reference
    interactome (5007 nodes / 44755 edges) used as inputs
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:9b13ff971cd21e270950f2c55f8234b150676cc92f0d30b96df201914c4e00c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# differential-expression-pvalue-seeding

## Summary

Uses per-gene differential-expression P-values from RNA-seq group comparisons as seed inputs to active-subnetwork detection (Kou's Steiner tree via SEMtree type='ST'), so that perturbed backbone extraction is driven by statistical evidence of differential expression rather than a fixed gene list.

## When to use

Use when you have case/control RNA-seq data (e.g. GSE172114 COVID-19, 46 critical vs 23 non-critical cases), have computed per-gene DE P-values, and need to extract a perturbed subnetwork from a reference interactome such as the KEGG graph (5007 nodes / 44755 edges).

## When NOT to use

- You already have a curated list of disease seed genes and want a fixed-gene Steiner tree rather than data-driven P-value seeding
- Your data are single-group or lack a binary group vector, so no differential-expression P-values or r2z edge weights can be computed
- The goal is full SEM model fitting (SEMrun) rather than perturbed backbone extraction from seed genes

## Inputs

- Case/control RNA-seq expression matrix with binary group vector (e.g. GSE172114 COVID-19: 46 critical vs 23 non-critical cases)
- Per-gene differential-expression P-value vector between groups
- Reference interactome as igraph object (SEMgraph::kegg, 5007 nodes / 44755 edges)

## Outputs

- Perturbed backbone subgraph: minimum-cost Steiner tree (Kou's algorithm) connecting all seed P-value-significant genes
- Tree node set usable for gene-level precision/recall/F1 against a reference disease gene list (e.g. 92 COVID-19 genes; ST gene F1 = 0.11)
- Tree gene set for downstream GO enrichment mapping (535 GO terms recovered in the benchmark)

## How to apply

Compute per-gene differential-expression P-values between patient and control groups from the raw expression matrix (e.g. GSE172114 from GEO). Apply the nonparanormal transform with SEMgraph::transformData to relax the normality assumption before SEM-based fitting. Weight the reference interactome (SEMgraph::kegg) with SEMgraph::weightGraph using the Fisher 'r-to-z' method (method='r2z') so edges carry group-difference evidence. Pass the gene P-value vector as the seed argument to SEMtree(W, data, seed=gene P-values, type='ST', eweight='pvalue'), which runs Kou's algorithm to connect all seed nodes at minimum cost and return the perturbed backbone. The rationale is that seeding from P-values lets the data select which genes anchor the Steiner tree, rather than supplying a static disease-gene list; the resulting tree genes can then be scored against reference sets (e.g. 92 COVID-19 genes, 1099 GO terms) by precision/recall/F1.

## Related tools

- **SEMtree()** (Extracts the perturbed backbone as a Steiner tree (type='ST', Kou's algorithm) with seed=gene P-values and eweight='pvalue') — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (Provides transformData (nonparanormal transform) and weightGraph (Fisher r-to-z edge weighting) to prepare the weighted input graph) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies the GSE172114 COVID-19 RNA-seq dataset and the frozen KEGG reference interactome (5007 nodes / 44755 edges) used as inputs) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
W <- SEMgraph::weightGraph(SEMgraph::kegg, data.npn, group, method="r2z"); st <- SEMtree(W, data.npn, seed=genePvals, type="ST", eweight="pvalue")
```

## Evaluation signals

- Reproduced gene-level F1 against the 92 reference COVID-19 genes must equal 0.11 for the ST method on GSE172114 (matching Table 3)
- Number of recovered GO terms must equal 535, matching the ST rows of Table 3
- Output graph is connected and spans all seed nodes — verify it is a minimum-cost Steiner tree (Kou's algorithm) rather than an arbitrary subgraph
- Input expression data passed through transformData; edge weights of the graph derive from method='r2z' group-difference testing
- Steiner tree node count should be plausible relative to seed count (tree contains seeds plus Steiner connector nodes)

## Limitations

- Benchmark performance is modest at gene level (ST gene F1 = 0.11), so expect partial recovery of the reference disease genes
- The transformData nonparanormal step is required to relax the normality assumption; skipping it may invalidate SEM-based testing
- Results are tied to the frozen KEGG interactome version (Nov 2021, 5007 nodes / 44755 edges) and reference sets in SEMdata; later database versions may shift metrics

## Evidence

- [abstract] from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods.: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [methods] we apply a nonparanormal transform to relax the normality assumption.: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] the fastest of them is based on the Fisher's "r-to-z" method, to test the group difference between correlation coefficients: "the fastest of them is based on the Fisher's "r-to-z" method, to test the group difference between correlation coefficients"
- [methods] Perturbed backbone as a Steiner tree (Kou's algorithm): "Perturbed backbone as a Steiner tree (Kou's algorithm)"
- [readme] COVID-19 RNA-seq dataset of 46 critical and 23 non-critical COVID-19 cases in young patients, from Carapito R. et al., 2022 (GEO accession: GSE172114).: [evidence span withheld — anchor status: external_doc]
