---
name: differential-abundance-comparison-with-sccoda
description: Use when testing whether cell type proportions shift between conditions
  (e.g. ischemic vs control) in a clustered single-cell AnnData object, especially
  when samples are unreplicated and bootstrapped calls need concordance checks against
  scCODA on replicate pairs.
when_to_use_negative:
- The input lacks cell type/cluster annotations — correct cell clustering and cell
  type identification is paramount for proportion analysis and must precede this skill
- A cluster is supported by less than 2% of all cells — benchmarking found such rare
  clusters produce unstable results, and bootstrapping has limited benefits for rare
  cell types
- The question is about differential gene expression within cell populations rather
  than shifts in cell type proportions
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Scanpro
  role: 'Primary proportion analysis tool: runs linear-regression testing (ANOVA/t-test)
    with arcsin transform and bootstrapping on the AnnData object; results accessed
    via out.results and plotted via out.plot()'
  repo: https://github.com/loosolab/scanpro
- name: scCODA
  role: Bayesian compositional analysis tool run on pairwise-replicate versions of
    the same atlas to validate the significance calls made by Scanpro's bootstrapping
- name: AnnData
  role: Data container format that Scanpro natively handles, carrying sample, cluster,
    and condition annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:854648f8c4d5806c6addcd089ee749c96714b3d91b8fc2ce6141a9088296dc67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - scCODA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differential-abundance-comparison-with-sccoda

## Summary

Performs differential cell type abundance analysis on annotated single-cell resolution data (AnnData) using Scanpro's linear-regression/bootstrapping approach, and cross-validates significant population shifts against scCODA results on replicated versions of the same dataset.

## When to use

Use when testing whether cell type proportions shift between conditions (e.g. ischemic vs control) in a clustered single-cell AnnData object, especially when samples are unreplicated and bootstrapped calls need concordance checks against scCODA on replicate pairs.

## When NOT to use

- The input lacks cell type/cluster annotations — correct cell clustering and cell type identification is paramount for proportion analysis and must precede this skill
- A cluster is supported by less than 2% of all cells — benchmarking found such rare clusters produce unstable results, and bootstrapping has limited benefits for rare cell types
- The question is about differential gene expression within cell populations rather than shifts in cell type proportions

## Inputs

- AnnData (.h5ad) single-cell dataset with cell type cluster annotations, sample identifiers, and condition labels (e.g. ischemic vs control)
- Pairwise-replicate variants of the same atlas for scCODA cross-comparison

## Outputs

- out.results table of per-cluster proportion analysis statistics with p-values/FDR
- Plots from out.plot() (line or boxplot via the kind parameter)
- Aggregated significant-cell-type CSV (e.g. scanpro_kuppe_results.csv)

## How to apply

Load the annotated atlas (e.g. the Kuppe et al. human MI snRNA-seq dataset) as an AnnData object carrying 'sample', 'cluster', and 'conds' columns, after cells have been clustered and cell-type-annotated. Run the scanpro function with samples_col='sample', clusters_col='cluster', conds_col='conds'; if samples_col is None or absent, Scanpro assumes an unreplicated dataset and applies iteration-based bootstrapping to simulate pseudo-replicates (e.g. 8 pseudo-replicates with arcsin transformation for ischemic-vs-control runs). Extract out.results and identify significantly changed populations (e.g. cardiomyocytes, cycling cells, myeloid cells on the MI atlas), then run scCODA on pairwise-replicate versions of the same atlas and compare its significant detections to the bootstrapped Scanpro calls. Visualize with out.plot(), using the kind parameter for boxplots where informative, and save the significant-cell-type table with FDR/p-values (e.g. scanpro_kuppe_results.csv). Note that Scanpro implements propeller's linear regression approach and extends it with bootstrapping, so unreplicated conclusions should be treated as indicative and require additional experimental validation.

## Related tools

- **Scanpro** (Primary proportion analysis tool: runs linear-regression testing (ANOVA/t-test) with arcsin transform and bootstrapping on the AnnData object; results accessed via out.results and plotted via out.plot()) — https://github.com/loosolab/scanpro
- **scCODA** (Bayesian compositional analysis tool run on pairwise-replicate versions of the same atlas to validate the significance calls made by Scanpro's bootstrapping)
- **AnnData** (Data container format that Scanpro natively handles, carrying sample, cluster, and condition annotations)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'); out.results; out.plot(kind='boxplot')
```

## Evaluation signals

- On the Kuppe et al. human MI atlas, bootstrapped Scanpro calls cardiomyocytes, cycling cells, and myeloid cells as significantly changed between ischemic and control — the three cell types reported by the original publication
- Significant detections are concordant with scCODA results on the pairwise-replicate versions of the same atlas
- Results are retrievable as out.results after the scanpro() call completes without error
- The exported CSV contains FDR/p-value columns for all tested clusters
- Clusters with <2% of all cells are excluded from interpretation, since benchmarking showed they yield unstable results

## Limitations

- Lack of replicates limits drawing definitive conclusions about variability in the data and per se is not advisable; bootstrapping-based results should be interpreted with caution and require additional experimental validation
- Bootstrapping has limited benefits when handling rare cell types, and clusters supported by less than 2% of all cells produce unstable results
- propeller-derived approaches (and by extension the design space Scanpro addresses) depend on replicates for all conditions investigated, which are rare due to cost
- Clustering parameters such as Leiden resolution critically affect the number and size of clusters, so clusters may not reflect real cell types; larger, less specific clusters with higher cell numbers are recommended

## Evidence

- [intro] we here present Scanpro (Single-cell analysis of proportions). Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "we here present Scanpro (Single-cell analysis of proportions). Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality"
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [discussion] Although this approach allows for proportion analysis on non replicated datasets and often results in the same conclusion as replicated datasets (see benchmarks): "Although this approach allows for proportion analysis on non replicated datasets and often results in the same conclusion as replicated datasets"
- [discussion] the results should be seen as an indication of cell types that have a proportional shift and should be interpreted with caution and additional experimental validation: "the results should be seen as an indication of cell types that have a proportional shift and should be interpreted with caution and additional experimental validation"
