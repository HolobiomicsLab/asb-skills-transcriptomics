---
name: leiden-over-clustering-of-scrnaseq-data
description: Use when preparing scRNA-seq data (e.g. PBMC 3K processed via the scanpy
  standard workflow) as input to Cytocipher, which requires deliberately over-clustered
  tentative labels from a high Leiden resolution before cluster merging.
when_to_use_negative:
- If you already have a curated, low-resolution cell-type annotation and only want
  to refine specific subpopulations — Cytocipher's merge strategy expects deliberate
  over-clustering, not near-final labels
- If the data has not been processed with the scanpy standard workflow to log-cpm
  normalised values — Cytocipher's code enrichment and merging assume that normalisation
  state
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Runs the standard scRNA-seq processing workflow and the Leiden clustering
    that produces the tentative over-clustered labels consumed by Cytocipher
  repo: https://github.com/BradBalderson/Cytocipher/
- name: AnnData
  role: In-memory object format holding the log-cpm normalised expression matrix and
    the 'leiden' grouping key passed to all Cytocipher functions
- name: cytocipher
  role: Downstream consumer of the over-clustered labels; runs marker identification
    (cc.tl.get_markers), code enrichment (cc.tl.code_enrich), and cluster merging
    (cc.tl.merge_clusters) against the 'leiden' key
  repo: https://github.com/BradBalderson/Cytocipher
- name: Jupyter
  role: Notebook environment in which Cytocipher documentation is inspected (e.g.
    ?cc.tl.get_markers) and the clustering/merging workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:874199ec1902b6c1f267ea6e01045496443856068ee5d02f66ff246124eee58f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# leiden-over-clustering-of-scrnaseq-data

## Summary

Deliberately over-cluster log-cpm normalised scRNA-seq data with Leiden at high resolution to generate tentative cluster labels, which Cytocipher then statistically tests and merges into significantly different cell populations.

## When to use

Use when preparing scRNA-seq data (e.g. PBMC 3K processed via the scanpy standard workflow) as input to Cytocipher, which requires deliberately over-clustered tentative labels from a high Leiden resolution before cluster merging.

## When NOT to use

- If you already have a curated, low-resolution cell-type annotation and only want to refine specific subpopulations — Cytocipher's merge strategy expects deliberate over-clustering, not near-final labels
- If the data has not been processed with the scanpy standard workflow to log-cpm normalised values — Cytocipher's code enrichment and merging assume that normalisation state

## Inputs

- AnnData object processed with the scanpy standard workflow (log-cpm normalised), e.g. the PBMC 3K dataset (.h5ad-style in-memory AnnData)
- Leiden resolution parameter (e.g. 2.0 for PBMC 3K to obtain 19 tentative clusters)

## Outputs

- Tentative over-clustered cell labels stored under a grouping key such as 'leiden' in the AnnData object (19 clusters for PBMC 3K at resolution 2.0)

## How to apply

Process the AnnData object with the scanpy standard workflow to produce log-cpm normalised data, then run Leiden clustering at a high resolution so that there is a lot of over-clustering — for PBMC 3K, resolution 2.0 yields 19 tentative clusters stored under the grouping key 'leiden'. The rationale is that Cytocipher's cc.tl.merge_clusters operates by merging non-significantly different clusters, so an over-clustered starting point is preferable to avoid missing subpopulations (e.g. CD8+ T cell subtypes marked by GZMK and GZMH). Keep the labels under an explicit key (e.g. 'leiden') because downstream Cytocipher calls — cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters, cc.pl.enrich_heatmap — all take that grouping key as an argument. A resolution sweep (0.5–1.5 in the PBMC 3K validation) can be used to check that no resolution resolves true subpopulations without over-clustering other cell types, motivating the high-resolution + merge strategy.

## Related tools

- **scanpy** (Runs the standard scRNA-seq processing workflow and the Leiden clustering that produces the tentative over-clustered labels consumed by Cytocipher) — https://github.com/BradBalderson/Cytocipher/
- **AnnData** (In-memory object format holding the log-cpm normalised expression matrix and the 'leiden' grouping key passed to all Cytocipher functions)
- **cytocipher** (Downstream consumer of the over-clustered labels; runs marker identification (cc.tl.get_markers), code enrichment (cc.tl.code_enrich), and cluster merging (cc.tl.merge_clusters) against the 'leiden' key) — https://github.com/BradBalderson/Cytocipher
- **Jupyter** (Notebook environment in which Cytocipher documentation is inspected (e.g. ?cc.tl.get_markers) and the clustering/merging workflow is executed)

## Examples

```
sc.tl.leiden(adata, resolution=2.0, key_added='leiden')
```

## Evaluation signals

- The 'leiden' key contains deliberately more clusters than expected cell types (PBMC 3K at resolution 2.0 gives 19 tentative clusters versus ~8 ground-truth cell types)
- The AnnData is log-cpm normalised before clustering, matching the scanpy standard workflow requirement
- Downstream Cytocipher calls (cc.tl.get_markers, cc.tl.code_enrich) succeed using the same grouping key, confirming the labels are wired correctly
- After cc.tl.merge_clusters, non-significant over-clusters (CD4+ T cell and CD14+ monocyte sub-clusters) merge after one iteration while genuinely different subpopulations (CD8+ T cell sub-clusters with GZMK/GZMH markers) remain split

## Limitations

- Resolution choice is a trade-off: sweeping Leiden resolution 0.5–1.5 on PBMC 3K showed no resolution resolves the CD8+ T cell subpopulations without over-clustering other cell types, which is why high-resolution over-clustering plus merging is recommended
- Over-clustering increases the number of cluster pairs Cytocipher must statistically test, adding runtime on large datasets (though Cytocipher was shown to scale to the Tabula Sapiens Atlas of >480,000 cells)

## Evidence

- [readme] An AnnData object, *data*, that has been processed similarly to the: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)"
- [readme] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high,"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [other] over-cluster it with Leiden at resolution 2.0 to obtain 19 tentative clusters stored under the grouping key 'leiden': [evidence span withheld — anchor status: not_found]
