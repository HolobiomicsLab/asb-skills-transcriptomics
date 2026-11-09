---
name: cluster-treatment-enrichment-testing
description: Use when a semi-supervised clustering run (e.g. ICAT's 'sslouvain' labels
  on perturbed scRNA-seq cells) must be checked for whether specific treatments are
  over-represented in specific clusters, as in the sea urchin PMC Fig 5A analysis.
when_to_use_negative:
- Cluster labels come from plain unsupervised Louvain on control cells only (no perturbed
  cells assigned) — there is no treatment composition to test
- The goal is differential expression between clusters rather than categorical enrichment
  of treatment labels across clusters
- Treatment labels are missing or ambiguous in the AnnData obs table
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: icat-sc
  role: Provides the semi-supervised clustering whose 'sslouvain' labels feed the
    enrichment tests; installed via pip install icat-sc
  repo: https://github.com/BradhamLab/icat
- name: ICAT
  role: Algorithm producing control-guided clusters of perturbed PMC cells that are
    then tested for treatment enrichment
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Used for log1p normalization, neighbors/Louvain clustering, and UMAP visualization
    of obsm['X_icat'] supporting phenotype interpretation
- name: AnnData
  role: Data structure carrying treatment labels in obs and cluster labels in obs['sslouvain']
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:d5b5d99bc0caf5953148b59088115c2f55ff719b593519939426793f08984601
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cluster-treatment-enrichment-testing

## Summary

Statistical testing of treatment-label enrichment across ICAT-derived cell-state clusters using G-tests and Fisher exact tests, producing a Fig 5A-style contingency table linking perturbations (e.g. MK-886, chlorate) to marker-defined PMC states.

## When to use

Use when a semi-supervised clustering run (e.g. ICAT's 'sslouvain' labels on perturbed scRNA-seq cells) must be checked for whether specific treatments are over-represented in specific clusters, as in the sea urchin PMC Fig 5A analysis.

## When NOT to use

- Cluster labels come from plain unsupervised Louvain on control cells only (no perturbed cells assigned) — there is no treatment composition to test
- The goal is differential expression between clusters rather than categorical enrichment of treatment labels across clusters
- Treatment labels are missing or ambiguous in the AnnData obs table

## Inputs

- AnnData object with final cluster labels in obs['sslouvain'] from model.cluster on the 435-cell GSE164240 PMC dataset
- Per-cell treatment labels in obs (control, MK-886, chlorate)
- Marker gene expression (sm50, pks2) for phenotype annotation of clusters

## Outputs

- Cluster-vs-treatment G-test and Fisher exact test statistics per cluster
- Fig 5A-style enrichment table of significant cluster-treatment associations
- Phenotype map (e.g. cluster 3 = sm50+/pks2−, cluster 4 = sm50−/pks2+)

## How to apply

After ICAT assigns final labels (collect them from the 'sslouvain' column of the returned AnnData), build a cluster-by-treatment contingency table over the 435 SMART-Seq2 cells labeled control, MK-886, or chlorate. For each cluster, run G-tests and Fisher exact tests of treatment counts against cluster membership to identify significant associations. Interpret enrichment biologically by cross-referencing marker phenotypes — e.g. cluster 3 is sm50+/pks2− while cluster 4 is the reciprocal sm50−/pks2+ state — so an enriched cluster can be tied to a perturbation-specific phenotype. Assemble the per-cluster statistics into a Fig 5A-style enrichment table. Correct application means the enrichment pattern matches the reported associations: cluster 3 enriched in one perturbation, cluster 4 in the other.

## Related tools

- **icat-sc** (Provides the semi-supervised clustering whose 'sslouvain' labels feed the enrichment tests; installed via pip install icat-sc) — https://github.com/BradhamLab/icat
- **ICAT** (Algorithm producing control-guided clusters of perturbed PMC cells that are then tested for treatment enrichment) — https://github.com/BradhamLab/icat
- **scanpy** (Used for log1p normalization, neighbors/Louvain clustering, and UMAP visualization of obsm['X_icat'] supporting phenotype interpretation)
- **AnnData** (Data structure carrying treatment labels in obs and cluster labels in obs['sslouvain'])

## Examples

```
out = model.cluster(adata, adata.obs['treatment']); labels = out.obs['sslouvain']; from scipy.stats import fisher_exact; import scipy.stats as stats # G-test / Fisher exact per cluster x treatment
```

## Evaluation signals

- The contingency table resolves exactly five clusters over the 435 cells with control-defined states preserved
- Cluster 3 shows a sm50+/pks2− phenotype and cluster 4 the reciprocal sm50−/pks2+ pattern
- G-test and Fisher exact test enrichment reproduces the Fig 5A associations: cluster 3 enriched in one perturbation, cluster 4 in the other
- All 435 cells carry a non-null 'sslouvain' label and a treatment label (control, MK-886, or chlorate)

## Limitations

- ICAT requires treatment labels for all cells and keeps control labels fixed during semi-supervised clustering, so enrichment tests are only meaningful when controls and perturbed cells were clustered jointly
- ICAT does not automatically compute UMAP/tSNE, so phenotype visualization must be done separately on adata.obsm['X_icat']
- Hyperparameters (Louvain n_neighbors/resolution, NCFS sigma/reg) must be optimized on control cells first; defaults may not transfer to other datasets

## Evidence

- [methods] Task workflow step 7: [evidence span withheld — anchor status: not_found]
- [results] Cluster phenotype definitions: [evidence span withheld — anchor status: not_found]
- [intro] ICAT semi-supervised clustering design: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [readme] Label collection from sslouvain: "The final clustering is stored in the `sslouvain` column of the returned `AnnData` object."
- [readme] Model invocation: "out = model.cluster(adata, adata.obs['treatment'])"
