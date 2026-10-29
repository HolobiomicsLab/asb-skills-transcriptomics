---
name: copy-number-segmentation-runtime-benchmarking
description: Use when reproducing or extending runtime/efficiency benchmarks of scRNA-seq
  copy-number (CNV) tools such as SCEVAN, CopyKAT, and inferCNV, or when justifying
  tool choice on large droplet datasets by measuring per-stage wall-clock runtimes.
when_to_use_negative:
- The analysis goal is CNV accuracy (e.g., F1 or segment concordance) rather than
  computational efficiency — use accuracy benchmarking instead
- The tumor type has minimal genomic alterations (leukemia, pediatric cancers, Ependymomas),
  where aneuploidy-based tools including the compared methods are unsuited regardless
  of runtime
- The comparison targets bulk or shallow WGS data only, where single-cell CNV pipelines
  and their classification stages do not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2648
tools:
- name: SCEVAN
  role: Tool under test; pipelineCNA (par_cores = 20) run per dataset with classifyTumorCells
    and getBreaksVegaMC timed separately
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Baseline for the classification-stage runtime comparison; notably slow on
    the MM data due to large cell counts
- name: inferCNV v1.4.0
  role: Baseline for the classification-stage runtime comparison
- name: DNAcopy v1.62.0
  role: Reference segmenter used for GBM sample segmentation timing
- name: CNVkit v0.9.9
  role: Reference segmenter used for Multiple Myeloma segmentation timing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:7bac4478ff4e65d1231a9199b7063dab36c285b9f4c0b5a6ed98568f4d495712
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CNVkit v0.9.9
  - CopyKAT v1.0.5
  - DNAcopy
  - DNAcopy v1.62.0
  - SCEVAN
  - inferCNV v1.4.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# copy-number-segmentation-runtime-benchmarking

## Summary

A benchmarking skill for reproducing per-stage (malignant/non-malignant classification and joint segmentation) runtime comparisons of SCEVAN against CopyKAT v1.0.5, inferCNV v1.4.0, DNAcopy v1.62.0, and CNVkit v0.9.9 on scRNA-seq tumor datasets. It quantifies how SCEVAN's greedy multichannel Mumford–Shah segmentation achieves speedups (e.g., 11× vs inferCNV and 19× vs CopyKAT on the Multiple Myeloma 10X data).

## When to use

Use when reproducing or extending runtime/efficiency benchmarks of scRNA-seq copy-number (CNV) tools such as SCEVAN, CopyKAT, and inferCNV, or when justifying tool choice on large droplet datasets by measuring per-stage wall-clock runtimes.

## When NOT to use

- The analysis goal is CNV accuracy (e.g., F1 or segment concordance) rather than computational efficiency — use accuracy benchmarking instead
- The tumor type has minimal genomic alterations (leukemia, pediatric cancers, Ependymomas), where aneuploidy-based tools including the compared methods are unsuited regardless of runtime
- The comparison targets bulk or shallow WGS data only, where single-cell CNV pipelines and their classification stages do not apply

## Inputs

- Raw scRNA-seq count matrices (genes × cells) for GSE131928 GBM samples MGH102, MGH104, MGH105, MGH106
- Multiple Myeloma 10X Genomics count matrix
- Reference segmenter configurations: DNAcopy v1.62.0 (GBM), CNVkit v0.9.9 (Multiple Myeloma)

## Outputs

- runtime_benchmark.csv with per-tool, per-stage (classification vs segmentation) runtimes
- Per-dataset, per-stage speedup factors for SCEVAN vs CopyKAT, inferCNV, DNAcopy, and CNVkit

## How to apply

Load the raw scRNA-seq count matrices — the GBM multiregional samples MGH102, MGH104, MGH105, MGH106 from GSE131928 and the Multiple Myeloma 10X count matrix — into R. Run SCEVAN::pipelineCNA with par_cores = 20, instrumenting classifyTumorCells (classification) and getBreaksVegaMC (joint segmentation) with independent timers. Time the classification stages of CopyKAT v1.0.5 and inferCNV v1.4.0 identically, and time segmentation with the reference segmenters used in the paper: DNAcopy v1.62.0 for GBM and CNVkit v0.9.9 for Multiple Myeloma. Compute per-dataset, per-stage speedup factors and assemble them with per-tool runtimes into runtime_benchmark.csv. The rationale is that SCEVAN's greedy multichannel Mumford–Shah segmentation and automatic classification avoid the manual reference-cell identification that makes inferCNV-like methods slow, while CopyKAT is particularly slow on the MM data because of the large number of cells.

## Related tools

- **SCEVAN** (Tool under test; pipelineCNA (par_cores = 20) run per dataset with classifyTumorCells and getBreaksVegaMC timed separately) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline for the classification-stage runtime comparison; notably slow on the MM data due to large cell counts)
- **inferCNV v1.4.0** (Baseline for the classification-stage runtime comparison)
- **DNAcopy v1.62.0** (Reference segmenter used for GBM sample segmentation timing)
- **CNVkit v0.9.9** (Reference segmenter used for Multiple Myeloma segmentation timing)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Reproduced classification speedup of SCEVAN vs CopyKAT falls in the reported 2–7× range
- Reproduced segmentation speedups fall in the reported ranges: 2×/5× on GBM (vs DNAcopy) and 19×/11× on Multiple Myeloma (vs CNVkit)
- runtime_benchmark.csv contains a runtime entry for every tool–stage–dataset combination and matching speedup factors
- Timing methodology is identical across tools (same count matrices, same instrumentation of classification vs segmentation stages)
- SCEVAN runs use par_cores = 20 as in the paper's workflow

## Limitations

- The runtime advantage is specific to aneuploidy-detecting pipelines; the approach may not be suited to tumors with minimal genomic alterations (e.g., leukemia, pediatric cancers, Ependymomas)
- Ground truth from WES/WGS bulk profiling has different resolution than single-cell data and requires resampling before accuracy comparison (runtime figures alone do not establish accuracy)
- SCEVAN's own version and the segmentation parameter β value are not stated in the article text; the β constant is the only segmentation parameter (default beta_vega = 0.5 per README)

## Evidence

- [discussion] CopyKAT and inferCNV versions used in benchmark: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
- [discussion] Reference segmenter for Multiple Myeloma: "In the case of Multiple Myeloma17, CNVkit v0.9.9 was used for segmentation."
- [discussion] Reference segmenter for GBM samples: "was segmented using DNAcopy (v1.62.0)47 (Supplementary Data 5)."
- [readme] Speed motivation from README: [evidence span withheld — anchor status: external_doc]
- [methods] Pipeline invocation used for timing: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
