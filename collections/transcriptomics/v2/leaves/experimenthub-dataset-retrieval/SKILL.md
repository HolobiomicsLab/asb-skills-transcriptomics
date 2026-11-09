---
name: experimenthub-dataset-retrieval
description: Use when an analysis workflow requires downloading a normalized GEO expression
  dataset and its sample annotation (e.g., GSE13015/EH5429) into R as a SummarizedExperiment
  to feed into module-level fingerprinting functions.
when_to_use_negative:
- The data are raw sequencing or array files requiring preprocessing/normalization
  rather than the pre-normalized ExperimentHub resource
- The transcriptome data are from tissues or species other than human blood, since
  the BloodGen3 repertoire is specifically designed for human blood transcriptome
  profiling
- Expression data are already Log2-transformed, which would conflict with the functions'
  internal transformations
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ExperimentHub
  role: Queries and retrieves the normalized GEO expression dataset and annotation
    (GSE13015 via EH5429) directly into R
- name: SummarizedExperiment
  role: Provides the container class for the retrieved expression matrix and sample
    metadata used by BloodGen3Module functions
- name: R
  role: Execution environment in which ExperimentHub retrieval and downstream BloodGen3Module
    analyses run
- name: BloodGen3Module (R package)
  role: Consumes the retrieved SummarizedExperiment as input to Groupcomparison, Groupcomparisonlimma,
    and Individualcomparison
  repo: github.com/Drinchai/BloodGen3Module
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:e7642b89f84b46a4ba492974e2b421a3e040ffa5cdcd3f8b8b78b66799d91277
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module (R package)
  - BloodGen3module (R package)
  - ExperimentHub
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# experimenthub-dataset-retrieval

## Summary

Retrieve normalized blood transcriptome expression matrices and sample annotations (e.g., GSE13015 via ExperimentHub entry EH5429) directly into R as SummarizedExperiment objects, providing the standard input for BloodGen3Module module repertoire analyses.

## When to use

Use when an analysis workflow requires downloading a normalized GEO expression dataset and its sample annotation (e.g., GSE13015/EH5429) into R as a SummarizedExperiment to feed into module-level fingerprinting functions.

## When NOT to use

- The data are raw sequencing or array files requiring preprocessing/normalization rather than the pre-normalized ExperimentHub resource
- The transcriptome data are from tissues or species other than human blood, since the BloodGen3 repertoire is specifically designed for human blood transcriptome profiling
- Expression data are already Log2-transformed, which would conflict with the functions' internal transformations

## Inputs

- GEO accession identifier (e.g., GSE13015) resolvable through ExperimentHub (EH5429)
- R environment with ExperimentHub, SummarizedExperiment, and BloodGen3Module packages installed
- Sample annotation dataframe with a Sample ID column matching the expression matrix and a group column (e.g., Group_test)

## Outputs

- SummarizedExperiment object containing the normalized (non-Log2) expression matrix and sample metadata
- Expression matrix with genes as rows (valid Gene Symbols) and Sample IDs as columns, ready for Groupcomparison/Individualcomparison

## How to apply

Attach the ExperimentHub and SummarizedExperiment libraries in R, create an ExperimentHub object, and query it with the GEO accession (e.g., query(dat, "GSE13015")) to locate the corresponding EH identifier (EH5429). Retrieve the resource with res[["EH5429"]], which returns the normalized expression matrix with genes as rows, valid Gene Symbol row names, and sample IDs as columns, together with sample annotation. Keep the expression data un-Log2-transformed, since BloodGen3Module functions such as Groupcomparison and Individualcomparison perform internal transformations. Feed the retrieved object directly as the first argument to Groupcomparison/Groupcomparisonlimma (FC = 1.5, pval = 0.1, FDR = TRUE) or Individualcomparison (FC = 1.5, DIFF = 10).

## Related tools

- **ExperimentHub** (Queries and retrieves the normalized GEO expression dataset and annotation (GSE13015 via EH5429) directly into R)
- **SummarizedExperiment** (Provides the container class for the retrieved expression matrix and sample metadata used by BloodGen3Module functions)
- **R** (Execution environment in which ExperimentHub retrieval and downstream BloodGen3Module analyses run)
- **BloodGen3Module (R package)** (Consumes the retrieved SummarizedExperiment as input to Groupcomparison, Groupcomparisonlimma, and Individualcomparison) — github.com/Drinchai/BloodGen3Module

## Examples

```
library(ExperimentHub); dat = ExperimentHub(); res = query(dat, "GSE13015"); GSE13015 = res[["EH5429"]]
```

## Evaluation signals

- query(dat, "GSE13015") returns a hit whose EH identifier is EH5429 and res[["EH5429"]] yields a non-empty SummarizedExperiment
- Retrieved expression matrix has genes as rows with valid Gene Symbol row names and sample IDs as columns matching the sample annotation
- Expression values are not Log2-transformed, consistent with the BloodGen3Module input requirement
- Groupcomparison on the retrieved object with FC = 1.5, pval = 0.1, FDR = TRUE produces module-level percentages of increased/decreased constitutive genes without errors
- Sample IDs in the annotation dataframe exactly match the column names of the expression matrix

## Limitations

- The ExperimentHub resource provides pre-normalized data; retrieving raw GEO data would require separate preprocessing before module repertoire analysis
- The BloodGen3 repertoire was built from Illumina BeadArray (probe-level) data, so 388 of 14,168 probes are duplicate genes across modules and require a handling choice
- The annotation dataframe must be loaded/structured in the specific format expected by BloodGen3Module (Sample ID and group columns exactly matching)

## Evidence

- [methods] Query ExperimentHub for the normalized expression data: "res = query(dat , "GSE13015")"
- [methods] Retrieve the EH5429 resource for GSE13015: "GSE13015 = res[["EH5429"]]"
- [readme] README specifies expression data must not be Log2 transformed and requires Gene Symbol row names: [evidence span withheld — anchor status: external_doc]
- [readme] README documents loading example data from ExperimentHub: [evidence span withheld — anchor status: external_doc]
