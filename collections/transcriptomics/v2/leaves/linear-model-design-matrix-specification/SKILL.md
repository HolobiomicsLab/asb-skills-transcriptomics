---
name: linear-model-design-matrix-specification
description: Use when an RNA-seq count table has a primary condition of interest but
  samples also differ by a nuisance variable (batch, species, individual) that must
  be blocked in the limma design before calling DEGs.
when_to_use_negative:
- The input is raw FASTQ reads rather than a count table — reads must first be quantified/annotated
  (e.g., Seq2Fun) before design specification.
- A single-factor design with no blocking variable exists — SetSelectedMetaInfo can
  be called with 'NA' for secondary and blocking factors instead of forcing a block.
- The goal is meta-analysis across multiple datasets (SetAnalType("metadata")), which
  uses PerformDEAnalMeta per dataset rather than a single shared design matrix.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalystR
  role: R package whose SetSelectedMetaInfo and SetupDesignMatrix functions define
    the metadata factors and build the limma design matrix locally
  repo: github.com/xia-lab/ExpressAnalystR
- name: limma
  role: DE algorithm selected in SetupDesignMatrix; builds the design matrix and fits
    the linear model with voom-normalized log2 CPM values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:ff1016b7bd9475a4577c53cba9e383e42bd466ac4f819a1c3ad1221d573755bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - limma
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# linear-model-design-matrix-specification

## Summary

Specify a limma linear-model design matrix within ExpressAnalystR by selecting a primary metadata factor plus a blocking factor, so that multi-factor designs (e.g., time while blocking on species) are handled correctly in differential expression analysis.

## When to use

Use when an RNA-seq count table has a primary condition of interest but samples also differ by a nuisance variable (batch, species, individual) that must be blocked in the limma design before calling DEGs.

## When NOT to use

- The input is raw FASTQ reads rather than a count table — reads must first be quantified/annotated (e.g., Seq2Fun) before design specification.
- A single-factor design with no blocking variable exists — SetSelectedMetaInfo can be called with 'NA' for secondary and blocking factors instead of forcing a block.
- The goal is meta-analysis across multiple datasets (SetAnalType("metadata")), which uses PerformDEAnalMeta per dataset rather than a single shared design matrix.

## Inputs

- Gene-level count table (e.g., salamander KO counts for samples SRR7499348–SRR7499365) loaded via ReadTabExpressData
- Sample metadata table containing the primary factor (time) and the blocking factor (species)
- Normalization choice (limma voom / log2 CPM for RNA-seq)

## Outputs

- A limma design matrix encoding primary factor and blocking factor
- DE results table (logFC, p-value, FDR-adjusted p-value per gene) suitable for export to salamander_de_results.csv
- DEG count at the chosen threshold (2780 at FDR < 0.1 in the case study)

## How to apply

In ExpressAnalystR local mode, initialize with Init.Data(FALSE) and SetAnalType("onedata"), load the count table with ReadTabExpressData, then normalize with limma voom (log2 CPM) for RNA-seq counts. Call SetSelectedMetaInfo to designate the primary factor (e.g., time) and a secondary/blocking factor (e.g., species), then immediately call SetupDesignMatrix with the "limma" algorithm to build the design matrix — the README stresses that every function must be run in sequence and none skipped. For microarray data only limma is supported; for RNA-seq the same design matrix feeds PerformDEAnal with a custom contrast such as time0 vs. time24. In the salamander case study, all 18 samples (SRR7499348–SRR7499365, minus outlier SRR7499350) are analyzed together with species as the blocking factor, and DEGs are defined at FDR-adjusted p < 0.1 per the original publication.

## Related tools

- **ExpressAnalystR** (R package whose SetSelectedMetaInfo and SetupDesignMatrix functions define the metadata factors and build the limma design matrix locally) — github.com/xia-lab/ExpressAnalystR
- **limma** (DE algorithm selected in SetupDesignMatrix; builds the design matrix and fits the linear model with voom-normalized log2 CPM values)

## Examples

```
dataSets <- SetSelectedMetaInfo("estrogen.txt","ER", "NA", F); dataSets <- SetupDesignMatrix("estrogen.txt", "limma");
```

## Evaluation signals

- The design matrix contains columns for the primary factor (time) and the blocking factor (species), and the contrast is specified as a custom pairwise comparison (time0 vs. time24).
- All ExpressAnalystR functions are executed in sequence with none skipped, as the README warns skipping commands causes downstream errors.
- The number of DEGs at FDR-adjusted p < 0.1 (no log2FC cut-off) matches the 2780 reported for the salamander case study.
- Outlier sample SRR7499350 is excluded from the 18 samples before the design matrix is built, matching the QA/QC of the original publication.
- The written results table contains per-gene logFC, p-value, and FDR-adjusted p-value columns.

## Limitations

- For microarray data, only the limma algorithm can be used in SetupDesignMatrix.
- ExpressAnalystR is still under development and full functionality is not guaranteed; it returns values saved as qs files rather than useful console output.
- Reads from the same gene may map to similar but different orthologs across species, so blocked cross-species designs require care in interpretation.

## Evidence

- [readme] SetSelectedMetaInfo and SetupDesignMatrix define factors then build design: "After selecting metadata, compute design matrix and select DE analysis algorithm by running ``SetupDesignMatrix`` function. For microarray data, only ``limma`` can be used."
- [readme] Functions must run in sequence: "Every function must be executed in sequence as it is shown on the R Command history, please do not skip any commands as this can result in errors downstream."
- [discussion] Salamander FDR threshold from the paper: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [discussion] Outlier exclusion before analysis: "excluding sample SRR7499350 which was identified as an outlier by QA/QC performed by the original publication"
- [results] Blocking factor used in salamander case study: [evidence span withheld — anchor status: not_found]
