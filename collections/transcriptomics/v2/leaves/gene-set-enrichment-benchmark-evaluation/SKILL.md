---
name: gene-set-enrichment-benchmark-evaluation
description: Use when a directed/undirected subnetwork extraction method must be quantitatively
  compared against active-subnetwork alternatives on a case-vs-control RNA-seq benchmark
  with known disease genes and GO term references.
when_to_use_negative:
- The goal is causal structure or perturbation statistics (node DRN / edge DRE testing
  via SEMrun), not enrichment benchmarking — use SEMrun fit=1/fit=2 perturbation models
  instead
- No reference gene or GO-term benchmark exists for the condition, so precision/recall/F1
  against a truth set cannot be computed
- The dataset is not a case-vs-control high-throughput expression study (e.g., the
  Sachs flow-cytometry data), so group-difference r2z edge weighting and disease-gene
  seeds are not defined
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: SEMtree()
  role: Extracts the perturbed backbone as Kou's Steiner tree (type='ST', eweight='pvalue')
    whose gene and GO content is scored
  repo: https://github.com/fernandoPalluzzi/SEMgraph/tree/master/SEMtree
- name: SEMgraph
  role: Supplies weightGraph (Fisher r-to-z edge weighting), transformData (nonparanormal
    transform), and the kegg interactome; SEMtree() is implemented in this R package
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Provides the GSE172114 COVID-19 dataset and the benchmarked KEGG reference
    networks/pathways
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Gene ID conversion (e.g., Entrez to SYMBOL) needed to map tree genes to GO
    terms
- name: igraph
  role: Graph object representation of the weighted KEGG interactome and extracted
    Steiner tree
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:2fd77abdb8486e4b5ea3496f1cbb43d899d60cf7d16ddc3fbca00d7d1864ff4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - igraph
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-set-enrichment-benchmark-evaluation

## Summary

Benchmark a subnetwork-detection method (here SEMtree's Steiner tree, type='ST') by computing precision, recall, and F1-score at both the gene level (against 92 reference COVID-19 genes) and the Gene Ontology term level (against 1099 reference GO terms) on the GSE172114 dataset, targeting reproduction of the paper's Table 3 values (gene F1 = 0.11, 535 GO terms recovered).

## When to use

Use when a directed/undirected subnetwork extraction method must be quantitatively compared against active-subnetwork alternatives on a case-vs-control RNA-seq benchmark with known disease genes and GO term references.

## When NOT to use

- The goal is causal structure or perturbation statistics (node DRN / edge DRE testing via SEMrun), not enrichment benchmarking — use SEMrun fit=1/fit=2 perturbation models instead
- No reference gene or GO-term benchmark exists for the condition, so precision/recall/F1 against a truth set cannot be computed
- The dataset is not a case-vs-control high-throughput expression study (e.g., the Sachs flow-cytometry data), so group-difference r2z edge weighting and disease-gene seeds are not defined

## Inputs

- GSE172114 COVID-19 RNA-seq expression matrix (46 critical vs 23 non-critical cases, from SEMdata)
- Per-gene differential expression P-values between patient and control groups
- KEGG interactome graph (SEMgraph::kegg, 5007 nodes / 44755 edges)
- Reference set of 92 COVID-19 genes
- Reference set of 1099 COVID-19 GO terms
- org.Hs.eg.db mapping for gene ID conversion

## Outputs

- Steiner tree (ST) perturbed backbone subnetwork from SEMtree()
- Gene-level precision, recall, F1-score table (ST row: gene F1 = 0.11)
- GO-term-level precision, recall, F1 table (ST row: 535 GO terms recovered)
- st_metrics_table.csv reproducing the ST rows of Table 3

## How to apply

Load the GSE172114 COVID-19 RNA-seq data (46 critical, 23 non-critical young cases; Carapito R. et al., 2022) and compute per-gene differential expression P-values between patient and control groups. Apply SEMgraph::transformData for a nonparanormal transform to relax the normality assumption on the expression matrix. Weight the KEGG interactome (SEMgraph::kegg, 5007 nodes / 44755 edges) with SEMgraph::weightGraph using the Fisher 'r-to-z' method (method='r2z') so edge weights encode group-difference correlation tests. Extract the perturbed backbone with SEMtree(W, data, seed=gene P-values, type='ST', eweight='pvalue') — Kou's algorithm connecting all seed nodes at minimum cost — then compute precision, recall, and F1 of the tree's genes against the 92 reference COVID-19 genes, and map tree genes to GO terms (e.g., via org.Hs.eg.db gene ID conversion) to compute precision, recall, and F1 against the 1099 reference GO terms. Decision point: compare ST against the other four active-subnetwork methods after converting all subnetworks to directed trees via Chu–Liu–Edmonds' causal additive trees, so the comparison is in terms of directed active subnetworks; ST should show the best gene-level F1 (0.11 for ST, 0.12 for STr2z) and recover 535 GO terms.

## Related tools

- **SEMtree()** (Extracts the perturbed backbone as Kou's Steiner tree (type='ST', eweight='pvalue') whose gene and GO content is scored) — https://github.com/fernandoPalluzzi/SEMgraph/tree/master/SEMtree
- **SEMgraph** (Supplies weightGraph (Fisher r-to-z edge weighting), transformData (nonparanormal transform), and the kegg interactome; SEMtree() is implemented in this R package) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Provides the GSE172114 COVID-19 dataset and the benchmarked KEGG reference networks/pathways) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Gene ID conversion (e.g., Entrez to SYMBOL) needed to map tree genes to GO terms)
- **igraph** (Graph object representation of the weighted KEGG interactome and extracted Steiner tree)

## Examples

```
W <- weightGraph(kegg, data = transformData(exprs)$data, group = group, method = "r2z"); ST <- SEMtree(W, data.npn, seed = pvalues, type = "ST", eweight = "pvalue")
```

## Evaluation signals

- Reproduced gene-level F1 must equal 0.11 for the ST method (0.12 for STr2z), matching the Table 3 ST rows
- Number of recovered GO terms must equal 535, matching the ST row of Table 3
- Edge weights derive from the Fisher r-to-z test of group correlation differences; verify weightGraph was called with method='r2z' on the NPN-transformed data
- ST must attain the highest gene F1-score among all subnetwork methods compared, as reported: 'The ST methods show the best performance in identifying COVID-19-related genes'
- KEGG interactome integrity check: graph must have 5007 nodes and 44755 edges (DNW- 5007 44755)

## Limitations

- The Discussion section text is absent from the source document, so caveats on generalization are not available in context
- Reference networks and pathways are freezed to benchmarked versions for SEMgraph (KEGG update Nov 2021, 225 pathways); using latest database versions instead of the frozen SEMdata versions may shift F1 values
- The normality assumption must be relaxed via the nonparanormal transform before r-to-z weighting; skipping it risks invalid correlation tests

## Evidence

- [results] ST methods outperform all alternatives on COVID genes: [evidence span withheld — anchor status: not_found]
- [abstract] Perturbed modules from seed genes/P-values with five methods: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [abstract] Chu–Liu–Edmonds conversion for directed comparison: "This conversion allows to compare the methods in terms of directed active subnetworks."
- [methods] Nonparanormal transform rationale: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] Kou Steiner tree backbone extraction: "# Perturbed backbone as a Steiner tree (Kou's algorithm)"
- [readme] GSE172114 dataset description: [evidence span withheld — anchor status: external_doc]
- [readme] KEGG interactome frozen size: [evidence span withheld — anchor status: external_doc]
