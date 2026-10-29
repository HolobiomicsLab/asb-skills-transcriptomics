---
name: gene-set-enrichment-scoring
description: Use when annotating clusters in a Seurat scRNA-seq object against a heterogeneous
  catalog of variable-sized cell-type marker gene sets, and you need bias-corrected
  per-cluster enrichment scores with statistical significance calls.
when_to_use_negative:
- The input is not a clustered single-cell expression object but a bulk or pre-aggregated
  feature table — Miko scoring operates per cell on scRNA-seq data via Seurat
- You need fully automated per-cell annotation without a curated marker catalog —
  marker-based annotations are only as good as the cell-type assigned to the source
  dataset
- The marker sets are derived from the same atlas being annotated — the workflow explicitly
  excludes such sets to avoid circularity
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMiko
  role: Provides nullScore(), mikoScore(), sigScore(), coherentFraction(), and benchmarkScores()/benchmarkCluster()
    functions implementing the scoring pipeline and filters
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework the scoring pipeline is built on; supplies the Seurat object assay/score
    slots consumed by the functions
  repo: https://satijalab.org/seurat/
- name: scPipeline
  role: Rmarkdown module that runs the Miko scoring pipeline and emits dashboard reports
    logging analytic parameters
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:f168784409f1486cf02811212a960eed4834f2e84197e90a50c6b451004452ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-set-enrichment-scoring

## Summary

The Miko scoring pipeline (scMiko) scores single-cell data against cell-type-specific marker gene sets in a three-step workflow — nullScore(), mikoScore(), sigScore() — that corrects for gene set-size bias, then applies hypothesis testing plus coherence and frequent-flier filters to accept or reject candidate cell-type annotations. It benchmarked at AUROC 0.776 against author-curated labels, outperforming GSVA (0.705) and SCINA (0.603).

## When to use

Use when annotating clusters in a Seurat scRNA-seq object against a heterogeneous catalog of variable-sized cell-type marker gene sets, and you need bias-corrected per-cluster enrichment scores with statistical significance calls.

## When NOT to use

- The input is not a clustered single-cell expression object but a bulk or pre-aggregated feature table — Miko scoring operates per cell on scRNA-seq data via Seurat
- You need fully automated per-cell annotation without a curated marker catalog — marker-based annotations are only as good as the cell-type assigned to the source dataset
- The marker sets are derived from the same atlas being annotated — the workflow explicitly excludes such sets to avoid circularity

## Inputs

- Preprocessed, clustered Seurat object (e.g., so_tyser2021_220621.rds, or ps_count_matrix.rds with ps_meta_data.rds)
- Cell-type marker gene set catalog (variable-sized sets, e.g., CellMatch/CellMarker/PanglaoDB-derived, shared across compared methods)

## Outputs

- Per-cell raw and significance-adjusted Miko score matrix
- Per-cluster accepted/rejected cell-type annotations with FDR, miko_score, coherence_fraction, and frequent-flier flag
- Benchmark outputs: ROC curves and per-method AUROC table (e.g., benchmark_auroc_comparison.csv)

## How to apply

Load the preprocessed Seurat object (e.g., so_tyser2021_220621.rds) in R with scMiko and score a shared marker catalog (excluding marker sets derived from the benchmarked atlas itself to avoid circularity). Step 1: run nullScore() with n.replicate=10, nbin=24, gene set sizes 2-200 and step.size=10 to fit a null model that corrects for gene set-size biases. Step 2: score marker sets per cell with mikoScore(); step 3: use sigScore() to identify marker sets significantly enriched in each cluster. Accept/reject candidate annotations via a hypothesis-testing framework (fdr < 0.05, miko_score > 0), then apply the coherence filter (coherence_fraction >= 0.8 via coherentFraction(), enforcing positive correlation between component gene expression and the Miko score) and the frequent-flier filter flagging gene sets exceeding a minimum significance rate (threshold 0.75). Assign cell types per cluster and, for benchmarking, compare against author-curated labels using scMiko benchmarkScores()/benchmarkCluster() to compute ROC curves and AUROC.

## Related tools

- **scMiko** (Provides nullScore(), mikoScore(), sigScore(), coherentFraction(), and benchmarkScores()/benchmarkCluster() functions implementing the scoring pipeline and filters) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework the scoring pipeline is built on; supplies the Seurat object assay/score slots consumed by the functions) — https://satijalab.org/seurat/
- **scPipeline** (Rmarkdown module that runs the Miko scoring pipeline and emits dashboard reports logging analytic parameters) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
df.cscore <- coherentFraction(object = so.query_scored, score.matrix = raw.mat)
```

## Evaluation signals

- Post-scoring filters hold: fdr < 0.05, miko_score > 0, coherence_fraction >= 0.8 for accepted annotations; frequent-flier flag below 0.75 threshold
- Accepted annotations are stable when the null model parameters (n.replicate=10, nbin=24, sizes 2-200, step.size=10) are varied
- Coherence filter returns positive correlation between component gene expression and the Miko score for the required minimum fraction of component genes (via coherentFraction())
- Benchmarked against author-curated labels, AUROC should approach the reported 0.776 and exceed GSVA (0.705) and SCINA (0.603) on the same marker catalog
- Marker sets derived from the benchmarked atlas are confirmed absent from the scoring catalog (no circularity)

## Limitations

- Marker-based annotations are only as good as the cell-type assigned to the original dataset; no validation of source marker catalog curations is provided
- Occasional co-similarities between marker sets from heterogeneous cell types reflect marker curation inaccuracies or cross-cell-type process similarities and are left unresolved
- CDI-based DE used to support the pipeline does not account for variation in biological replicates
- Pipeline depends on cluster quality; in single-replicate datasets, artifact genes highly expressed in a small subset of cells can produce spurious clusters, undermining annotation starting points

## Evidence

- [methods] The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions.: "The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions."
- [methods] nullScore corrects gene set-size bias: "In step 1 we run nullScore(), which fits a null model that corrects for gene set-size biases."
- [discussion] hypothesis-testing accept/reject of annotations: "accepts or rejects candidate annotations using a hypothesis-testing framework"
- [methods] coherence filter definition: "The first is a coherence filter in which a positive correlation between component gene expression and the Miko score is enforced for a minimum fraction of component genes."
- [results] benchmark AUROC result: [evidence span withheld — anchor status: not_found]
- [discussion] marker curation limitation: "we emphasize that marker-based annotations are only as good as the cell-type prescribed to the original dataset."
