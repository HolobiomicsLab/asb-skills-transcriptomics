---
name: ploidy-vs-annotation-agreement-analysis
description: Use when you have per-cell malignant/non-malignant predictions from scATOMIC
  on tumour-biopsy scRNA-seq and want to benchmark them against CNV-inferred ploidy
  status from CopyKAT.
when_to_use_negative:
- 'The sample is normal/non-cancer tissue: scATOMIC''s default model assumes at least
  some cancer cells are present, so ploidy-agreement validation is inappropriate (use
  the normal_tissue parameter instead)'
- The goal is to determine a cancer's tissue of origin rather than validate malignant-cell
  calls — CNV inference cannot provide the cancer's tissue of origin
- The dataset is already a curated cell-type annotation table with no raw counts to
  re-run CopyKAT CNV inference on
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: Generates the per-cell malignant/non-malignant (and pan-cancer hierarchy)
    predictions that are reconciled against CNV ploidy calls
  repo: https://github.com/abelson-lab/scATOMIC
- name: copykat
  role: Performs CNV inference (copy_kat_no_heatmap) inside create_summary_matrix(use_CNVs
    = TRUE) to assign aneuploid/diploid ploidy per cell
- name: R
  role: Environment for running the scATOMIC/CopyKAT agreement workflow and writing
    cnv_agreement_rates.csv
- name: Rmagic
  role: Used by scATOMIC to impute values during classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:5ca86421991191e35c66ad36d7f41a113d86bb0c1e5edc7b54dfa877126fd58b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - copykat
  - scATOMIC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ploidy-vs-annotation-agreement-analysis

## Summary

Cross-validates scATOMIC's malignant/non-malignant cell annotations against CopyKAT-inferred CNV ploidy (aneuploid vs diploid) per biopsy, quantifying agreement and annotating discordant samples with CNV-related failure causes.

## When to use

Use when you have per-cell malignant/non-malignant predictions from scATOMIC on tumour-biopsy scRNA-seq and want to benchmark them against CNV-inferred ploidy status from CopyKAT.

## When NOT to use

- The sample is normal/non-cancer tissue: scATOMIC's default model assumes at least some cancer cells are present, so ploidy-agreement validation is inappropriate (use the normal_tissue parameter instead)
- The goal is to determine a cancer's tissue of origin rather than validate malignant-cell calls — CNV inference cannot provide the cancer's tissue of origin
- The dataset is already a curated cell-type annotation table with no raw counts to re-run CopyKAT CNV inference on

## Inputs

- Raw gene-by-cell count matrices (sparse) for external validation tumour biopsies
- scATOMIC run_scATOMIC() per-cell predictions at every layer of the pan-cancer hierarchy
- CopyKAT aneuploid/diploid ploidy assignments per cell (generated via create_summary_matrix with use_CNVs = TRUE)

## Outputs

- Per-biopsy agreement rates between scATOMIC malignant/non-malignant calls and CopyKAT ploidy assignments
- Discordance annotations (low CNV burden, subclones, near-diploid cancer cells, few normal control cells)
- cnv_agreement_rates.csv summarizing per-biopsy agreement and discordance causes

## How to apply

Filter each gene-by-cell raw count matrix by removing cells with >25% mitochondrial reads or fewer than 500 unique expressed features, and convert matrices to sparse format. Run scATOMIC's run_scATOMIC() per biopsy (it should only be run on individual patient/biopsy samples because inter-patient cancer-cell effects interfere with its cancer signature scoring), then call create_summary_matrix() with use_CNVs = TRUE and modify_results = TRUE, which invokes CopyKAT CNV inference (copy_kat_no_heatmap) on the raw counts; pass mc.cores for multi-core parallel processing to speed up the CNV correction. For every biopsy, reconcile scATOMIC's malignant/non-malignant calls against CopyKAT's aneuploid/diploid assignments and compute the per-biopsy agreement rate. Summarize agreement across the cohort (median 85.9%, first-quartile 63.2%) and flag biopsies below the Q1 threshold as discordant. Annotate discordant biopsies with plausible CNV-related causes: low CNV burden, intra-tumoural malignant subclones with variable CNV levels, near-diploid cancer cells harbouring few CNVs, or too few normal cells serving as CopyKAT controls.

## Related tools

- **scATOMIC** (Generates the per-cell malignant/non-malignant (and pan-cancer hierarchy) predictions that are reconciled against CNV ploidy calls) — https://github.com/abelson-lab/scATOMIC
- **copykat** (Performs CNV inference (copy_kat_no_heatmap) inside create_summary_matrix(use_CNVs = TRUE) to assign aneuploid/diploid ploidy per cell)
- **R** (Environment for running the scATOMIC/CopyKAT agreement workflow and writing cnv_agreement_rates.csv)
- **Rmagic** (Used by scATOMIC to impute values during classification)

## Examples

```
results <- create_summary_matrix(mat, use_CNVs = TRUE, modify_results = TRUE, mc.cores = 8)
```

## Evaluation signals

- The per-biopsy agreement table reproduces the reported median scATOMIC–CopyKAT agreement of 85.9% across the validation cohort
- The discordant-quartile threshold (Q1) of 63.2% agreement is recovered, and biopsies below it are flagged as discordant
- Every discordant biopsy is annotated with at least one plausible CNV-related cause (low CNV burden, subclonal variable CNVs, near-diploid cancer cells, or few normal CopyKAT control cells)
- Upstream QC is verifiable: no cells with >25% mitochondrial reads or <500 expressed features remain in the filtered matrices
- cnv_agreement_rates.csv is produced with per-biopsy agreement rates and discordance annotations matching the schema

## Limitations

- CNV inference may fall short in samples with cancer sub-clones of variable CNV burdens, near-diploid cancer cells, or few normal cells to serve as controls
- Solely relying on inferred CNVs to call malignant cells can yield false negatives in cancers with minimal genomic structural variation or nearly diploid genomes
- A degree of discordancy between scATOMIC and CNV inference persists even in validated tumour biopsies
- scATOMIC should be run only on individual patient/biopsy samples, and the default model assumes the presence of at least some cancer cells

## Evidence

- [methods] remove any cells with more than 25% of reads mapping to mitochondrial genes: "remove any cells with more than 25% of reads mapping to mitochondrial genes"
- [methods] CNV correction mode: "In this mode we run the create_summary_matrix() function with use_CNVs = TRUE."
- [discussion] CNV inference failure modes: "in samples with cancer sub-clones defined by variable CNV burdens, cancer cells with near diploid genomic profiles, or few normal cells to serve as controls, CNV inference may fall short"
- [discussion] combined use of scATOMIC and CNV inference: "using scATOMIC in conjunction with CNV inference to annotate cancer cells and their type is recommended"
- [intro] CNV-only malignant calling risk: "Solely relying on the presence of inferred CNVs to annotate malignant cells may lead to false negatives or undefined cells in cancers with minimal genomic structural variation"
- [readme] per-biopsy-only recommendation: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
