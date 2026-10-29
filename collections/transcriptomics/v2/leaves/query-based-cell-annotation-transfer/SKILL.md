---
name: query-based-cell-annotation-transfer
description: Use when new scRNA-seq data (h5ad/AnnData) needs cell type labels or
  continuous fate annotations transferred from a curated reference dataset, especially
  across batches or species, or when screening for putative novel cell types rejected
  by the query.
when_to_use_negative:
- Reference datasets with fewer than ~1000 cells, since the over-parameterized DIRECTi
  model may suffer under-training (3000 cells is generally sufficient)
- Mega-scale dataset integration with highly skewed cell-type composition, where adversarial
  batch alignment may over-correct batches
- Queries against data lacking unified, curated annotations, since no suitable reference
  exists for label transfer
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: 'Core cell-querying tool: builds BLAST index from the reference model, searches
    query cells, and transfers annotations based on posterior-distribution cell-to-cell
    similarity'
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on the reference dataset (via Cell_BLAST.directi.fit_DIRECTi)
    to learn batch-effect-corrected low-dimensional cell embeddings; supports online
    tuning to adapt to the query
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Competing VAE-based method used as a benchmark comparator for annotation-transfer
    accuracy (e.g. JSD ranking in Supplementary Fig. 16b)
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:68e57e6ca0ac463d95cfd3c4eb2f4e8d60fc78f0e7cd7060b0247a71dbf7d309
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - Cell_BLAST.blast.BLAST
  - Cell_BLAST.directi.fit_DIRECTi
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# query-based-cell-annotation-transfer

## Summary

Annotate unlabeled single-cell RNA-seq query cells by searching a curated reference with Cell BLAST, which fits a DIRECTi deep generative model on the reference and transfers reference annotations (cell type or continuous fate probabilities) to query cells via a posterior-distribution-based cell-to-cell similarity metric.

## When to use

Use when new scRNA-seq data (h5ad/AnnData) needs cell type labels or continuous fate annotations transferred from a curated reference dataset, especially across batches or species, or when screening for putative novel cell types rejected by the query.

## When NOT to use

- Reference datasets with fewer than ~1000 cells, since the over-parameterized DIRECTi model may suffer under-training (3000 cells is generally sufficient)
- Mega-scale dataset integration with highly skewed cell-type composition, where adversarial batch alignment may over-correct batches
- Queries against data lacking unified, curated annotations, since no suitable reference exists for label transfer

## Inputs

- Reference scRNA-seq dataset in h5ad (AnnData) format with cell type or fate annotations (e.g. Montoro airway epithelium, Tusi hematopoietic progenitor runs)
- Query scRNA-seq dataset in h5ad (AnnData) format (e.g. Plasschaert airway epithelium, Velten human hematopoiesis)
- One-to-one ortholog gene mapping (for cross-species queries)
- Legacy h5 reference files convertible via cb.data.h5_to_h5ad or h5_to_h5ad.py

## Outputs

- Per-query-cell transferred annotations (Cell Ontology cell types or continuous fate probability vectors)
- List of rejected/unassigned query cells constituting a putative novel cell type cluster
- Differential expression table for rejected clusters (e.g. Cftr, Foxi1, Ascl3 enrichment)
- Quantitative evaluation tables, e.g. MBA scores, per-method JSD comparison (jsd_comparison.csv), per-fate Spearman correlations (fate_probability_spearman.csv)

## How to apply

Load the reference and query datasets as h5ad/AnnData objects (converting legacy h5 via cb.data.h5_to_h5ad if needed) and, for cross-species transfer, map genes to a common space using one-to-one orthologs. Fit a DIRECTi deep generative model on the reference with Cell_BLAST.directi.fit_DIRECTi; the model explicitly handles multilevel batch effect and can optionally be online-tuned to adapt its low-dimensional embeddings to the query's distribution. Build a Cell_BLAST.blast.BLAST index from the fitted model and query the query cells, measuring cell-to-cell similarity as the distance between posterior distributions rather than cosine or Hamming distances that suffer from nonbiological confounding variation. Transfer annotations from hit reference cells to query cells; cell-type hits below the rejection cutoff are flagged as unassigned, signaling a putative novel cell type that should be confirmed by differential expression (e.g. ionocyte markers Cftr, Foxi1, Ascl3). For continuous annotations, transfer multi-dimensional fate probabilities and evaluate with Jensen–Shannon divergence or Spearman correlation against lineage marker expression. Reproduction targets include MBA of 0.873 with 14 of 19 Plasschaert ionocytes rejected against an ionocyte-removed Montoro reference.

## Related tools

- **Cell BLAST** (Core cell-querying tool: builds BLAST index from the reference model, searches query cells, and transfers annotations based on posterior-distribution cell-to-cell similarity) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on the reference dataset (via Cell_BLAST.directi.fit_DIRECTi) to learn batch-effect-corrected low-dimensional cell embeddings; supports online tuning to adapt to the query) — https://github.com/gao-lab/Cell_BLAST
- **scVI** (Competing VAE-based method used as a benchmark comparator for annotation-transfer accuracy (e.g. JSD ranking in Supplementary Fig. 16b))

## Examples

```
import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref); blast = cb.blast.BLAST(model); hits = blast.query(query_data)
```

## Evaluation signals

- Marker-based accuracy (MBA) of transferred annotations matches reported values, e.g. 0.873 on the Plasschaert query against the ionocyte-removed Montoro reference
- Rejection count under the default cutoff equals the reported novel-type signal: 14 of 19 Plasschaert ionocytes left unassigned
- Differential expression on the rejected cluster confirms expected markers (Cftr, Foxi1, Ascl3 for ionocytes)
- Jensen–Shannon divergence between transferred and ground-truth fate probabilities ranks Cell BLAST among the lowest, matching Supplementary Fig. 16b (vs scmap, CellFishing.jl, CCA, scANVI)
- Spearman correlations between predicted fate probabilities and lineage marker expression in the Velten query reproduce the inter-species gains attributed to online tuning (Supplementary Fig. 17)

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- The Cell BLAST model is over-parameterized and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues

## Evidence

- [abstract] Cell BLAST effectively annotates discrete cell types and continuous cell differentiation potential, and can identify novel cell types: "we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential, as well as identifying novel cell types"
- [intro] DIRECTi handles batch effect; similarity is posterior-based: "we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect, as well as a cell-to-cell similarity metric"
- [methods] Install and import via pip: "pip install Cell-BLAST"
- [methods] Legacy h5 conversion command: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] Robustness rationale from discussion: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [readme] README installation in conda environment: [evidence span withheld — anchor status: external_doc]
