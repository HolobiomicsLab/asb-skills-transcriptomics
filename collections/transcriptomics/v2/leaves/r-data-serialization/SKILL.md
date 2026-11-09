---
name: r-data-serialization
description: Reference for chaining R-based scRNA-seq variant preprocessing (SClineager
  read_sclineager/run_sclineager) or any workflow that must save/reload filtered mutations_mat
  and coverage_mat objects between steps.
when_to_use_negative:
- Data exchange is needed with non-R tools (Python, Snakemake, Galaxy) — .RData is
  R-specific; export CSV/CSV-like flat files instead.
- The matrices are already loaded in the same R session and the next step (e.g. run_sclineager)
  can read them directly from the existing cleaned.RData on disk.
- Long-term archival of results is required — RData serialization is version-sensitive
  across R and package versions (e.g. SClineager 1.30 under R 4.0.3).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SClineager
  role: 'Produces and consumes the serialized RData artifacts: read_sclineager writes
    cleaned.RData and run_sclineager loads it and writes results.RData'
  repo: https://github.com/inmybrain/SClineager
- name: R
  role: Runtime environment whose load()/save() functions perform the serialization
    and deserialization of the matrices and result objects
  repo: https://github.com/inmybrain/SClineager
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:df83c8db1a57f4de53533f1ff294a267a7a703447a5d6b11521bab4cfe17ad13
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# r-data-serialization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Persist and reload intermediate R analysis objects (VAF mutation matrices, coverage matrices, run info, annotations) as .RData files so that SClineager preprocessing, filtering, and MCMC lineage-inference stages can be chained and validated across sessions.

## When to use

Reference for chaining R-based scRNA-seq variant preprocessing (SClineager read_sclineager/run_sclineager) or any workflow that must save/reload filtered mutations_mat and coverage_mat objects between steps.

## When NOT to use

- Data exchange is needed with non-R tools (Python, Snakemake, Galaxy) — .RData is R-specific; export CSV/CSV-like flat files instead.
- The matrices are already loaded in the same R session and the next step (e.g. run_sclineager) can read them directly from the existing cleaned.RData on disk.
- Long-term archival of results is required — RData serialization is version-sensitive across R and package versions (e.g. SClineager 1.30 under R 4.0.3).

## Inputs

- Raw VAF mutation matrix (mutations_mat, variants x cells)
- Coverage matrix (coverage_mat) aligned to mutations_mat
- runinfo data.frame of cells (Cell, Path)
- Variant annotation records
- SClineager model output object `results` (genotype_mat, sigma, etc.)

## Outputs

- cleaned.RData per sample containing R object `results` (mutations_mat, runinfo, annotation, coverage_mat)
- results.RData containing genotype_mat, genotype_mat_orig, sigma, genotype_mat_all, sigma_all, runinfo, annotation
- cleaned_toy_data.RData holding filtered mutations_mat and coverage_mat for the toy dataset

## How to apply

In the SClineager workflow, read_sclineager writes one cleaned.RData per sample into each ./processed/<sample>/ folder; the saved R object named `results` bundles mutations_mat (raw variant-by-cell VAF matrix), coverage_mat, runinfo, and variant annotation. The downstream run_sclineager step then loads this file via its file_in argument (e.g. './processed/301/cleaned.RData') and writes its own results.RData containing genotype_mat, genotype_mat_orig, sigma, genotype_mat_all, sigma_all, runinfo and annotation. For filtered toy data, load the raw matrices with load('./data/toy_data.RData'), apply coverage_cutoff = 3, coverage_percentage = 0.2, cell_percentage = 0.2 and artefact_percentage = 0.03 filtering, then save the reduced matrices with save(..., file='cleaned_toy_data.RData'). Persisting both the pre-filter and post-filter RData files makes the filtering auditable: variant/cell counts must decrease, no remaining variant may have constant VAF across cells, and no cell may retain more than 20% unobservable variants.

## Related tools

- **SClineager** (Produces and consumes the serialized RData artifacts: read_sclineager writes cleaned.RData and run_sclineager loads it and writes results.RData) — https://github.com/inmybrain/SClineager
- **R** (Runtime environment whose load()/save() functions perform the serialization and deserialization of the matrices and result objects) — https://github.com/inmybrain/SClineager

## Examples

```
load("./data/toy_data.RData"); save(mutations_mat, coverage_mat, file="cleaned_toy_data.RData")
```

## Evaluation signals

- cleaned_toy_data.RData exists and is loadable via load() without error in R
- Loaded filtered matrices have fewer variants and cells than the raw toy_data matrices after applying coverage_cutoff=3 and 20% cutoffs
- No remaining variant has a VAF constant across all cells (skip_common = T consistency)
- No remaining cell has more than 20% unobservable variants (coverage < 3)
- cleaned.RData's `results` object exposes all four required elements: mutations_mat, runinfo, annotation, coverage_mat, with coverage_mat sharing dimensions/order with mutations_mat

## Limitations

- RData files are R-version and package-version sensitive; results saved under R 4.0.3 / SClineager 1.30 may fail to load or misbehave under other versions.
- cleaned.RData is produced per sample (e.g. 301, 304); the analysis must be run and serialized independently for each sample, so there is no single cross-sample serialized object.
- The mutation .txt inputs follow a fixed folder layout (./mutations/<sample>/<cell>/coverage.txt and germline_mutations_hg38.txt); deviating layouts break read_sclineager's serialization step.

## Evidence

- [readme] cleaned.RData contains an R object named `results`, which has the following information: "After running the above code, we get for each mutation a RData file that
will be used in the SClineager model"
- [readme] coverage_mat holds coverage for the same cells and variants: "`coverage_mat` : coverage data of the same cells and same variants as
  in `mutations_mat`"
- [other] Workflow card: load toy dataset and save filtered matrices to cleaned_toy_data.RData: [evidence span withheld — anchor status: not_found]
