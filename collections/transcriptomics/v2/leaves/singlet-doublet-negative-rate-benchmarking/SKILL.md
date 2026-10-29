---
name: singlet-doublet-negative-rate-benchmarking
description: Use when a task requires comparing HTO demultiplexing tools (HTOdemux,
  MULTI-seq, GMM_Demux, BFF, DropletUtils, HTOreader) by singlet/doublet/negative
  rates on the same hashtag dataset, especially datasets with degraded hashtag staining.
when_to_use_negative:
- Input lacks an HTO assay entirely (e.g. SNP/genotype-only data from demuxlet, scSplit,
  Vireo, or Souporcell) — hashtag-based rate benchmarking does not apply
- Goal is a single-sample identity assignment rather than cross-method performance
  comparison of singlet/doublet/negative rates
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HTOreader
  role: Provides HTOClassification (cutoff calling and singlet/doublet/negative assignment)
    and PlotHTO (cutoff quality inspection) used as the reference method in the comparison
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: HTOdemux
  role: Seurat-based comparison method whose R125 singlet rate (~62.3%) is benchmarked;
    known to misassign cells to the negative group
- name: MULTI-seq
  role: Comparison method with the lowest R125 singlet rate (~49.7%) due to excessive
    negative-group assignment
- name: GMM_Demux
  role: GMM-based comparison method run on the same HTO matrix
- name: BFF
  role: Comparison method run in both BFF_raw and BFF_cluster modes (BFF_cluster reported
    ~94.5% singlet rate on R125)
- name: DropletUtils
  role: Comparison method producing per-cell hashtag identity assignments on the shared
    HTO matrix
- name: Seurat
  role: Loads the 'HTO' assay as a Seurat object and provides VlnPlot for visualizing
    HTO signal by assigned HTOid
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:f6f517eff0b85fbf7845ec7a889da8a1cb683832ff26f19fb3fd1c73834a9e50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BFF
  - DropletUtils
  - GMM_Demux
  - HTOClassification
  - HTOdemux
  - HTOreader
  - MULTI-seq
  - PlotHTO
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# singlet-doublet-negative-rate-benchmarking

## Summary

Benchmark cell-hashing demultiplexing methods by tabulating per-method singlet, doublet, and negative rates on a shared HTO assay, then comparing against published values (e.g. R125: MULTI-seq ~49.7%, HTOdemux ~62.3%, BFF_cluster ~94.5% singlet rates).

## When to use

Use when a task requires comparing HTO demultiplexing tools (HTOdemux, MULTI-seq, GMM_Demux, BFF, DropletUtils, HTOreader) by singlet/doublet/negative rates on the same hashtag dataset, especially datasets with degraded hashtag staining.

## When NOT to use

- Input lacks an HTO assay entirely (e.g. SNP/genotype-only data from demuxlet, scSplit, Vireo, or Souporcell) — hashtag-based rate benchmarking does not apply
- Goal is a single-sample identity assignment rather than cross-method performance comparison of singlet/doublet/negative rates

## Inputs

- Seurat object with an 'HTO' assay (e.g. R125 3-hashtag dataset from Mendeley Data DOI 10.17632/3jdywv5jrv.3)
- HTO count/normalized matrix shared by all compared methods
- HTOreader package installed via devtools (with flexmix and hash dependencies)

## Outputs

- Per-method singlet, doublet, and negative counts and rates across pooled cells
- Summary comparison table (e.g. r125_demultiplexing_comparison.csv)
- QC figures: VlnPlot of HTO signal grouped by HTOid, per-hashtag cutoff plots combined with plot_grid

## How to apply

Download the pooled hashtag dataset (e.g. R125 3-hashtag from Mendeley Data, DOI 10.17632/3jdywv5jrv.3) and load its HTO assay as a Seurat object with the Seurat package. Run HTOreader's HTOClassification on the 'HTO' assay (method 'CLR' or 'log', automatic cutoff calling, optionally verifying cutoffs with PlotHTO and plot_grid), then run each comparison method — HTOdemux (Seurat), MULTI-seq, GMM_Demux, DropletUtils, BFF_raw, BFF_cluster — on the same HTO matrix to get per-cell hashtag identity assignments. Tabulate per-method counts and rates of singlets, doublets, and negatives across the pooled cells; a spike in negative assignments (incorrect assignment of many cells into the negative group) is the diagnostic signal explaining low singlet rates on poorly stained datasets like R125. Render QC figures (VlnPlot of HTO signal grouped by assigned HTOid, per-hashtag cutoff plots combined via plot_grid) and write the summary table to a CSV such as r125_demultiplexing_comparison.csv.

## Related tools

- **HTOreader** (Provides HTOClassification (cutoff calling and singlet/doublet/negative assignment) and PlotHTO (cutoff quality inspection) used as the reference method in the comparison) — github.com/WilsonImmunologyLab/HTOreader
- **HTOdemux** (Seurat-based comparison method whose R125 singlet rate (~62.3%) is benchmarked; known to misassign cells to the negative group)
- **MULTI-seq** (Comparison method with the lowest R125 singlet rate (~49.7%) due to excessive negative-group assignment)
- **GMM_Demux** (GMM-based comparison method run on the same HTO matrix)
- **BFF** (Comparison method run in both BFF_raw and BFF_cluster modes (BFF_cluster reported ~94.5% singlet rate on R125))
- **DropletUtils** (Comparison method producing per-cell hashtag identity assignments on the shared HTO matrix)
- **Seurat** (Loads the 'HTO' assay as a Seurat object and provides VlnPlot for visualizing HTO signal by assigned HTOid)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plots1 <- PlotHTO(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- Per-method singlet+doublet+negative counts sum to the total number of pooled cells for every method
- Computed singlet rates fall near reported R125 values: ~49.7% (MULTI-seq), ~62.3% (HTOdemux), ~94.5% (BFF_cluster)
- Low-singlet-rate methods show a correspondingly high negative-group assignment rate, confirming the 'incorrect assignment into the negative group' mechanism
- PlotHTO cutoff plots show each hashtag's cutoff placed between its positive and negative signal distributions
- Summary table written to r125_demultiplexing_comparison.csv contains a row per method with rates (not just counts)

## Limitations

- Existing cell hashing demultiplexing methods can fail on challenging real-world datasets, and parameter tuning only marginally improves performance
- Poor staining quality of certain hashtags can decrease cell recovery by increasing the negative rate (e.g. the degraded R125 hashtags)
- Cell hashing suffers from unpredictable hashtag failure, high false doublet rates when using many hashtags, and difficulty distinguishing true positive from background signal

## Evidence

- [intro] HTOdemux (Seurat), Multi-seq, GMM_Demux, BFF, and DropletUtils have been shown to work well with benchmark datasets: "HTOdemux (Seurat), Multi-seq, GMM_Demux, BFF, and DropletUtils have been shown to work well with benchmark datasets"
- [readme] Please load your "HTO" assay using the Seurat package: "Please load your "HTO" assay using the Seurat package (More details in our example)."
- [discussion] the poor staining quality of certain hashtags that occur sporadically can further decrease cell recovery by increasing the negative rate: "the poor staining quality of certain hashtags that occur sporadically can further decrease cell recovery by increasing the negative rate"
- [discussion] while tuning parameters can marginally improve performance, it is not sufficient to overcome failures on challenging datasets: "while tuning parameters can marginally improve performance, it is not sufficient to overcome failures on challenging datasets"
- [readme] Users can use PlotHTO function to check the quality of the cutoff of each HTO: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
