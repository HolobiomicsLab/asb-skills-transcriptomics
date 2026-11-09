---
name: cluster-enrichment-binomial-testing
description: Use when a cell-attribute aware clustering result (e.g. Eva_partitions
  output on a kNN graph with a vertex attribute like age) must be scored for which
  communities are significantly enriched for attribute levels before DA regression.
when_to_use_negative:
- The input is raw count data that has not yet been clustered — enrichment testing
  requires an existing community/partition assignment from Eva_partitions or an equivalent
  clustering step.
- The attribute of interest is not represented as a vertex/cell metadata attribute
  (e.g. no age or phenotype labels attached), since the binomial test scores enrichment
  of that attribute per community.
- Cell-type annotation confidence is low relative to phenotype-label quality — the
  article reports ELVAR is more sensitive to false positives in the clustering attribute
  than in cell-type annotation.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ELVAR
  role: R-package providing the ProcessEVA function that binomial-tests community
    enrichment for the attribute with Bonferroni significance and merges significant
    cells; installed via devtools::install_github("aet21/ELVAR")
  repo: aet21/ELVAR
- name: Seurat
  role: Provides the Seurat object whose metadata (age, mouse replicate) supplies
    the attribute levels being tested for per-community enrichment
- name: R
  role: Language in which the whole ELVAR DA-testing pipeline, including the enrichment-testing
    step, is implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:d8b62d6401afa027009adf1100e023e22529ac5c7f4d9e5419c7f828d8a552fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - ggplot2
  - igraph
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cluster-enrichment-binomial-testing

## Summary

Test each EVA-inferred cell community for enrichment of a condition attribute (e.g. age group) using binomial tests with Bonferroni significance correction, merging significant cells into communities ready for differential abundance regression. This is the ProcessEVA step of the ELVAR pipeline that links cell-attribute aware clustering to DA statistics.

## When to use

Use when a cell-attribute aware clustering result (e.g. Eva_partitions output on a kNN graph with a vertex attribute like age) must be scored for which communities are significantly enriched for attribute levels before DA regression.

## When NOT to use

- The input is raw count data that has not yet been clustered — enrichment testing requires an existing community/partition assignment from Eva_partitions or an equivalent clustering step.
- The attribute of interest is not represented as a vertex/cell metadata attribute (e.g. no age or phenotype labels attached), since the binomial test scores enrichment of that attribute per community.
- Cell-type annotation confidence is low relative to phenotype-label quality — the article reports ELVAR is more sensitive to false positives in the clustering attribute than in cell-type annotation.

## Inputs

- EVA partition output object (eva.o) from Eva_partitions run at chosen alpha (e.g. alpha=0.8, resolution=1)
- Seurat object with cell metadata containing the clustering attribute (e.g. age as 0/1/2/3/5 for 1m/3m/18m/21m/30m)
- Attribute name string (attrName, e.g. 'age')
- Replicate metadata (e.g. mouse ID) for checking replicate coverage of each attribute group

## Outputs

- Significant-cluster object (sigcl.o) with Bonferroni-significant, attribute-enriched merged cell communities
- Merged significant cells per attribute group, ready as input to DoDA negative binomial regression

## How to apply

After running Eva_partitions on the undirected cell-cell nearest-neighbor graph (vertex attributes such as age attached), call ProcessEVA on the EVA partition object and the Seurat object with attrName set to the clustering attribute (e.g. 'age'). For each inferred community, perform a binomial test of enrichment per attribute level, apply Bonferroni correction to define significance, and merge significantly enriched cells across communities into the significant-cluster object. Check that replicate structure is respected — e.g. at least two mouse replicates contribute to each age-group — before proceeding. The resulting sigcl.o object is then fed to DoDA (with varDA such as 'naive' and varREP='mouse') for negative binomial regression of attribute-linked fractions on age, producing Wald and LRT statistics.

## Related tools

- **ELVAR** (R-package providing the ProcessEVA function that binomial-tests community enrichment for the attribute with Bonferroni significance and merges significant cells; installed via devtools::install_github("aet21/ELVAR")) — aet21/ELVAR
- **Seurat** (Provides the Seurat object whose metadata (age, mouse replicate) supplies the attribute levels being tested for per-community enrichment)
- **R** (Language in which the whole ELVAR DA-testing pipeline, including the enrichment-testing step, is implemented)

## Examples

```
sigcl.o <- ProcessEVA(eva.o,seu.o,attrName="age"); da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");
```

## Evaluation signals

- ProcessEVA returns a non-empty set of Bonferroni-significant, attribute-enriched clusters for each expected attribute level (e.g. each age-group).
- Replicate check passes: at least two mouse replicates contribute to each age-group among significant cells.
- Downstream DoDA Wald/LRT statistics show a significant decrease of the naive (CS1) fraction and increase of the mature/activated (CS0) fraction with age, highly significant P-values.
- Significance strength exceeds the Louvain-analog benchmark (LVnonseq), consistent with Fig. 3g of the paper.
- Overlaying the run's purity and modularity on purmodInfo boxplots at alpha=0.8 confirms the tested partition is typical.

## Limitations

- Bonferroni correction is conservative; weakly enriched communities may be discarded even when biologically relevant.
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label), which can degrade community quality.
- No benchmark against miloDE was performed; the authors flag this as an open question.
- Cell-attribute aware clustering and its enrichment testing incur increased computational complexity and runtimes compared to Milo or the Louvain-analog.

## Evidence

- [methods] ProcessEVA call with attrName: "sigcl.o <- ProcessEVA(eva.o,seu.o,attrName="age");"
- [methods] EVA partitions input to enrichment testing: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [readme] ELVAR purpose as attribute-aware DA testing: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data. It implements an Extended Louvain clustering Algorithm (EVA)"
- [methods] DA regression downstream of enrichment testing: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
