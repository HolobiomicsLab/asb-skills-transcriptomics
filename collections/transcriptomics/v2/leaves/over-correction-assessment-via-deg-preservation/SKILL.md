---
name: over-correction-assessment-via-deg-preservation
description: Use when benchmarking single-cell batch correction methods (scCobra,
  Harmony, Scanorama, scVI, Seurat) on two same-condition samples and you need to
  detect over-correction by comparing DEG counts before vs. after correction.
when_to_use_negative:
- The two samples come from different conditions (e.g. healthy vs. disease) — true
  biological differences would then produce legitimate DEGs, and DEG reduction would
  wrongly flag over-correction, which the article describes as ignoring 'the potential
  inherent difference associated with different batches'.
- Only one sample per condition is available, since the metric relies on a pairwise
  DE test between two same-condition samples.
- You need batch-correction metrics other than over-correction (e.g. bio-conservation
  scoring via the scIB pipeline with Leiden clustering at resolutions 0.1–2.0).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCobra
  role: Batch-correction method under test; deep generative network combining contrastive
    learning with domain adaptation that produced the fewest post-correction DEGs
    (30)
  repo: https://github.com/mcgilldinglab/scCobra
- name: scanpy
  role: Preprocessing (filtering, normalization to 10,000 counts, log1p), T/NK cell
    extraction, and differential expression testing with log2FC > 0.6 and adjusted
    P < 0.05 thresholds
- name: Harmony
  role: Benchmark batch-correction method (40 post-correction DEGs)
- name: Scanorama
  role: Benchmark batch-correction method included in the comparison
- name: scVI
  role: Benchmark batch-correction method run via scvi-tools by adjusting the 'n_batch'
    parameter (66 post-correction DEGs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:3f21cb7bdd592c50066b7d359334b0689ccc270ea4b209b9ef76e50bd33d019a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - scCobra
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# over-correction-assessment-via-deg-preservation

## Summary

Quantifies batch-correction over-correction by counting differentially expressed genes (DEGs) between two healthy samples from the same condition after applying each correction method; a well-behaved method (e.g. scCobra) should shrink raw DEG counts (470 for GSE149614 raw) toward few DEGs while over-correcting or under-correcting methods leave more (Harmony 40, scVI 66).

## When to use

Use when benchmarking single-cell batch correction methods (scCobra, Harmony, Scanorama, scVI, Seurat) on two same-condition samples and you need to detect over-correction by comparing DEG counts before vs. after correction.

## When NOT to use

- The two samples come from different conditions (e.g. healthy vs. disease) — true biological differences would then produce legitimate DEGs, and DEG reduction would wrongly flag over-correction, which the article describes as ignoring 'the potential inherent difference associated with different batches'.
- Only one sample per condition is available, since the metric relies on a pairwise DE test between two same-condition samples.
- You need batch-correction metrics other than over-correction (e.g. bio-conservation scoring via the scIB pipeline with Leiden clustering at resolutions 0.1–2.0).

## Inputs

- Two single-cell RNA-seq samples from the same condition (e.g. healthy liver cancer samples HCC03N and HCC04N from GSE149614)
- Raw count matrix (.h5ad) for each sample with cell/gene metadata
- Corrected expression matrices (.h5ad) from each batch-correction method: scCobra, Harmony, Scanorama, scVI

## Outputs

- DEG counts per condition (raw, scCobra, Harmony, Scanorama, scVI) tabulated in deg_counts_S3.csv
- Comparison against the Supplementary Fig. S3 pattern (reference: raw=470, scCobra=30, Harmony=40, scVI=66 DEGs)
- Per-method corrected .h5ad matrices annotated with the T/NK cell population

## How to apply

Obtain two healthy samples from the same condition (e.g. HCC03N and HCC04N from the GSE149614 liver cancer dataset), preprocess them in scanpy (drop cells with <200 expressed genes and genes detected in <3 cells, remove cells with >5% mitochondrial fraction, normalize to 10,000 counts per cell and log1p transform, select 2000 highly variable genes), then apply each batch-correction method — scCobra, Harmony, Scanorama, and scVI (scvi-tools, adjusting the 'n_batch' parameter) — saving each corrected matrix as .h5ad. Annotate and extract the T/NK cell population from the raw and each corrected matrix so that all DE tests run on a matched cell type, since batch noise should not produce real DEGs between same-condition samples while true biology must survive. Run differential expression between the two samples on the raw and each corrected matrix, calling DEGs at log2FC > 0.6 and adjusted P < 0.05, and tabulate counts per condition (raw, scCobra, Harmony, Scanorama, scVI) into deg_counts_S3.csv for comparison against Supplementary Fig. S3. A smaller post-correction DEG count indicates better batch-noise removal without over-correction of biological signal.

## Related tools

- **scCobra** (Batch-correction method under test; deep generative network combining contrastive learning with domain adaptation that produced the fewest post-correction DEGs (30)) — https://github.com/mcgilldinglab/scCobra
- **scanpy** (Preprocessing (filtering, normalization to 10,000 counts, log1p), T/NK cell extraction, and differential expression testing with log2FC > 0.6 and adjusted P < 0.05 thresholds)
- **Harmony** (Benchmark batch-correction method (40 post-correction DEGs))
- **Scanorama** (Benchmark batch-correction method included in the comparison)
- **scVI** (Benchmark batch-correction method run via scvi-tools by adjusting the 'n_batch' parameter (66 post-correction DEGs))

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; pip install scanpy scib; jupyter notebook run_integration.ipynb
```

## Evaluation signals

- Raw (uncorrected) DEG count between the two healthy samples should be high (470 for GSE149614 HCC03N vs. HCC04N); if raw DEGs are near zero the batch-effect simulation is not set up correctly.
- Each corrected matrix should yield FEWER DEGs than raw, with scCobra expected at ~30, Harmony ~40, scVI ~66 per Supplementary Fig. S3; a corrected count exceeding raw indicates the correction ran incorrectly.
- DEG calls must use the fixed thresholds log2FC > 0.6 and adjusted P < 0.05; changing thresholds invalidates comparability with the reference pattern.
- DE tests must be restricted to the extracted T/NK cell population in both raw and corrected matrices — mixing cell types would confound batch effect with cell-type composition.
- The corrected outputs are valid .h5ad files with an expression matrix per method, and the final counts land in deg_counts_S3.csv with one row per method.

## Limitations

- Benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, so counts may not reflect their tuned performance.
- DEG-based over-correction was assessed between two healthy samples only; the article also derives over-correction scores from two-dimensional UMAP embeddings (following the SCALEX methodology), which introduces dependence on UMAP.
- scVI assumes a gene expression distribution (e.g. negative binomial), which the article notes cannot be universal across datasets and platforms; results may vary with distributional fit.

## Evidence

- [abstract] scCobra effectively mitigates batch effects, minimizes over-correction, and ensures biologically meaningful data integration without assuming specific gene expression distributions.: "scCobra effectively mitigates batch effects, minimizes over-correction, and ensures biologically meaningful data integration without assuming specific gene expression distributions."
- [discussion] using two healthy samples (HCC03N and HCC04N) from the liver cancer dataset: "using two healthy samples (HCC03N and HCC04N) from the liver cancer dataset"
- [discussion] we first filtered out cells that expressed fewer than 200 genes: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression: "we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression"
- [readme] Please refer run_integration.ipynb: [evidence span withheld — anchor status: external_doc]
