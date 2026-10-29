---
name: classification-metrics-computation
description: Use when demultiplexing assignments from hashing methods (HTOreader,
  HTOdemux, MULTI-seq, GMM-Demux, DropletUtils, BFF) must be validated quantitatively
  against known per-cell sample labels.
when_to_use_negative:
- Samples lack distinct genetic backgrounds, so scSplit/SNP-based ground truth cannot
  be produced (e.g. animal studies or same-donor longitudinal samples)
- You only need to run demultiplexing itself rather than score its output against
  labels — use the HTOreader/HybridDemultiplexing workflow instead
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: HTOreader
  role: One of the demultiplexing methods evaluated; its HTOClassification (CLR) calls
    are scored against ground truth
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: HTOdemux
  role: Seurat-based baseline demultiplexer run at positive quantile 0.99 whose calls
    are scored
- name: MULTI-seq
  role: Baseline demultiplexer run with defaults whose calls are scored
- name: GMM_Demux
  role: Baseline demultiplexer run with defaults whose calls are scored
- name: DropletUtils
  role: Baseline demultiplexer run with defaults whose calls are scored
- name: BFF
  role: BFF_raw and BFF_cluster from cellhashR run with defaults; their calls are
    scored
- name: scSplit
  role: SNP-based method providing the ground-truth cell labels used to compute the
    metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:d7927ec416d73d79d9ef9d8ef3b7be7ae3f0c842dcab766c1cb862ba3ebeff34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BFF
  - DropletUtils
  - GMM_Demux
  - HTOdemux
  - HTOreader
  - MULTI-seq
  - cellhashR
  - scSplit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# classification-metrics-computation

## Summary

Computes singlet rate, accuracy, recall, precision and F1 for cell-hashing demultiplexing methods by comparing each method's per-cell hashtag assignments (singlet/doublet/negative and sample-id calls) against scSplit-derived ground-truth labels, as in the Stoeckius-2018 HTOreader benchmark.

## When to use

Use when demultiplexing assignments from hashing methods (HTOreader, HTOdemux, MULTI-seq, GMM-Demux, DropletUtils, BFF) must be validated quantitatively against known per-cell sample labels.

## When NOT to use

- Samples lack distinct genetic backgrounds, so scSplit/SNP-based ground truth cannot be produced (e.g. animal studies or same-donor longitudinal samples)
- You only need to run demultiplexing itself rather than score its output against labels — use the HTOreader/HybridDemultiplexing workflow instead

## Inputs

- Per-cell demultiplexing calls (singlet/doublet/negative plus sample-id) from each method, e.g. HTOreader HTOClassification output on a Seurat object's 'HTO' assay
- Ground-truth per-cell sample labels (scSplit-derived) for the benchmark dataset
- HTO count matrix (e.g. Stoeckius-2018 8-hashtag matrix from GEO accession GSE108313)

## Outputs

- benchmark_metrics.csv with singlet rate, accuracy, recall, precision and F1 per method
- Per-sample and per-class TP/FP/FN confusion counts for each method

## How to apply

Run each demultiplexing method on the same HTO count matrix (e.g. the Stoeckius-2018 8-hashtag matrix from GSE108313), using the method's recommended settings such as HTOreader's HTOClassification with CLR normalization, HTOdemux with positive quantile 0.99, and default settings for MULTI-seq, GMM-Demux, DropletUtils and BFF_raw/BFF_cluster from cellhashR. Retrieve scSplit-derived ground-truth cell labels and compare each method's singlet/doublet/negative classes and sample-id assignments to them, counting true positives, false positives and false negatives per sample and per class. From these counts compute singlet rate, accuracy, recall, precision and F1 for every method and write the results table (e.g. benchmark_metrics.csv). Accept the run only when all five metrics exist for all methods and each metric lies in the [0,1] range.

## Related tools

- **HTOreader** (One of the demultiplexing methods evaluated; its HTOClassification (CLR) calls are scored against ground truth) — github.com/WilsonImmunologyLab/HTOreader
- **HTOdemux** (Seurat-based baseline demultiplexer run at positive quantile 0.99 whose calls are scored)
- **MULTI-seq** (Baseline demultiplexer run with defaults whose calls are scored)
- **GMM_Demux** (Baseline demultiplexer run with defaults whose calls are scored)
- **DropletUtils** (Baseline demultiplexer run with defaults whose calls are scored)
- **BFF** (BFF_raw and BFF_cluster from cellhashR run with defaults; their calls are scored)
- **scSplit** (SNP-based method providing the ground-truth cell labels used to compute the metrics)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- benchmark_metrics.csv contains all five metrics (singlet rate, accuracy, recall, precision, F1) for all seven methods
- Every metric value lies in the [0,1] range
- TP+FN equals the ground-truth count per sample/class, and TP+FP equals the predicted count — confusion counts are internally consistent
- Singlet rates land in the empirically expected ~80-83% band for all methods on the Stoeckius-2018 benchmark
- PlotHTO visual checks confirm each HTO's cutoff quality before calls are scored

## Limitations

- SNP-based ground truth from scSplit only works with samples having a distinct genetic background, limiting which datasets can be validated this way
- Existing methods can fail on challenging datasets (poor hashtag staining, imbalanced sample sizes, many hashtags) and parameter tuning only marginally improves performance
- SNP-based clustering used as ground truth can itself be low-quality for cells with abnormal expression profiles (e.g. plasmablasts or dying cells), high true doublet rates, or poor transcriptome quality

## Evidence

- [intro] HTOdemux (Seurat), Multi-seq, GMM_Demux, BFF, and DropletUtils have been shown to work well with benchmark datasets: "HTOdemux (Seurat), Multi-seq, GMM_Demux, BFF, and DropletUtils have been shown to work well with benchmark datasets"
- [intro] Hybrid benchmark accuracy claim: "For both benchmark and real-world datasets, our hybrid strategy significantly increases accuracy and cell recovery rate when compared to existing demultiplexing approaches for cell hashing alone"
- [readme] HTOreader invocation: "pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')"
- [readme] Cutoff quality check: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [discussion] Failure-mode caveat: "while tuning parameters can marginally improve performance, it is not sufficient to overcome failures on challenging datasets"
