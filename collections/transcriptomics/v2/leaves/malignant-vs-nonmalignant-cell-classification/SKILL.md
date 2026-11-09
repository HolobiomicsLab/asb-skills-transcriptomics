---
name: malignant-vs-nonmalignant-cell-classification
description: Use when you have a raw gene-by-cell count matrix from an individual
  patient/biopsy tumour sample and need per-cell malignant vs non-malignant (or detailed
  TME cell-type) annotations across the pan-cancer hierarchy.
when_to_use_negative:
- Input is a normal/non-tumour tissue sample — the default scATOMIC model assumes
  at least some cancer cells are present; use the normal_tissue parameter instead
- Sample pools multiple patients/biopsies — scATOMIC must be run on individual patient/biopsy
  samples because inter-patient cancer-cell effects interfere with signature scoring
  and confidence
- Input matrices have not been quality-filtered (cells with >25% mitochondrial reads
  or <500 expressed features can distort classification and downstream CNV inference)
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: scATOMIC
  role: Primary hierarchical classifier producing malignant/non-malignant and TME
    cell-type predictions from count matrices; run via run_scATOMIC() and create_summary_matrix()
  repo: https://github.com/abelson-lab/scATOMIC
- name: copykat
  role: CopyKAT CNV inference invoked inside create_summary_matrix(use_CNVs = TRUE)
    to assign aneuploid/diploid ploidy per cell for cross-validation of malignant
    calls
- name: Rmagic
  role: MAGIC imputation used internally by scATOMIC for imputing values during classification
- name: cutoff.scATOMIC
  role: Fork of the cutoff package (bimodal mixture-model cutoff fitting) used by
    scATOMIC v2 to avoid CRAN name conflicts
  repo: https://github.com/inofechm/cutoff.scATOMIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:04fe2b2d88db73fe8a4132b833fb0fc963247287570203829dd8c10beb5e6676
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - copykat
  - cutoff.scATOMIC
  - scATOMIC
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# malignant-vs-nonmalignant-cell-classification

## Summary

Classifies individual cells from tumour-biopsy scRNA-seq count matrices as malignant or non-malignant using scATOMIC's hierarchical pan-cancer models, optionally cross-checking calls against CopyKAT-inferred CNV ploidy (median agreement 85.9%).

## When to use

Use when you have a raw gene-by-cell count matrix from an individual patient/biopsy tumour sample and need per-cell malignant vs non-malignant (or detailed TME cell-type) annotations across the pan-cancer hierarchy.

## When NOT to use

- Input is a normal/non-tumour tissue sample — the default scATOMIC model assumes at least some cancer cells are present; use the normal_tissue parameter instead
- Sample pools multiple patients/biopsies — scATOMIC must be run on individual patient/biopsy samples because inter-patient cancer-cell effects interfere with signature scoring and confidence
- Input matrices have not been quality-filtered (cells with >25% mitochondrial reads or <500 expressed features can distort classification and downstream CNV inference)

## Inputs

- Raw gene-by-cell count matrices from individual tumour biopsies (scRNA-seq), preferably as sparse matrices (e.g. Seurat @assays$RNA@counts)
- Gene/feature annotation for the count matrix (e.g. GSE161529 features list) to identify mitochondrial genes for QC
- Optional: matched normal tissue reference counts (e.g. Kim et al normal lung sample) to support CopyKAT CNV inference

## Outputs

- Per-cell malignant/non-malignant predictions and detailed TME annotations at every layer of the scATOMIC hierarchy (run_scATOMIC() annotation object)
- create_summary_matrix() summary object reconciled against CopyKAT aneuploid/diploid ploidy calls
- Per-biopsy agreement rates and discordance annotations (e.g. cnv_agreement_rates.csv)

## How to apply

Run scATOMIC per biopsy (never on pooled multi-patient data, because inter-patient cancer-cell effects interfere with its cancer signature scoring), after first removing cells with >25% mitochondrial reads or fewer than 500 unique expressed features and converting counts to a sparse matrix. Call run_scATOMIC() on the filtered sparse matrix to get per-cell predictions at every layer of the pan-cancer hierarchy, then summarize with create_summary_matrix(use_CNVs = TRUE, modify_results = TRUE), which invokes CopyKAT CNV inference (copy_kat_no_heatmap) with multi-core parallel processing via mc.cores. Reconcile scATOMIC's malignant/non-malignant calls against CopyKAT's aneuploid/diploid assignments per biopsy and compute per-biopsy agreement rates; expect a median agreement of ~85.9% with a discordant quartile (Q1) at 63.2%. Annotate discordant biopsies with plausible CNV-related causes: low CNV burden, intra-tumoural malignant subclones with variable CNV burdens, near-diploid cancer cells, few cancer cells harbouring the CNV, or too few normal cells serving as CopyKAT controls. Using scATOMIC in conjunction with CNV inference is explicitly recommended by the authors; note that CNV inference alone cannot assign tissue of origin and yields false negatives in nearly diploid cancers.

## Related tools

- **scATOMIC** (Primary hierarchical classifier producing malignant/non-malignant and TME cell-type predictions from count matrices; run via run_scATOMIC() and create_summary_matrix()) — https://github.com/abelson-lab/scATOMIC
- **copykat** (CopyKAT CNV inference invoked inside create_summary_matrix(use_CNVs = TRUE) to assign aneuploid/diploid ploidy per cell for cross-validation of malignant calls)
- **Rmagic** (MAGIC imputation used internally by scATOMIC for imputing values during classification)
- **cutoff.scATOMIC** (Fork of the cutoff package (bimodal mixture-model cutoff fitting) used by scATOMIC v2 to avoid CRAN name conflicts) — https://github.com/inofechm/cutoff.scATOMIC

## Examples

```
scATOMIC_results <- run_scATOMIC(sparse_matrix); summary <- create_summary_matrix(raw_counts, use_CNVs = TRUE, modify_results = TRUE, mc.cores = 4)
```

## Evaluation signals

- Per-biopsy scATOMIC–CopyKAT agreement reproduces the reported median of 85.9% across the external validation cohort
- Discordant biopsies fall at or below the reported first-quartile agreement threshold of 63.2% and are annotated with plausible CNV-related causes (low CNV burden, subclones, near-diploid cancer cells, few control cells)
- Cells with >25% mitochondrial reads or <500 unique expressed features are removed before classification
- The output annotation object contains predictions at every layer of the pan-cancer hierarchy and can be summarized with create_summary_matrix() and visualized with scATOMICTree()
- cnv_agreement_rates.csv is produced with per-biopsy agreement values and discordance annotations

## Limitations

- CNV inference (CopyKAT) may fall short in samples with cancer sub-clones defined by variable CNV burdens, cancer cells with near-diploid genomic profiles, or few normal cells to serve as controls — a degree of discordancy between scATOMIC and CNV inference is expected
- Solely relying on inferred CNVs to annotate malignant cells can yield false negatives or undefined cells in cancers with minimal structural variation or nearly diploid genomes, and CNV inference cannot provide the cancer's tissue of origin
- The default scATOMIC model assumes the presence of at least some cancer cells and should not be used on normal tissue (use the normal_tissue parameter); for >15,000 cells, use a machine with more than 16 GB RAM; Windows does not natively perform parallel functions
- Cancers of unknown primary and mucinous ovarian carcinoma may be misclassified; bile duct, bone, endometrial, esophageal, gallbladder, and gastric cancer classes are not yet externally validated

## Evidence

- [methods] We strongly recommend the user filter low quality cells from their dataset. We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500: "We strongly recommend the user filter low quality cells from their dataset. We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500"
- [methods] To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object.: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [methods] In this mode we run the create_summary_matrix() function with use_CNVs = TRUE.: "In this mode we run the create_summary_matrix() function with use_CNVs = TRUE."
- [discussion] in samples with cancer sub-clones defined by variable CNV burdens, cancer cells with near diploid genomic profiles, or few normal cells to serve as controls, CNV inference may fall short: "in samples with cancer sub-clones defined by variable CNV burdens, cancer cells with near diploid genomic profiles, or few normal cells to serve as controls, CNV inference may fall short"
- [readme] We strongly recommend running scATOMIC only on individual patient/biopsy samples.: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
- [readme] The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications.: "The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications."
