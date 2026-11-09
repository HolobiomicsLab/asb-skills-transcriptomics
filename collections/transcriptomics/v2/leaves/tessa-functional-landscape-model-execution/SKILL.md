---
name: tessa-functional-landscape-model-execution
description: Use when benchmarking antigen specificity prediction or clustering against
  a clonotype-level TCR-only baseline, or when a Bayesian TCR functional landscape
  estimate supervised by scRNA-Seq is required on preprocessed T cell data.
when_to_use_negative:
- The goal is a cell-level multimodal embedding fusing transcriptome and TCR — tessa
  uses a clonotype-level approach that fuses cells with identical TCRs and loses within-clonotype
  phenotype information
- No antigen specificity annotation or reference atlas is available to supervise the
  model
- The task requires only TCR sequence analysis without scRNA-Seq supervision
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: tessa
  role: Bayesian TCR functional landscape estimation model supervised with scRNA-Seq;
    the baseline method executed for antigen specificity prediction and clustering
- name: mvTCR
  role: Competing multi-view deep learning model retrained with CDR3β-only TCR input
    to make its representation comparable to tessa in the benchmark
  repo: github.com/SchubertLab/mvTCR_reproducibility
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:c0f08286b5aee0d11ed8795084538b886cc424c843a7f19d40a9d36d1b4898ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - mvTCR
  - tessa
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tessa-functional-landscape-model-execution

## Summary

Executes tessa (TCR functional landscape estimation), Zhang et al.'s Bayesian model supervised with scRNA-Seq, to predict and cluster antigen specificity from TCR sequences, serving as the baseline against mvTCR in benchmark comparisons.

## When to use

Use when benchmarking antigen specificity prediction or clustering against a clonotype-level TCR-only baseline, or when a Bayesian TCR functional landscape estimate supervised by scRNA-Seq is required on preprocessed T cell data.

## When NOT to use

- The goal is a cell-level multimodal embedding fusing transcriptome and TCR — tessa uses a clonotype-level approach that fuses cells with identical TCRs and loses within-clonotype phenotype information
- No antigen specificity annotation or reference atlas is available to supervise the model
- The task requires only TCR sequence analysis without scRNA-Seq supervision

## Inputs

- Preprocessed single-cell T cell datasets (10x >60k cells; Minervina) with cell type, clonotype, and antigen specificity annotations
- Query dataset(s) to be mapped onto a reference atlas for label prediction
- CDR3β TCR sequences and matched scRNA-Seq profiles

## Outputs

- Antigen specificity predictions for query cells (atlas-query mapping)
- Clustering assignments of T cells by functional/specificity landscape
- Per-dataset F1-Score and NMI metrics, aggregated gains and paired-test p-values (benchmark_comparison.csv)

## How to apply

Run tessa on the identically preprocessed 10x (>60k cells) and Minervina T cell datasets used for the competing method, so the comparison is not confounded by preprocessing differences. To keep the TCR input comparable, restrict the competing model's TCR representation to the CDR3β sequence only. Perform atlas-query prediction: map query data to the reference atlas and predict query cell labels with both methods. Compute F1-Score (prediction) and NMI (clustering) per dataset, aggregate average gains, and test significance with paired tests (in the mvTCR benchmark this yielded ~23.6% F1 gain, p=4.7e-6, and ~27.7% NMI gain, p=4.6e-6). Write per-dataset scores, gains, and p-values to a comparison table (e.g. benchmark_comparison.csv).

## Related tools

- **tessa** (Bayesian TCR functional landscape estimation model supervised with scRNA-Seq; the baseline method executed for antigen specificity prediction and clustering)
- **mvTCR** (Competing multi-view deep learning model retrained with CDR3β-only TCR input to make its representation comparable to tessa in the benchmark) — github.com/SchubertLab/mvTCR_reproducibility

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3  # matching benchmark environment; run tessa on the same preprocessed datasets, then write per-dataset F1/NMI to benchmark_comparison.csv
```

## Evaluation signals

- Per-dataset F1-Score for atlas-query specificity prediction is computed and reported for both tessa and the competing method
- Per-dataset NMI for clustering is computed and compared against the competing method
- Average gains over tessa reproduce the expected magnitude (~23.6% F1, p=4.7e-6; ~27.7% NMI, p=4.6e-6 in the mvTCR benchmark)
- Paired significance tests across datasets yield p-values reported alongside the gains
- All metrics, gains, and p-values are consolidated into benchmark_comparison.csv

## Limitations

- tessa uses a clonotype-level approach, fusing cells with identical TCRs, and therefore loses within-clonotype phenotype information
- In the mvTCR benchmark tessa was outperformed on atlas-query mapping across all datasets, so it may underperform as a specificity predictor
- Comparison fairness depends on restricting the competing model's TCR input to CDR3β only; otherwise inputs are not comparable

## Evidence

- [intro] tessa description: "Zhang et al. developed a Bayesian model called TCR functional landscape estimation supervised with scRNA-Seq analysis (tessa) to correlate both modalities"
- [intro] clonotype-level limitation: "they use a clonotype-level approach, fusing cells with identical TCRs. However, cells from the same clonotype can have distinct phenotypes"
- [discussion] benchmark result: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction"
- [readme] reproducibility setup: [evidence span withheld — anchor status: external_doc]
- [readme] evaluation workflow: [evidence span withheld — anchor status: external_doc]
