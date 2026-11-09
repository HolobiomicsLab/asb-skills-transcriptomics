---
name: demultiplexing-benchmark-evaluation
description: Use when a cell-hashing HTO count matrix (e.g. 8-hashtag Stoeckius-2018,
  GSE108313) must be demultiplexed by multiple tools and their sample assignments
  compared to scSplit-derived ground-truth labels.
when_to_use_negative:
- The samples lack a distinct genetic background (e.g. most animal studies or same-donor
  longitudinal studies), so no scSplit ground-truth labels can be derived to validate
  against
- The goal is production demultiplexing of a single dataset with the hybrid hashing+SNP
  strategy rather than comparative method evaluation
- The input is not an HTO hashtag count assay (e.g. a gene expression matrix or an
  already-demultiplexed object)
tools:
- name: HTOreader
  role: Primary method under benchmark; HTOClassification with CLR normalization yields
    per-cell hashtag assignments
  repo: https://github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Provides the Seurat object/HTO assay container; HTOdemux is run from Seurat
    with positive quantile 0.99
- name: MULTI-seq
  role: Baseline demultiplexing method run with default settings on the same count
    matrix
- name: GMM_Demux
  role: Baseline demultiplexing method run with default settings on the same count
    matrix
- name: DropletUtils
  role: Baseline demultiplexing method run with default settings on the same count
    matrix
- name: cellhashR
  role: Supplies BFF_raw and BFF_cluster, run with default parameters
- name: scSplit
  role: SNP-based method whose genotype-derived labels serve as the benchmark ground
    truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:381df6a36d790f32758e51071398e9914ea25f7bc8a1c5a75b4227883aa82e68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/HTOreader
  tools:
  - BFF
  - DropletUtils
  - GMM_Demux
  - HTOdemux
  - HTOreader
  - MULTI-seq
  - Seurat
  - cellhashR
  - scSplit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# demultiplexing-benchmark-evaluation

## Summary

Benchmark cell hashing demultiplexing methods (HTOreader, HTOdemux, MULTI-seq, GMM-Demux, DropletUtils, BFF_raw/BFF_cluster) on the Stoeckius-2018 8-hashtag dataset (GSE108313), scoring singlet/doublet/negative assignments against scSplit-derived ground-truth labels using singlet rate, accuracy, recall, precision and F1.

## When to use

Use when a cell-hashing HTO count matrix (e.g. 8-hashtag Stoeckius-2018, GSE108313) must be demultiplexed by multiple tools and their sample assignments compared to scSplit-derived ground-truth labels.

## When NOT to use

- The samples lack a distinct genetic background (e.g. most animal studies or same-donor longitudinal studies), so no scSplit ground-truth labels can be derived to validate against
- The goal is production demultiplexing of a single dataset with the hybrid hashing+SNP strategy rather than comparative method evaluation
- The input is not an HTO hashtag count assay (e.g. a gene expression matrix or an already-demultiplexed object)

## Inputs

- Stoeckius-2018 8-hashtag HTO count matrix (GEO accession GSE108313)
- scSplit-derived ground-truth cell labels per barcode
- Seurat object with an 'HTO' assay

## Outputs

- Per-cell hashtag assignments (singlet / doublet / negative plus sample id) for each of the seven methods
- benchmark_metrics.csv with singlet rate, accuracy, recall, precision and F1 per method

## How to apply

Download the Stoeckius-2018 8-hashtag HTO count matrix from GEO accession GSE108313 and retrieve the scSplit-derived ground-truth cell labels. Load the HTO assay as a Seurat object and run HTOreader's HTOClassification with CLR normalization under default parameters to obtain per-cell hashtag assignments, optionally inspecting per-HTO cutoffs with PlotHTO. Run HTOdemux from Seurat with positive quantile 0.99, plus MULTI-seq, GMM-Demux and DropletUtils at default settings on the same count matrix, and run BFF_raw and BFF_cluster from the cellhashR package with defaults. Compare each method's singlet/doublet/negative and sample-id assignments to the scSplit ground truth, counting true positives, false positives and false negatives per sample and per class. Compute singlet rate, accuracy, recall, precision and F1 for every method and write the results to benchmark_metrics.csv; the run is accepted when all five metrics for all seven methods exist and each lies in [0,1].

## Related tools

- **HTOreader** (Primary method under benchmark; HTOClassification with CLR normalization yields per-cell hashtag assignments) — https://github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Provides the Seurat object/HTO assay container; HTOdemux is run from Seurat with positive quantile 0.99)
- **MULTI-seq** (Baseline demultiplexing method run with default settings on the same count matrix)
- **GMM_Demux** (Baseline demultiplexing method run with default settings on the same count matrix)
- **DropletUtils** (Baseline demultiplexing method run with default settings on the same count matrix)
- **cellhashR** (Supplies BFF_raw and BFF_cluster, run with default parameters)
- **scSplit** (SNP-based method whose genotype-derived labels serve as the benchmark ground truth)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- benchmark_metrics.csv contains all five metrics (singlet rate, accuracy, recall, precision, F1) for all seven methods
- Every reported metric value lies in the [0,1] range
- All methods on the Stoeckius-2018 benchmark are expected to show comparable singlet rates of roughly 80-83%
- Each method produces a per-cell classification covering singlet, doublet and negative classes comparable to the scSplit labels
- PlotHTO cutoff plots confirm each HTO's cutoff is sensible before trusting HTOClassification assignments

## Limitations

- SNP-based ground truth (scSplit) only works with samples that have a distinct genetic background, so it is not applicable in most animal studies or same-donor longitudinal studies
- SNP-based clustering can produce low-quality labels due to abnormal gene expression profiles (e.g. plasmablasts or dying cells), high true doublet rates, and poor transcriptome quality
- Parameter tuning of existing cell hashing methods only marginally improves performance and cannot overcome failures on challenging datasets, such as those with poor hashtag staining quality
- Cell hashing has difficulties distinguishing true positive from background signal, unpredictable hashtag failure, and high false doublet rates when using many hashtags

## Evidence

- [intro] HTOdemux (Seurat), Multi-seq, GMM_Demux, BFF, and DropletUtils have been shown to work well with benchmark datasets: "HTOdemux (Seurat), Multi-seq, GMM_Demux, BFF, and DropletUtils have been shown to work well with benchmark datasets"
- [abstract] HTOreader developed for improved cutoff calling: "we developed HTOreader, a demultiplexing tool for cell hashing that improves the accuracy of cut-off calling"
- [readme] README quick start for HTOClassification: "There are two main functions, `HTOClassification` for Hashtag demultiplexing, `PlotHTO` for plotting distributions of hashtags"
- [intro] Benchmark vs ground truth shows consistent gains: "For both benchmark and real-world datasets, our hybrid strategy significantly increases accuracy and cell recovery rate when compared to existing demultiplexing approaches for cell hashing alone"
