---
name: annotation-tool-parameter-configuration
description: Use when reproducing the scRNA-seq annotation benchmark of Accordion
  vs marker-based tools, or when replicating its exact tool settings (e.g. max_n_marker
  = 30, SCINA 100-marker cap, 4 threads).
when_to_use_negative:
- The goal is comparison with non-marker-based annotation tools — the article explicitly
  states no direct comparison was conducted with tools that do not rely on marker
  genes
- The input dataset lacks reliable ground-truth cell-type labels, which the benchmark
  scoring via ontology similarity requires
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell Marker Accordion
  role: Primary annotation tool run with max_n_marker = 30 and dataset tissue specified;
    also supplies positive markers to the baseline tools
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ScType
  role: Marker-based baseline run with its in-built marker database and default parameters
- name: SCINA
  role: Marker-based baseline given Accordion positive markers, capped at 100 marker
    genes per cell type
- name: clustifyR
  role: Marker-based baseline given Accordion positive markers as input
- name: scCATCH
  role: Marker-based baseline run with its in-built marker database and default parameters
- name: scSorter
  role: Marker-based baseline given Accordion positive markers; excluded ('Timeout')
    if runtime exceeds 3 hours
- name: Seurat
  role: 'Standard preprocessing pipeline: log-normalization, variable-feature selection,
    PCA, Louvain clustering'
- name: ontologySimilarity
  role: get_sim_grid function (v2.7) computes percentage similarity between predicted
    and ground-truth cell types over the Cell Ontology tree
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:9a2465de277bdf55bfec8fc11ec3749ee302436ee5d1ae31aae04c279d550637
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - SCINA
  - ScType
  - Seurat
  - clustifyR
  - ontologySimilarity
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# annotation-tool-parameter-configuration

## Summary

Configure the Cell Marker Accordion and its marker-based competitor annotation tools (ScType, SCINA, clustifyR, scCATCH, scSorter) with the article's specific parameter settings so benchmark comparisons are reproducible and fair.

## When to use

Use when reproducing the scRNA-seq annotation benchmark of Accordion vs marker-based tools, or when replicating its exact tool settings (e.g. max_n_marker = 30, SCINA 100-marker cap, 4 threads).

## When NOT to use

- The goal is comparison with non-marker-based annotation tools — the article explicitly states no direct comparison was conducted with tools that do not rely on marker genes
- The input dataset lacks reliable ground-truth cell-type labels, which the benchmark scoring via ontology similarity requires

## Inputs

- Preprocessed Seurat objects or raw/normalized count matrices from benchmark single-cell datasets (PBMC, bone marrow CITE-seq/AbSeq, E-HCAD-4, pancreas, liver, lung, kidney)
- Ground-truth cell-type annotations mapped to the Cell Ontology
- Cell Marker Accordion built-in marker database (accordion_marker / disease_accordion_marker)

## Outputs

- Per-cluster predicted cell-type annotations from each of the six tools
- Per-tool, per-dataset table of percentage similarity to ground truth and runtime
- Average improvement (~23%) of Cell Marker Accordion over marker-based baselines

## How to apply

Preprocess each benchmark single-cell dataset (PBMC FACS-sorted, CITE-seq/AbSeq bone marrow, E-HCAD-4, pancreas, liver, lung, kidney) with the standard Seurat pipeline (log-normalization, variable-feature selection, PCA, Louvain clustering). Run the Cell Marker Accordion with max_n_marker set to 30 and the dataset's tissue specified, using the accordion() function on a Seurat object or count matrix. Run ScType and scCATCH with their in-built marker databases at default parameters, while SCINA, clustifyR, and scSorter receive positive markers extracted from the Cell Marker Accordion database, capping SCINA at 100 marker genes per cell type. Exclude scSorter if it exceeds a 3-hour runtime ('Timeout') and record running times with all tools configured to use 4 threads on the same machine.

## Related tools

- **Cell Marker Accordion** (Primary annotation tool run with max_n_marker = 30 and dataset tissue specified; also supplies positive markers to the baseline tools) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ScType** (Marker-based baseline run with its in-built marker database and default parameters)
- **SCINA** (Marker-based baseline given Accordion positive markers, capped at 100 marker genes per cell type)
- **clustifyR** (Marker-based baseline given Accordion positive markers as input)
- **scCATCH** (Marker-based baseline run with its in-built marker database and default parameters)
- **scSorter** (Marker-based baseline given Accordion positive markers; excluded ('Timeout') if runtime exceeds 3 hours)
- **Seurat** (Standard preprocessing pipeline: log-normalization, variable-feature selection, PCA, Louvain clustering)
- **ontologySimilarity** (get_sim_grid function (v2.7) computes percentage similarity between predicted and ground-truth cell types over the Cell Ontology tree)

## Examples

```
data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30, include_detailed_annotation_info=TRUE, plot=TRUE)
```

## Evaluation signals

- Cell Marker Accordion achieves the highest annotation performance on healthy-tissue datasets, consistent with the reported average increase of 23% over other marker-based tools
- Every cell of the per-tool, per-dataset table carries a finite similarity score on the 0-100 scale from get_sim_grid (ontologySimilarity v2.7)
- SCINA runs never receive more than 100 positive marker genes per cell type
- Any scSorter run exceeding 3 hours is recorded as 'Timeout' rather than a partial score
- All tools are benchmarked with 4 threads on the same machine so runtimes are comparable

## Limitations

- No direct comparison was conducted with annotation tools that do not rely on marker genes, so conclusions apply only to marker-based methods
- scSorter may exceed the 3-hour limit and be excluded as 'Timeout' on large datasets
- Limited availability of high-quality ground-truth datasets constrains benchmark expansion

## Evidence

- [methods] Accordion run with max_n_marker=30: [evidence span withheld — anchor status: not_found]
- [methods] Baseline tool configuration: [evidence span withheld — anchor status: not_found]
- [methods] Similarity scoring: [evidence span withheld — anchor status: not_found]
- [methods] Runtime configuration: [evidence span withheld — anchor status: not_found]
- [discussion] Reported benchmark gain: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [readme] Example accordion() invocation parameters: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
