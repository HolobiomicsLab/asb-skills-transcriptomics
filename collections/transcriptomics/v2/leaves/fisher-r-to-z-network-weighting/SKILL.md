---
name: fisher-r-to-z-network-weighting
description: Use when a directed/undirected reference network must be re-weighted
  from case-control high-throughput data (e.g., GSE172114 COVID-19 RNA-seq) so edge
  costs encode between-group correlation differences prior to Steiner tree or perturbation
  extraction.
when_to_use_negative:
- There is no binary group variable (single-condition data) — Fisher r-to-z tests
  between-group correlation differences, so it is undefined without two conditions.
- The input is already an edge-weighted perturbation graph or a precomputed SEM perturbation
  output rather than raw expression data plus a reference interactome.
- Data are not approximately continuous/normalizable (e.g., raw counts without the
  nonparanormal transform), since r-to-z correlation tests assume transformed expression
  values.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: SEMgraph::weightGraph
  role: Applies the Fisher 'r-to-z' method to weight interactome edges by the group
    difference between correlation coefficients
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph::transformData
  role: Performs the nonparanormal transform on expression data to relax the normality
    assumption before correlation-based weighting
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph::kegg
  role: Supplies the KEGG interactome reference network (5007 nodes / 44755 edges)
    that is re-weighted
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMtree()
  role: Consumes the r2z-weighted graph to extract the perturbed Steiner tree backbone
    (type='ST', eweight='pvalue')
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Provides the GSE172114 COVID-19 dataset and benchmark reference networks/pathways
    used with the weighting workflow
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:7ef544507b25886b80ae5d7724912a71f98bb5d355fdb756a77fb479949cd7e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMgraph::kegg
  - SEMgraph::transformData
  - SEMgraph::weightGraph
  - SEMtree()
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# fisher-r-to-z-network-weighting

## Summary

Weights edges of a reference interactome (e.g., the KEGG interactome of 5007 nodes / 44755 edges) by testing group differences between correlation coefficients with Fisher's 'r-to-z' method, producing a group-difference weighted graph used to extract perturbed backbones with SEMtree().

## When to use

Use when a directed/undirected reference network must be re-weighted from case-control high-throughput data (e.g., GSE172114 COVID-19 RNA-seq) so edge costs encode between-group correlation differences prior to Steiner tree or perturbation extraction.

## When NOT to use

- There is no binary group variable (single-condition data) — Fisher r-to-z tests between-group correlation differences, so it is undefined without two conditions.
- The input is already an edge-weighted perturbation graph or a precomputed SEM perturbation output rather than raw expression data plus a reference interactome.
- Data are not approximately continuous/normalizable (e.g., raw counts without the nonparanormal transform), since r-to-z correlation tests assume transformed expression values.

## Inputs

- Nonparanormal-transformed expression matrix (n samples x p genes) from SEMgraph::transformData
- Binary group vector of experimental conditions (e.g., critical vs non-critical COVID-19 cases from GSE172114)
- Reference interactome as an igraph object (SEMgraph::kegg, 5007 nodes / 44755 edges; also Reactome, STRING, DO, GOBP, GOCC, GOMF available in SEMdata)

## Outputs

- Group-difference edge-weighted igraph object W (same topology as the input interactome, edges re-weighted by Fisher r-to-z correlation-difference tests)
- Weighted graph ready for SEMtree() Steiner tree backbone extraction (type='ST', eweight='pvalue')

## How to apply

First compute per-gene differential expression P-values between patient and control groups (e.g., COVID-19 vs control samples of GSE172114) and relax normality with the nonparanormal transform via SEMgraph::transformData. Then call SEMgraph::weightGraph on the reference interactome (SEMgraph::kegg, 5007 nodes / 44755 edges) with method='r2z', supplying the nonparanormal expression matrix and the binary group vector, so each edge is weighted by the Fisher r-to-z test of the group difference between correlation coefficients. The article notes this r-to-z weighting is 'the fastest' of the available options, making it preferable for very large models like the KEGG interactome. The resulting weighted graph W is passed to SEMtree(W, data, seed=gene P-values, type='ST', eweight='pvalue') to extract Kou's Steiner tree perturbed backbone. Validate by checking that the weighted graph preserves the input node/edge counts and that downstream ST metrics reproduce the benchmark (gene F1 = 0.11, 535 GO terms against 92 reference COVID-19 genes and 1099 reference GO terms).

## Related tools

- **SEMgraph::weightGraph** (Applies the Fisher 'r-to-z' method to weight interactome edges by the group difference between correlation coefficients) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph::transformData** (Performs the nonparanormal transform on expression data to relax the normality assumption before correlation-based weighting) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph::kegg** (Supplies the KEGG interactome reference network (5007 nodes / 44755 edges) that is re-weighted) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMtree()** (Consumes the r2z-weighted graph to extract the perturbed Steiner tree backbone (type='ST', eweight='pvalue')) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Provides the GSE172114 COVID-19 dataset and benchmark reference networks/pathways used with the weighting workflow) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
data.npn <- transformData(exprs)$data; W <- weightGraph(kegg, data = data.npn, group = group, method = "r2z"); st <- SEMtree(W, data.npn, seed = pv, type = "ST", eweight = "pvalue")
```

## Evaluation signals

- Weighted graph W must retain the input interactome topology (KEGG: 5007 nodes, 44755 edges; DNW- check via summary(kegg)).
- Downstream SEMtree ST run on the r2z-weighted graph reproduces the Table 3 benchmark: gene F1 = 0.11 and 535 recovered GO terms against 92 reference COVID-19 genes.
- Input expression data were nonparanormal-transformed (transformData) before weighting; correlations are computed on transformed values.
- Edge weights reflect significant between-group correlation differences; the article identifies r-to-z as the fastest weighting option for very large models.
- Group vector is binary and aligned sample-wise with the expression matrix (e.g., 46 critical vs 23 non-critical COVID-19 cases in GSE172114).

## Limitations

- Fisher r-to-z weighting tests only between-group correlation differences; node-level differential expression signal must be supplied separately as seed gene P-values to SEMtree().
- Correlation-based weighting depends on the nonparanormal transform; skipping it violates the normality assumption underlying r-to-z tests.
- Weighting is tied to the frozen benchmarked versions of reference networks in SEMdata (KEGG update Nov 2021); newer database versions may change results.

## Evidence

- [methods] Weighting with Fisher r-to-z (r2z): "the fastest of them is based on the Fisher's "r-to-z" method, to test the group difference between correlation coefficients"
- [methods] Nonparanormal transform before weighting: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] ST backbone extraction on weighted graph: "# Perturbed backbone as a Steiner tree (Kou's algorithm)"
- [readme] KEGG interactome dimensions: [evidence span withheld — anchor status: external_doc]
- [abstract] COVID-19 benchmark dataset: "We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets"
