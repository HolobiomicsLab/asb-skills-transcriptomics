---
name: coverage-based-missingness-handling
description: Use when a scRNA-seq mutation (VAF) matrix paired with a per-cell sequencing-coverage
  matrix must be marked and filtered for low-coverage/unobservable observations before
  SClineager lineage inference.
when_to_use_negative:
- A valid cleaned.RData already exists for the sample — re-running read_sclineager
  filtering is redundant and slow; proceed to explore_sclineager or run_sclineager
  instead.
- Input is bulk or aggregate sequencing data without per-cell coverage values, where
  per-cell unobservability percentages are undefined.
- The goal is lineage/genotype inference itself rather than preprocessing — residual
  drop-outs are handled by SClineager's Bayesian model, not by harder filtering.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: SClineager
  role: Supplies read_sclineager, which applies coverage_cutoff=3, coverage_percentage=0.2,
    cell_percentage=0.2 and artefact_percentage=0.03 to mark unobservable positions
    and drop failing variants/cells, and run_sclineager's skip_common=T to drop constant-VAF
    variants.
  repo: https://github.com/inmybrain/SClineager
- name: R
  role: Runtime in which the filtering executes (SClineager ver 1.30 in R ver 4.0.3);
    the toy VAF/coverage matrices are loaded via load('./data/toy_data.RData').
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:0b43e6e74b202ba30665c1ffd6a0376787273315cde565cb9827dc33be5a3ae3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/inmybrain/SClineager
  repo_url: https://github.com/inmybrain/SClineager
  tools:
  - R
  - SClineager
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# coverage-based-missingness-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Marks and removes low-quality observations from a single-cell RNA-seq variant (VAF) matrix by applying a sequencing-coverage cutoff and unobservability thresholds before SClineager lineage inference. This addresses the inherent scRNA-seq issues of low coverage and allelic bias that would otherwise corrupt genetic lineage reconstruction.

## When to use

Use when a scRNA-seq mutation (VAF) matrix paired with a per-cell sequencing-coverage matrix must be marked and filtered for low-coverage/unobservable observations before SClineager lineage inference.

## When NOT to use

- A valid cleaned.RData already exists for the sample — re-running read_sclineager filtering is redundant and slow; proceed to explore_sclineager or run_sclineager instead.
- Input is bulk or aggregate sequencing data without per-cell coverage values, where per-cell unobservability percentages are undefined.
- The goal is lineage/genotype inference itself rather than preprocessing — residual drop-outs are handled by SClineager's Bayesian model, not by harder filtering.

## Inputs

- Raw single-cell variant matrix of VAFs (mutations_mat), e.g. from load('./data/toy_data.RData')
- Sequencing coverage matrix (coverage_mat) over the same cells and variants
- runinfo data.frame listing cell names and per-cell paths containing coverage.txt and germline_mutations_hg38.txt

## Outputs

- Per-sample cleaned.RData whose results object holds the filtered mutations_mat, coverage_mat, runinfo (only retained cells, same order), and variant annotation
- summary.pdf per sample visualizing data information
- cleaned_toy_data.RData with the filtered mutations and coverage matrices (toy-data workflow)

## How to apply

In R (SClineager ver 1.30 under R ver 4.0.3), load the raw VAF and coverage matrices, e.g. load('./data/toy_data.RData'), and set the README thresholds: coverage_cutoff <- 3, coverage_percentage <- 0.2, cell_percentage <- 0.2, artefact_percentage <- 0.03. Build a runinfo data.frame mapping each cell to its per-cell coverage.txt folder, then call read_sclineager(runinfo, coverage_cutoff, coverage_percentage, cell_percentage, out_folder, artefact_percentage) once per sample: positions with coverage below 3 are marked unobservable, variants unobservable in more than 20% of cells are dropped, and cells with more than 20% unobservable variants are dropped. The artefact_percentage = 0.03 acts as the artefact threshold during this preprocessing. Variants whose VAF is constant across all cells carry no cell-to-cell information and are excluded, consistent with skip_common <- T passed to run_sclineager; this is the rationale given the two inherent issues of single-cell variant calling (low coverage and allelic bias from monoallelic expression or sampling bias). Each sample's filtered data are written to out_folder as cleaned.RData (a results object holding mutations_mat, runinfo, annotation, coverage_mat) plus summary.pdf, and downstream sclineager_internal consumes the filtered mutations and coverage inputs.

## Related tools

- **SClineager** (Supplies read_sclineager, which applies coverage_cutoff=3, coverage_percentage=0.2, cell_percentage=0.2 and artefact_percentage=0.03 to mark unobservable positions and drop failing variants/cells, and run_sclineager's skip_common=T to drop constant-VAF variants.) — https://github.com/inmybrain/SClineager
- **R** (Runtime in which the filtering executes (SClineager ver 1.30 in R ver 4.0.3); the toy VAF/coverage matrices are loaded via load('./data/toy_data.RData').)

## Examples

```
read_sclineager(runinfo, 3, 0.2, 0.2, "./processed/301", 0.03)
```

## Evaluation signals

- cleaned_toy_data.RData (or per-sample cleaned.RData) exists and loads in R, with results containing mutations_mat, coverage_mat, runinfo, and annotation.
- Variant and cell dimensions of the filtered matrices are strictly smaller than in the raw toy_data matrices.
- No retained cell has more than 20% (cell_percentage = 0.2) of variants below coverage_cutoff = 3 (i.e. unobservable).
- No retained variant is unobservable in more than 20% of cells, and no retained variant has a VAF constant across all cells (skip_common = T semantics).
- results$runinfo lists exactly the cells present in mutations_mat, in the same order, as documented for cleaned.RData.

## Limitations

- Filtering cannot remove expressional drop-outs themselves — low coverage and allelic bias from monoallelic expression or sampling bias persist and are instead modeled downstream by SClineager's Bayesian sampling.
- read_sclineager takes about 30 seconds per mutation-information read, so repeated re-filtering across many samples is slow.
- The cutoffs (3, 0.2, 0.2, 0.03) are README defaults for the example workflow; dropping constant-VAF variants and applying aggressive cutoffs can discard genuinely clonal variants or informative cells if not recalibrated to the dataset.

## Evidence

- [methods] coverage_cutoff <- 3
coverage_percentage <- 0.2
cell_percentage <- 0.2
artefact_percentage <- 0.03: "coverage_cutoff <- 3
coverage_percentage <- 0.2
cell_percentage <- 0.2
artefact_percentage <- 0.03"
- [methods] dfreedom <- 100
  skip_common <- T: "dfreedom <- 100
  skip_common <- T"
- [readme] Single cell variant calling has two inherent issues: (1) low coverage on many positions in many cells: "Single cell variant calling has two inherent issues: (1) low coverage on many positions in many cells"
- [readme] We read each mutation information using `read_sclineager`, which takes about 30 seconds.: "We read each mutation information using `read_sclineager`, which takes about 30 seconds."
- [readme] mutations_mat : raw matrix of variants and VAFs: "mutations_mat : raw matrix of variants and VAFs"
