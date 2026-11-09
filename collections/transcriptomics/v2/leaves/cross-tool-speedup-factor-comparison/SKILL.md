---
name: cross-tool-speedup-factor-comparison
description: Use when reproducing or extending runtime/speedup benchmarks of SCEVAN
  against CopyKAT, inferCNV, DNAcopy, or CNVkit on scRNA-seq count matrices, timing
  classification and segmentation stages separately.
when_to_use_negative:
- The goal is accuracy (F1 score, CNA concordance with bulk WGS/WGS ground truth)
  rather than runtime — this skill measures speedup only
- Input datasets differ in cell count or technology between tools being compared,
  making wall-clock speedup factors non-comparable
- The tumor type has minimal genomic aneuploidy (leukemia, pediatric cancers, Ependymomas),
  where the compared aneuploidy-based methods may not be appropriate at all
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3673
tools:
- name: SCEVAN
  role: Tool being benchmarked; its classifyTumorCells and getBreaksVegaMC stages
    are timed individually via pipelineCNA
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Comparison method for the classification stage; particularly slow on the large
    Multiple Myeloma dataset
- name: inferCNV v1.4.0
  role: Comparison method for the classification stage on the same count matrices
- name: DNAcopy v1.62.0
  role: Reference segmentation baseline for the GBM samples
- name: CNVkit v0.9.9
  role: Reference segmentation baseline for the Multiple Myeloma 10X data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:3ae47d06271d56ce8a8910e23655d2cd9f8e9ad4d360e117520598dc10936d11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AntonioDeFalco/SCEVAN
  tools:
  - CNVkit v0.9.9
  - CopyKAT v1.0.5
  - DNAcopy
  - DNAcopy v1.62.0
  - SCEVAN
  - inferCNV v1.4.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-tool-speedup-factor-comparison

## Summary

A benchmarking skill for measuring per-stage (classification vs segmentation) runtimes of SCEVAN against CopyKAT, inferCNV, DNAcopy, and CNVkit on the same scRNA-seq count matrices, and computing cross-tool speedup factors per dataset.

## When to use

Use when reproducing or extending runtime/speedup benchmarks of SCEVAN against CopyKAT, inferCNV, DNAcopy, or CNVkit on scRNA-seq count matrices, timing classification and segmentation stages separately.

## When NOT to use

- The goal is accuracy (F1 score, CNA concordance with bulk WGS/WGS ground truth) rather than runtime — this skill measures speedup only
- Input datasets differ in cell count or technology between tools being compared, making wall-clock speedup factors non-comparable
- The tumor type has minimal genomic aneuploidy (leukemia, pediatric cancers, Ependymomas), where the compared aneuploidy-based methods may not be appropriate at all

## Inputs

- Raw scRNA-seq count matrices with genes on rows and cells on columns (Gene Symbol or Ensembl ID), e.g. MGH102/MGH104/MGH105/MGH106 from GSE131928 and the Multiple Myeloma 10X Genomics count matrix
- Tool versions to benchmark: SCEVAN, CopyKAT v1.0.5, inferCNV v1.4.0, DNAcopy v1.62.0, CNVkit v0.9.9
- Compute configuration: number of cores (par_cores = 20 in the reference benchmark)

## Outputs

- Per-tool, per-stage runtime measurements (classification and segmentation separately) for each dataset
- SCEVAN speedup factors per dataset and stage (e.g. 19×/11× on Multiple Myeloma; 2×/5× segmentation and 2–7× classification ranges on GBM)
- runtime_benchmark.csv assembling runtimes and speedup factors

## How to apply

Load the scRNA-seq raw count matrices (e.g., the GBM multiregional samples MGH102, MGH104, MGH105, MGH106 from GSE131928 and the Multiple Myeloma 10X count matrix) into R. Run SCEVAN's pipelineCNA with par_cores = 20, instrumenting classifyTumorCells (malignant/non-malignant classification) and getBreaksVegaMC (joint segmentation) separately with timers. Run the classification stage of CopyKAT v1.0.5 and inferCNV v1.4.0 on the same matrices, then run segmentation with the reference segmenters used in the benchmark: DNAcopy v1.62.0 for GBM and CNVkit v0.9.9 for Multiple Myeloma. Compute speedup factors per dataset and stage (classification vs CopyKAT; segmentation vs DNAcopy on GBM, vs CNVkit on MM) and assemble per-tool, per-stage runtimes into a runtime_benchmark.csv. On the Multiple Myeloma data, CopyKAT is particularly slow due to the large number of cells, which is why the speedup there (19× classification, 11× segmentation) exceeds the GBM ranges (2–7× classification, 2×/5× segmentation).

## Related tools

- **SCEVAN** (Tool being benchmarked; its classifyTumorCells and getBreaksVegaMC stages are timed individually via pipelineCNA) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Comparison method for the classification stage; particularly slow on the large Multiple Myeloma dataset)
- **inferCNV v1.4.0** (Comparison method for the classification stage on the same count matrices)
- **DNAcopy v1.62.0** (Reference segmentation baseline for the GBM samples)
- **CNVkit v0.9.9** (Reference segmentation baseline for the Multiple Myeloma 10X data)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Reproduced classification speedup vs CopyKAT falls in the reported 2–7× range
- Reproduced segmentation speedup on GBM (vs DNAcopy) falls around 2×/5×, and on Multiple Myeloma (vs CNVkit) around 11×–19×
- Per-tool per-stage runtimes in runtime_benchmark.csv are measured identically (same par_cores, same stage boundaries) across all tools
- CopyKAT runtime on the Multiple Myeloma dataset is markedly larger than on smaller datasets, consistent with slowness driven by the large number of cells

## Limitations

- Speedup factors are dataset- and hardware-dependent; they are valid ranges only for the specific datasets (GSE131928 GBM samples, Multiple Myeloma 10X) and par_cores = 20 used in the benchmark
- The β segmentation parameter value and mixture-model initial parameters are not fully specified in the text (deferred to Supplementary Table 1), affecting exact reproducibility of stage timings
- Aneuploidy-based tools benchmarked here may not be suited to tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)

## Evidence

- [results] SCEVAN segmentation speedup on MM: [evidence span withheld — anchor status: not_found]
- [readme] SCEVAN speed claim: [evidence span withheld — anchor status: external_doc]
- [discussion] Comparison tool versions: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
- [discussion] Segmentation baselines per dataset: "In the case of Multiple Myeloma17, CNVkit v0.9.9 was used for segmentation."
- [methods] Reference invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
