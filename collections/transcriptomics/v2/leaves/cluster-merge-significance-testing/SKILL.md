---
name: cluster-merge-significance-testing
description: Use when scRNA-seq data is log-cpm normalised with high-resolution Leiden
  over-clusters and you must decide which clusters are significantly different versus
  which should be merged.
when_to_use_negative:
- Input data lacks tentative cluster labels or has not been log-cpm normalised via
  the scanpy standard workflow — Cytocipher expects over-clustered, normalised AnnData
- Clusters are already known to be biologically distinct and no significance testing
  of merges is needed (the method is for resolving over-clustering, not de novo clustering)
- The goal is scoring cell-cluster membership alone; use cc.tl.code_enrich / benchmark
  scoring methods (Giotto PAGE, Scanpy scoring) rather than the merge-testing workflow
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers, cc.tl.code_enrich, and cc.tl.merge_clusters to
    statistically determine significantly different clusters and merge non-significant
    ones
  repo: https://github.com/BradBalderson/Cytocipher
- name: scanpy
  role: Standard workflow used to produce log-cpm normalised AnnData with tentative
    Leiden cluster labels prior to merge testing
- name: AnnData
  role: Input container format expected by Cytocipher functions
- name: Jupyter
  role: Notebook environment for running Cytocipher and inspecting function documentation
    (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:ab339436f4acdbcb21505a70fb10fb133ff3e76bf1d54d614f3961a309a460b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  - scvelo
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cluster-merge-significance-testing

## Summary

Statistically tests whether over-clustered Leiden groups in scRNA-seq data are significantly different and merges non-significant clusters using Cytocipher, recovering biologically meaningful populations (e.g. CD8+ T cell subtypes, pancreas intermediate states).

## When to use

Use when scRNA-seq data is log-cpm normalised with high-resolution Leiden over-clusters and you must decide which clusters are significantly different versus which should be merged.

## When NOT to use

- Input data lacks tentative cluster labels or has not been log-cpm normalised via the scanpy standard workflow — Cytocipher expects over-clustered, normalised AnnData
- Clusters are already known to be biologically distinct and no significance testing of merges is needed (the method is for resolving over-clustering, not de novo clustering)
- The goal is scoring cell-cluster membership alone; use cc.tl.code_enrich / benchmark scoring methods (Giotto PAGE, Scanpy scoring) rather than the merge-testing workflow

## Inputs

- AnnData object with log-cpm normalised counts and tentative cluster labels (e.g. 'leiden') from over-clustering
- Over-clustered Leiden labels at high resolution (e.g. resolution 2.0 for PBMC 3K, 3.5 for E15.5 pancreas)

## Outputs

- 'leiden_merged' cluster labels stored in the AnnData object
- Cell-by-cluster enrichment scores (from cc.tl.code_enrich)
- Enrichment score heatmap and pair-level significance diagnostic violin plots with p-values

## How to apply

Start from an AnnData object processed with the scanpy standard workflow (log-cpm normalised) carrying tentative Leiden labels at high resolution, since over-clustering gives Cytocipher candidates to merge. Run marker gene identification with cc.tl.get_markers(data, 'leiden'), then compute cell-by-cluster enrichment scores with cc.tl.code_enrich(data, 'leiden'), and finally run cc.tl.merge_clusters(data, 'leiden') to produce 'leiden_merged' labels by merging non-significantly different clusters. Inspect pair-level significance with cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3','9')), which prints p-values comparing enrichment-score violins between cluster pairs, and visualise merged results with cc.pl.enrich_heatmap(data, 'leiden_merged'). Tune the adjusted P-value cutoff only where the paper does so (e.g. 0.045 on the E15.5 pancreas data); otherwise use defaults. Validate by comparing merged assignments against ground-truth cell types or subcluster labels, as in the PBMC 3K (Leiden resolution 2.0, 19 over-clusters) and simulated-data benchmarks.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers, cc.tl.code_enrich, and cc.tl.merge_clusters to statistically determine significantly different clusters and merge non-significant ones) — https://github.com/BradBalderson/Cytocipher
- **scanpy** (Standard workflow used to produce log-cpm normalised AnnData with tentative Leiden cluster labels prior to merge testing)
- **AnnData** (Input container format expected by Cytocipher functions)
- **Jupyter** (Notebook environment for running Cytocipher and inspecting function documentation (e.g. ?cc.tl.get_markers))

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden_merged')
```

## Evaluation signals

- cc.tl.merge_clusters produces a 'leiden_merged' key in the AnnData with fewer clusters than the original over-clustering
- On PBMC 3K at Leiden resolution 2.0, CD4+ T cell and CD14+ monocyte over-clusters merge after one iteration while CD8+ T cell sub-clusters (2 and 8, GZMK/GZMH markers) stay split
- On simulated data with three-way artificial subcluster ground truth, ROC/AUROC of cluster merges exceeds 0.99 regardless of scoring method
- cc.pl.sig_cluster_diagnostics prints p-values for the chosen pair (e.g. p=0.913... for clusters 3 vs 9) and plots upper/lower bounds for significant vs non-significant pairs
- On E15.5 pancreas at Leiden resolution 3.5 with adjusted P-value cutoff 0.045, the merged partition contains 16 distinctly different clusters

## Limitations

- Requires substantial over-clustering: 'It's better if the Leiden resolution is high, so that there is alot of over-clustering'; resolution sweeps (0.5–1.5 on PBMC 3K) may fail to resolve CD8 subpopulations without over-clustering other cell types
- Significance testing compares small groups when clusters contain few cells (e.g. 3 cells; 3 scores in the README example), which may limit statistical power
- Performance depends on the enrichment scoring method: on E18.5 hypothalamus data Scanpy-scoring was the only approach not exceeding AUROC 0.9, whereas coexpr-score reached 0.9715

## Evidence

- [readme] README expected input: [evidence span withheld — anchor status: not_found]
- [readme] README merge example: "Below runs the cluster merging and visualises the heatmap of enrichment scores per cell for each of the new merged clusters."
- [readme] README diagnostics: "the following visualises the scores as violin plots of the enrichment scores & prints the p-values determined by comparing the scores"
- [methods] Article methods workflow: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [abstract] Abstract purpose: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
- [methods] Methods Leiden resolution advice: "tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high"
