---
name: cell-type-proportion-summarization
description: Use when per-cell DA scores (or similar per-cell condition-association
  statistics) have been computed on an integrated single-cell dataset with cell-type
  annotations and you need to rank cell types by the fraction of cells linked to a
  condition such as severe disease.
when_to_use_negative:
- Cells lack cell-type annotations, so per-cell DA scores cannot be aggregated into
  per-type proportions
- The input expression matrix has not been integrated across samples/studies (dataset
  variability has not been removed, e.g. with scMerge2), so per-cell abundance differences
  may reflect batch or study effects rather than condition
- The analysis goal is joint integration of multiple modalities — the paper's integration
  was performed within each modality only, deferring multi-modal integration to future
  work
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Provides the adjusted single-cell expression matrix (from the 20-study COVID-19
    collection of ~5 million cells) whose 30-PC embedding is used for DA-seq scoring
    and downstream cell-type proportion summarization
  repo: https://github.com/SydneyBioX/scMerge
- name: DA-seq
  role: Computes per-cell differential abundance scores on the 30-PC embedding (k
    from 50 to 500) that are then aggregated by cell type
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:47b8da570f8b243ba1031e5d9a76544bef3a64a9bbc8f522b244d786d60eac58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - DA-seq
  - scMerge2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cell-type-proportion-summarization

## Summary

Aggregate per-cell differential abundance (DA) scores from a DA-seq analysis of scMerge2-integrated single-cell data by annotated cell type, yielding the proportion of condition-associated cells within each type — used to rank cell types (e.g. neutrophils) by association with severe COVID-19 outcome.

## When to use

Use when per-cell DA scores (or similar per-cell condition-association statistics) have been computed on an integrated single-cell dataset with cell-type annotations and you need to rank cell types by the fraction of cells linked to a condition such as severe disease.

## When NOT to use

- Cells lack cell-type annotations, so per-cell DA scores cannot be aggregated into per-type proportions
- The input expression matrix has not been integrated across samples/studies (dataset variability has not been removed, e.g. with scMerge2), so per-cell abundance differences may reflect batch or study effects rather than condition
- The analysis goal is joint integration of multiple modalities — the paper's integration was performed within each modality only, deferring multi-modal integration to future work

## Inputs

- Per-cell DA-seq differential abundance scores computed on the 30-PC embedding of the scMerge2-adjusted COVID-19 single-cell expression matrix (k swept from 50 to 500)
- Cell-type annotations (colData labels) for each cell in the integrated dataset
- Patient severity labels (moderate vs severe) defining the DA-seq contrast
- UMAP embedding of the scMerge2-adjusted data for overlaying DA scores

## Outputs

- Per-cell DA score file mapping each cell to its differential abundance score
- Cell-type DA-proportion table: proportion of severe-outcome-associated cells within each annotated cell type
- Ranked identification of the cell type with the highest severe-outcome association (neutrophils in the reference analysis)

## How to apply

Start from the scMerge2-adjusted COVID-19 expression matrix reduced to the top 30 principal components, then run DA-seq on this 30-PC embedding comparing moderate versus severe patients, sweeping the neighborhood size k from 50 to 500 to obtain per-cell differential abundance scores. Identify the severity-associated DA regions and overlay the per-cell DA scores on the UMAP of the adjusted data as a sanity/visualization check. Then group per-cell DA scores by the annotated cell type (aggregate score per cell type) and compute, within each cell type, the proportion of cells falling in severe-outcome-associated DA regions. The rationale is that scMerge2's integration — via hierarchical integration, pseudo-bulk building, and pseudo-replication across the 20-study COVID-19 collection of ~5 million cells from 963 donors and 1298 samples — removes dataset variability so that per-cell DA signals can be meaningfully attributed to biological cell states rather than study effects. Applying the aggregation to the integrated COVID-19 data confirms neutrophils hold the highest proportion of severe-outcome-associated cells, consistent with neutrophil accumulation in critical COVID-19 illness.

## Related tools

- **scMerge2** (Provides the adjusted single-cell expression matrix (from the 20-study COVID-19 collection of ~5 million cells) whose 30-PC embedding is used for DA-seq scoring and downstream cell-type proportion summarization) — https://github.com/SydneyBioX/scMerge
- **DA-seq** (Computes per-cell differential abundance scores on the 30-PC embedding (k from 50 to 500) that are then aggregated by cell type)

## Evaluation signals

- The per-cell DA score file covers all cells in the integrated dataset and its scores were computed with k swept across 50–500 neighborhoods
- The cell-type DA-proportion table's proportions sum consistently with the number of cells in each annotated cell type
- Neutrophils emerge as the cell type with the highest proportion of severe-outcome-associated cells, matching the reference finding for the moderate-vs-severe contrast
- Overlaid per-cell DA scores on the UMAP of the adjusted data form spatially coherent DA regions rather than scattered noise
- Cell-state discrimination in the integrated data is improved relative to unintegrated data, consistent with scMerge2 removing dataset variability

## Limitations

- The upstream scMerge2 integration illustrated in the paper was done within each modality; incorporating multi-modal information to better identify pseudo-replicates is deferred to future work
- An embedding output by itself increases interpretability challenges, e.g. for downstream features such as cell–cell interactions or pathway information
- The integration order is data-guided (batches within one study or studies of similar size integrated first); knowledge-guided ordering is described as an unused alternative

## Evidence

- [abstract] scMerge2 enables multi-sample multi-condition integration on the COVID-19 collection: "Using a large COVID-19 data collection with over five million cells from 1000+ individuals, we demonstrate that scMerge2 enables multi-sample multi-condition scRNA-seq data integration"
- [intro] Integration quality supports downstream severity discrimination: "the integration using scMerge2 improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity and facilitates diverse single-cell downstream"
- [discussion] Innovations underlying the adjusted matrix used for DA-seq: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [readme] Where to obtain scMerge/scMerge2 for reproducing the adjusted matrix: [evidence span withheld — anchor status: not_found]
