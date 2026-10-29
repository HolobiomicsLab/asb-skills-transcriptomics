---
name: binomial-enrichment-testing-with-bonferroni-correction
description: Use when you have cell-attribute-aware clusters (Eva_partitions output
  over an igraph cell-cell nearest-neighbour graph) and need to flag which communities
  are significantly enriched for a condition label before downstream DoDA regression.
when_to_use_negative:
- You only need DA testing on pre-defined annotated cell types without attribute-aware
  community detection — Milo or a Louvain-analog (alpha=0) may suffice and avoids
  the Binomial community-enrichment step.
- 'The cell-cell graph contains cells whose attribute labels are unreliable: the paper
  notes ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype
  label), which can degrade enrichment and community quality.'
- The EVA clustering produced strongly Louvain-like partitions (no deviation from
  alpha=0 cluster count), where attribute enrichment adds little beyond standard Louvain
  community testing.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ELVAR
  role: R package supplying ProcessEVA, the Binomial-test community-enrichment step
    with Bonferroni thresholding, within the ELVAR DA-testing pipeline
  repo: https://github.com/aet21/ELVAR
- name: igraph
  role: Holds the undirected cell-cell graph with vertex attributes whose community-level
    enrichment is tested
- name: Seurat
  role: Provides the Seurat object and per-cell attribute metadata consumed by ProcessEVA
    via seu.o@meta.data
- name: MASS
  role: Negative binomial regression used in the downstream DoDA step that consumes
    the enriched-community sets
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:f144a5420c52da4c53710f0a0111a188bb9d955d614ce0d7a7332a611bd7284b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# binomial-enrichment-testing-with-bonferroni-correction

## Summary

Identify attribute-enriched cellular communities from EVA partitions in the ELVAR pipeline by testing, for each community, whether the frequency of cells carrying a given vertex attribute (e.g. age, Covid-19 severity, disease stage, smell-loss phenotype) exceeds expectation via a Binomial test with a Bonferroni significance threshold.

## When to use

Use when you have cell-attribute-aware clusters (Eva_partitions output over an igraph cell-cell nearest-neighbour graph) and need to flag which communities are significantly enriched for a condition label before downstream DoDA regression.

## When NOT to use

- You only need DA testing on pre-defined annotated cell types without attribute-aware community detection — Milo or a Louvain-analog (alpha=0) may suffice and avoids the Binomial community-enrichment step.
- The cell-cell graph contains cells whose attribute labels are unreliable: the paper notes ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label), which can degrade enrichment and community quality.
- The EVA clustering produced strongly Louvain-like partitions (no deviation from alpha=0 cluster count), where attribute enrichment adds little beyond standard Louvain community testing.

## Inputs

- EVA partitioning output (eva.o) from Eva_partitions run at the selected optimal alpha
- Seurat object with per-cell attribute metadata in meta.data (e.g. age, Covid-19 severity, disease stage, smell-loss phenotype)
- Vertex-attribute-labelled igraph cell-cell nearest-neighbour graph (k.param=20, dims=1:8 PCA)
- Attribute name string for vertex_attr / attrName (e.g. "age")

## Outputs

- Significant enriched-community object (sigcl.o) with Bonferroni-significant Binomial-test P-values per community and attribute group
- Merged attribute-enriched cell sets (cells from multiple enriched clusters per condition group) for downstream DoDA negative-binomial regression

## How to apply

After Eva_partitions clusters the cell-cell graph at the optimal alpha (selected by SelOptAlpha with purity and modularity within the 75th centile of their maxima and a cluster count deviating from standard Louvain at alpha=0), run ProcessEVA on the EVA output and the Seurat object, passing the attribute name (e.g. 'age', 'severity', 'disease stage') as attrName. ProcessEVA performs a Binomial test per community against the overall attribute frequency and applies a Bonferroni threshold to control family-wise error across the many communities tested. Multiple enriched clusters belonging to the same attribute group are merged into a single enriched cell set per group, so the downstream negative-binomial DoDA test compares whole enriched groups rather than individual clusters. Verify replicate/sample coverage per enriched group before running DoDA, and record the enriched community assignments as the significance object (sigcl.o) passed to DoDA with the condition as varDA and sample as varREP.

## Related tools

- **ELVAR** (R package supplying ProcessEVA, the Binomial-test community-enrichment step with Bonferroni thresholding, within the ELVAR DA-testing pipeline) — https://github.com/aet21/ELVAR
- **igraph** (Holds the undirected cell-cell graph with vertex attributes whose community-level enrichment is tested)
- **Seurat** (Provides the Seurat object and per-cell attribute metadata consumed by ProcessEVA via seu.o@meta.data)
- **MASS** (Negative binomial regression used in the downstream DoDA step that consumes the enriched-community sets)

## Examples

```
sigcl.o <- ProcessEVA(eva.o, seu.o, attrName="age"); da.o <- DoDA(seu.o, sigcl.o, varDA="naive", varREP="mouse");
```

## Evaluation signals

- Each enriched community's Binomial test P-value must pass the Bonferroni significance threshold after correction across all tested communities.
- Multiple enriched clusters from the same attribute group must be merged; the final enriched set should cover replicates/patients per group (e.g. check replicate coverage per enriched severity or smell-loss group).
- The parent EVA run must satisfy SelOptAlpha criteria: purity and modularity within the 75th centile of maxima, and cluster count deviating from standard Louvain (alpha=0) at the 95% quantile.
- Downstream DoDA on the merged enriched sets should yield significant Wald/LRT statistics with biologically expected directions (e.g. increased neutrophil and decreased moDC in severe Covid-19; decreased OSN and increased microvillar cells in hyposmic cases).
- The Enriched-community identification should be reproducible across the ~100 EVA runs at the selected alpha.

## Limitations

- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label) than to false positives in cell-type annotation, and FPs in the clustering attribute can affect the quality of the inferred communities.
- The cell-attribute aware pipeline carries increased computational complexity and runtimes compared to Milo or the Louvain-analog, and optimal alpha selection may require ~900 EVA runs (100 runs per alpha over 0.1–0.9).
- No benchmark comparison of ELVAR against miloDE was performed, so the enrichment-regression approach is unvalidated against that 2nd-order kNN-graph extension of Milo.

## Evidence

- [methods] ProcessEVA call: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [methods] ProcessEVA invocation: "sigcl.o <- ProcessEVA(eva.o,seu.o,attrName="age");"
- [readme] ELVAR purpose: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
- [intro] EVA adaptation: "we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method18"
- [abstract] attribute-aware clustering purpose: "uses cell attribute aware clustering when inferring differentially enriched communities within the single-cell manifold"
