---
name: multiple-testing-benjamini-hochberg-correction
description: Use when you have computed thousands of per-gene statistics in parallel
  (e.g., a Pearson correlation p-value for every gene vs. a marker gene like VIM)
  and must select a FDR-controlled gene set before overlap or enrichment testing.
when_to_use_negative:
- You are testing a single pre-specified hypothesis (no family of thousands of parallel
  tests), so correction is unnecessary.
- The statistic is not a p-value (e.g., raw fold-changes or correlation coefficients
  without assigned p-values); compute p-values first.
- Your downstream test already internally controls family-wise error (e.g., an enrichment
  tool that estimates its own null), making a second correction layer inappropriate.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: python3
  role: Runtime used to script the p-value computation and Benjamini-Hochberg correction
    over the gene-wise correlation results
- name: Docker (sridnona/python3:180925.v1)
  role: Container image providing the pinned python3 environment in which the correction
    script is executed for reproducibility
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:3bb1c35a088103ae3d3a28b41f9fe2f98f882d84431b97caeda5e539dc6b8c2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - python3
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multiple-testing-benjamini-hochberg-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to per-gene statistics (e.g., Pearson correlation p-values against VIM across TCGA AML samples) computed in a python3 workflow, converting raw p-values into q-values so that only genes passing a strict significance threshold (q < 0.001) are carried forward as a gene set.

## When to use

Use when you have computed thousands of per-gene statistics in parallel (e.g., a Pearson correlation p-value for every gene vs. a marker gene like VIM) and must select a FDR-controlled gene set before overlap or enrichment testing.

## When NOT to use

- You are testing a single pre-specified hypothesis (no family of thousands of parallel tests), so correction is unnecessary.
- The statistic is not a p-value (e.g., raw fold-changes or correlation coefficients without assigned p-values); compute p-values first.
- Your downstream test already internally controls family-wise error (e.g., an enrichment tool that estimates its own null), making a second correction layer inappropriate.

## Inputs

- Vector of raw per-gene p-values (one per tested gene, e.g., from Pearson correlation of each gene with VIM across all TCGA AML samples)
- TCGA AML expression matrix used to derive the correlation p-values
- Significance threshold for the corrected values (q < 0.001 in the source workflow)

## Outputs

- Per-gene q-values (Benjamini-Hochberg adjusted p-values)
- FDR-controlled gene set: the 2191 genes with q < 0.001 constituting the VIM-correlated gene set
- Results table written for downstream analysis (e.g., vim_gata2_overlap_results.tsv includes the set and overlap p-value)

## How to apply

First compute a raw statistic per gene — in the source workflow, the Pearson correlation coefficient between every gene and VIM across all TCGA AML samples, with a p-value assigned to each correlation. Then apply Benjamini-Hochberg multiple-testing correction to the correlation p-values, ranking them and scaling by their rank over the total number of genes tested to obtain q-values. Filter the corrected results at q < 0.001, which in the source analysis yielded 2191 VIM-correlated genes from TCGA AML expression data. This FDR-controlled set is then used as input to downstream overlap testing — e.g., a scipy.stats hypergeometric test against GATA2R361C subclone-associated genes produced by task_006 — so the correction step directly controls the size and specificity of the gene set feeding the overlap p-value. Run the script in the paper's python3 environment (Docker image sridnona/python3:180925.v1) for reproducibility.

## Related tools

- **python3** (Runtime used to script the p-value computation and Benjamini-Hochberg correction over the gene-wise correlation results)
- **Docker (sridnona/python3:180925.v1)** (Container image providing the pinned python3 environment in which the correction script is executed for reproducibility)

## Evaluation signals

- Every input p-value maps to exactly one q-value; the q-value vector has the same length and gene ordering as the input p-value vector.
- All q-values are monotonically non-decreasing with the raw p-value ranking (BH adjustment preserves order).
- After filtering, the retained set contains exactly 2191 genes with q < 0.001, matching the expected VIM-correlated gene set size.
- The downstream hypergeometric overlap test against the GATA2R361C subclone gene list reproduces an intersection of ~198 genes with p = 3.5e-95.
- Rerunning the correction inside the sridnona/python3:180925.v1 Docker image reproduces the identical gene set (deterministic output).

## Limitations

- The q < 0.001 cutoff and the resulting 2191-gene set size are specific to this TCGA AML cohort and correlation analysis; other datasets will yield different set sizes.
- Correction controls only the false discovery rate of the correlation tests; it does not validate that the resulting gene set represents a coherent regulon — that requires the independent hypergeometric overlap test against subclone genes.
- BH correction assumes the p-values are valid; correlation p-values from expression data with dropout and limited sensitivity may be miscalibrated.

## Evidence

- [methods] Apply Benjamini-Hochberg multiple-testing correction to the correlation p-values and retain the 2191 genes with q < 0.001 as the VIM-correlated gene set.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the Pearson correlation coefficient between every gene and VIM across all TCGA AML samples, and assign a p-value to each correlation.: [evidence span withheld — anchor status: not_found]
- [methods] Docker image
----------
* sridnona/python3:180925.v1: "Docker image
----------
* sridnona/python3:180925.v1"
