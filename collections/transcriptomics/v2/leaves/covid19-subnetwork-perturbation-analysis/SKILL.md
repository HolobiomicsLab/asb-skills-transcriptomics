---
name: covid19-subnetwork-perturbation-analysis
description: Use when a case-vs-control RNA-seq dataset must be reduced to a perturbed
  active subnetwork connecting seed genes at minimum cost, and performance must be
  measured against a reference gene or GO term list.
when_to_use_negative:
- The goal is pathway-level perturbation testing rather than subnetwork extraction
  (use SEMgsa-style pathway analysis instead).
- The input is a pre-computed feature table or already-directed active subnetwork
  — ST expects raw expression data and a P-value-weighted undirected graph.
- The dataset has no group contrast (case/control or critical/non-critical labels),
  since edge weights require group-difference r-to-z testing.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: SEMtree()
  role: Extracts the perturbed backbone as a Steiner tree (Kou's algorithm) from the
    weighted graph using seed gene P-values (type='ST', eweight='pvalue')
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: Provides transformData (nonparanormal transform), weightGraph (Fisher r-to-z
    edge weighting), the KEGG interactome, and the SEMtree() function (v1.2.4+)
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies the GSE172114 COVID-19 RNA-seq dataset and the frozen KEGG interactome
    and pathways for benchmarking
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Gene ID conversion (Entrez to gene symbols via mapIds) needed to align graph
    nodes and expression data
- name: igraph
  role: Graph representation for the KEGG interactome, Steiner tree backbone, and
    TAHC/EBC clustering
  repo: https://igraph.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:60fc3210ee8c71b2a26d5e4db006a235d60448e69a8bf0367c2106ebc385506d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# covid19-subnetwork-perturbation-analysis

## Summary

Extracts a perturbed disease subnetwork (Kou's Steiner tree backbone) from RNA-seq differential expression data using SEMtree()'s ST method, then evaluates gene and GO enrichment recovery against a reference gene/GO set, reproducing the ST benchmark on the GSE172114 COVID-19 dataset.

## When to use

Use when a case-vs-control RNA-seq dataset must be reduced to a perturbed active subnetwork connecting seed genes at minimum cost, and performance must be measured against a reference gene or GO term list.

## When NOT to use

- The goal is pathway-level perturbation testing rather than subnetwork extraction (use SEMgsa-style pathway analysis instead).
- The input is a pre-computed feature table or already-directed active subnetwork — ST expects raw expression data and a P-value-weighted undirected graph.
- The dataset has no group contrast (case/control or critical/non-critical labels), since edge weights require group-difference r-to-z testing.

## Inputs

- GSE172114 COVID-19 RNA-seq expression matrix (46 critical and 23 non-critical cases, Carapito R. et al., 2022)
- Per-gene differential expression P-values between patient and control groups
- KEGG interactome as igraph object (SEMgraph::kegg, 5007 nodes / 44755 edges)
- Reference set of 92 COVID-19-related genes
- Reference set of 1099 Gene Ontology terms

## Outputs

- Steiner tree perturbed backbone (Kou's algorithm) as an igraph object
- Gene-level precision, recall, and F1-score against the 92 reference genes (ST gene F1 = 0.11)
- GO-level precision, recall, and F1-score against the 1099 reference GO terms (535 GO terms recovered)
- Metric table rows (e.g. st_metrics_table.csv) for comparison with Table 3 benchmark values

## How to apply

Load the COVID-19 RNA-seq dataset (GSE172114, 46 critical and 23 non-critical cases) and compute per-gene differential expression P-values between patient and control groups. Apply the nonparanormal transform (SEMgraph::transformData) to relax the normality assumption, then weight the KEGG interactome (SEMgraph::kegg, 5007 nodes / 44755 edges) with SEMgraph::weightGraph using the Fisher 'r-to-z' method (method='r2z') to obtain group-difference edge weights. Extract the perturbed backbone with SEMtree(W, data, seed=gene P-values, type='ST', eweight='pvalue'), i.e. Kou's Steiner tree connecting all seed nodes at minimum cost. Optionally apply Tree agglomerative hierarchical clustering (TAHC) to detect communities within the Steiner tree. Compute precision, recall, and F1 of tree genes against the 92 reference COVID-19 genes, and map tree genes to GO terms (expect 535 recovered GO terms) to compute precision, recall, and F1 against the 1099 reference GO terms.

## Related tools

- **SEMtree()** (Extracts the perturbed backbone as a Steiner tree (Kou's algorithm) from the weighted graph using seed gene P-values (type='ST', eweight='pvalue')) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (Provides transformData (nonparanormal transform), weightGraph (Fisher r-to-z edge weighting), the KEGG interactome, and the SEMtree() function (v1.2.4+)) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies the GSE172114 COVID-19 RNA-seq dataset and the frozen KEGG interactome and pathways for benchmarking) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Gene ID conversion (Entrez to gene symbols via mapIds) needed to align graph nodes and expression data)
- **igraph** (Graph representation for the KEGG interactome, Steiner tree backbone, and TAHC/EBC clustering) — https://igraph.org/

## Examples

```
W <- weightGraph(kegg, data = data.npn, group = group, method = "r2z"); ST <- SEMtree(W, data.npn, seed = pv, type = "ST", eweight = "pvalue")
```

## Evaluation signals

- Reproduced gene-level F1 must equal 0.11 for the ST method (0.12 for STr2z), matching the ST rows of Table 3.
- Number of recovered GO terms must equal 535, matching the ST row of Table 3.
- GO F1 must equal 0.44 against the 1099 reference GO terms.
- Weighted KEGG graph must retain 5007 nodes and 44755 edges (41122 directed and 1582 bidirected).
- Edge weights must come from Fisher r-to-z group-difference tests (method='r2z'), and data must be nonparanormal-transformed before weighting.

## Limitations

- Normality assumptions on expression data must be relaxed via the nonparanormal transform; skipping this can bias SEM fitting.
- Benchmark values depend on the frozen KEGG interactome version in SEMdata (Nov 2021, 225 pathways); using the latest database version may shift metrics.
- Reference sets are fixed (92 COVID-19 genes, 1099 GO terms), so F1 scores are only comparable to other methods evaluated on the same benchmark.
- Discussion section text is absent from the source document.

## Evidence

- [results] ST methods show the best performance in identifying COVID-19-related genes: [evidence span withheld — anchor status: not_found]
- [methods] Kou's Steiner tree extraction: "# Perturbed backbone as a Steiner tree (Kou's algorithm)"
- [methods] nonparanormal transform rationale: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] Fisher r-to-z weighting: "the fastest of them is based on the Fisher's "r-to-z" method, to test the group difference between correlation coefficients"
- [readme] GSE172114 dataset description: [evidence span withheld — anchor status: not_found]
- [readme] KEGG interactome dimensions: [evidence span withheld — anchor status: external_doc]
