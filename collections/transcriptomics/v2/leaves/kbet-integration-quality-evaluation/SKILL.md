---
name: kbet-integration-quality-evaluation
description: Use when two gene-by-barcode matrices (e.g., a Chromium reference GBM
  and a platform-specific GBM) have been integrated and you need quantitative metrics
  of batch-effect removal and cluster separation.
when_to_use_negative:
- When the two GBMs have not yet been harmonised to matching barcodes and genes, since
  kBET and Silhouette require comparable cell and gene sets
- When comparing within-platform replicates where no batch effect is expected, making
  kBET acceptance rates uninformative
- When the goal is direct GBM agreement (Pearson correlation) or clustering concordance
  (ARI) rather than integration quality
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R package used to integrate each comparison GBM independently with the reference
    GBM prior to quality evaluation
  repo: https://github.com/minoda-lab/universc
- name: kBET
  role: R package (version 0.99.6) used to compute the kBET metric quantifying batch-effect
    removal in each integration
- name: cluster
  role: R package (version 2.1.3) used to compute Silhouette scores assessing cluster
    distinctness after integration
- name: UniverSC
  role: Produces one of the two SmartSeq2 GBMs being compared (uniform pipeline for
    all datasets)
  repo: https://github.com/minoda-lab/universc
- name: zUMIs
  role: Platform-specific pipeline producing the alternative SmartSeq2 GBM baseline
    for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:b0eb379d557b37b3da405a26f084dc1c28db9993ed9218b5ced9d68b7c2f3af6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - Seurat
  - UniverSC
  - cluster
  - kBET
  - zUMIs
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kbet-integration-quality-evaluation

## Summary

Evaluates the quality of single-cell data integration between a reference and a comparison dataset (e.g., Chromium vs SmartSeq2 GBMs integrated with Seurat) by computing kBET batch-effect acceptance rates and Silhouette scores, as done in the UniverSC paper to compare cross-platform processing pipelines.

## When to use

Use when two gene-by-barcode matrices (e.g., a Chromium reference GBM and a platform-specific GBM) have been integrated and you need quantitative metrics of batch-effect removal and cluster separation.

## When NOT to use

- When the two GBMs have not yet been harmonised to matching barcodes and genes, since kBET and Silhouette require comparable cell and gene sets
- When comparing within-platform replicates where no batch effect is expected, making kBET acceptance rates uninformative
- When the goal is direct GBM agreement (Pearson correlation) or clustering concordance (ARI) rather than integration quality

## Inputs

- Seurat-integrated object from a reference GBM (e.g., Chromium) and a comparison GBM (e.g., SmartSeq2 processed by UniverSC or zUMIs)
- Gene-by-barcode matrices harmonised to shared gene and barcode sets (intersecting barcodes kept, missing genes filled with 0 UMIs)

## Outputs

- kBET acceptance-rate value per integration (e.g., ~0.06 for UniverSC vs ~0.11 for zUMIs)
- Silhouette score per integration (e.g., ~0.43 for UniverSC vs ~0.36 for zUMIs)
- integration_scores.csv containing kBET and Silhouette values for both integrations

## How to apply

After integrating each comparison GBM (e.g., UniverSC- or zUMIs-processed SmartSeq2 from mouse primary cells, mapped to the full mouse genome GRCm39 with no downsampling) with a reference GBM using the Seurat integration workflow in R, compute kBET with the R package kBET (version 0.99.6) and the Silhouette score with the R package cluster (version 2.1.3) for each integration independently. Lower kBET indicates better batch-effect removal; higher Silhouette indicates more distinct clusters. Harmonise the input GBMs first by keeping only intersecting barcodes and adding missing genes with 0 UMIs so both matrices share identical gene and barcode sets. Write the resulting kBET and Silhouette values for each pipeline (e.g., UniverSC 0.06 kBET / 0.43 Silhouette vs zUMIs 0.11 kBET / 0.36 Silhouette) to a CSV such as integration_scores.csv, and check that the direction reproduces the expected result.

## Related tools

- **Seurat** (R package used to integrate each comparison GBM independently with the reference GBM prior to quality evaluation) — https://github.com/minoda-lab/universc
- **kBET** (R package (version 0.99.6) used to compute the kBET metric quantifying batch-effect removal in each integration)
- **cluster** (R package (version 2.1.3) used to compute Silhouette scores assessing cluster distinctness after integration)
- **UniverSC** (Produces one of the two SmartSeq2 GBMs being compared (uniform pipeline for all datasets)) — https://github.com/minoda-lab/universc
- **zUMIs** (Platform-specific pipeline producing the alternative SmartSeq2 GBM baseline for comparison)

## Examples

```
Rscript -e 'library(kBET); library(cluster); k <- kBET(integrated@meta.data$batch, integrated@reductions$PCA@cell.embeddings, n_repeat=100); sil <- silhouette(integrated@meta.data$seurat_clusters, dist(integrated@reductions$PCA@cell.embeddings)); write.csv(cbind(kBET=mean(k$abr), Silhouette=mean(sil[,3])), "integration_scores.csv")'
```

## Evaluation signals

- UniverSC integration should show lower kBET than zUMIs integration (~0.06 vs ~0.11), reproducing the direction of Fig. 3B
- UniverSC integration should show higher Silhouette than zUMIs integration (~0.43 vs ~0.36), reproducing Fig. 3C
- Both kBET and Silhouette are computed per integration independently, so each comparison GBM yields exactly one kBET and one Silhouette value
- Input GBMs must share identical gene sets; verify missing genes were added with 0 UMIs and only intersecting barcodes retained before scoring
- integration_scores.csv must contain kBET and Silhouette values for both the UniverSC and zUMIs integrations

## Limitations

- kSilhouette and kBET values are only comparable across integrations built on identically harmonised GBMs; differences in barcode/gene harmonisation confound the metrics
- Results were validated for 3' UMI-based platforms; 5' scRNA-seq technologies other than Chromium are not supported by UniverSC at the time of writing
- The reported scores are specific to the mouse primary cell benchmarking datasets mapped to GRCm39 without downsampling; other datasets may show different magnitudes

## Evidence

- [methods] each version of SmartSeq2 GBM was integrated with the reference GBMs independently using Seurat: "each version of SmartSeq2 GBM was integrated with the reference GBMs independently using Seurat"
- [methods] To evaluate the quality of integration, kBET and Silhouette score were calculated for each case: "To evaluate the quality of integration, kBET12 and Silhouette score13 were calculated for each case"
- [methods] The pair of GBMs was adjusted to have matching sets of barcodes and genes: "The pair of GBMs was adjusted to have matching sets of barcodes and genes: only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [methods] The full mouse genome (GRCm39) was used as the reference genome and no downsampling was performed for these datasets.: "The full mouse genome (GRCm39) was used as the reference genome and no downsampling was performed for these datasets."
- [abstract] UniverSC supports any unique molecular identifier-based single-cell RNA-seq platform: "a universal single-cell RNA-seq data processing tool that supports any unique molecular identifier-based platform"
