---
name: cross-platform-gene-identifier-harmonization
description: Use when assembling circadian time-course datasets from different GEO
  platforms (e.g., GSE54651 with Ensembl IDs, GSE54650 raw CEL files) whose identifiers
  must be unified to uppercase gene names before tauFisher training or cross-method
  benchmarking.
when_to_use_negative:
- Input datasets already share a consistent uppercase gene-symbol identifier space,
  so no mapping or collapsing of non-unique genes is needed
- Analysis targets single-cell resolution circadian prediction — tauFisher cannot
  overcome scRNAseq dropout and works only on bulk or pseudobulk data
- The goal is training tauFisher on diseased or circadian-disturbed samples — a time
  series from diseased individuals is required and healthy-data harmonization will
  not suffice
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: biomaRt
  role: Converts Ensembl gene IDs to gene names during harmonization (applied to GSE54651)
- name: tauFisher
  role: Target pipeline whose preprocessing convention (uppercase gene names, averaging
    non-unique genes) the harmonization reproduces; harmonized matrices feed run_metacycle,
    find_periodic_genes, train_tauFisher, and test_tauFisher
  repo: github.com/micnngo/tauFisher
- name: TimeSignatR
  role: Cross-platform comparator (within-subject renormalization plus elastic net)
    whose platform generalizability motivates harmonized inputs for fair benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:91fdfeb7b8df20b8511eb1326477a1f05abf41ff4ad5cf08aa70ce94b672e2c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TimeSignatR
  - ZeitZeiger
  - biomaRt
  - tauFisher
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-platform-gene-identifier-harmonization

## Summary

Maps heterogeneous GEO transcriptomic gene identifiers (Ensembl IDs, Affymetrix CEL probes, normalized matrices) into a shared uppercase gene-symbol space with non-unique genes averaged, following tauFisher's preprocessing convention. This is the prerequisite for training tauFisher and fairly benchmarking it against TimeSignatR and ZeitZeiger across assay platforms.

## When to use

Use when assembling circadian time-course datasets from different GEO platforms (e.g., GSE54651 with Ensembl IDs, GSE54650 raw CEL files) whose identifiers must be unified to uppercase gene names before tauFisher training or cross-method benchmarking.

## When NOT to use

- Input datasets already share a consistent uppercase gene-symbol identifier space, so no mapping or collapsing of non-unique genes is needed
- Analysis targets single-cell resolution circadian prediction — tauFisher cannot overcome scRNAseq dropout and works only on bulk or pseudobulk data
- The goal is training tauFisher on diseased or circadian-disturbed samples — a time series from diseased individuals is required and healthy-data harmonization will not suffice

## Inputs

- GEO bulk transcriptomic time-course matrices with heterogeneous identifiers (e.g., GSE54651 Ensembl gene IDs)
- Raw Affymetrix CEL files (e.g., GSE54650 kidney and liver)
- Pre-normalized expression matrix (e.g., GSE38622)
- Sample metadata with time points for dataset-specific filtering (e.g., GSE56931 wakefulness/recovery labels)

## Outputs

- Harmonized expression matrices with uppercase gene names and non-unique genes averaged by mean
- Platform-consistent gene-by-sample matrices ready for run_metacycle / find_periodic_genes rhythmicity detection (JTK_pvalue < 0.05) and train_tauFisher / test_tauFisher
- benchmark_results.csv of per-dataset, per-method accuracy and RMSE built on the harmonized inputs

## How to apply

For each benchmark GEO dataset (GSE54650, GSE54651, GSE56931, GSE113883, GSE38622, GSE157077), import the appropriate raw form: raw CEL files for probe-array studies (e.g., GSE54650 kidney and liver) or the normalized matrix where the paper specifies one (GSE38622). Convert Ensembl gene IDs to gene names with the R package biomaRt (as done for GSE54651), then harmonize all matrices by converting gene identifiers to uppercase gene names and averaging expression values of non-unique genes, matching tauFisher's preprocessing. Apply dataset-specific filters while harmonizing: remove GSE56931 time points from the 38-hour continuous wakefulness and recovery sleep, and restrict GSE157077 to mice fed normal chow. The rationale is that tauFisher predicts circadian time irrespective of assay platform, but the comparators are not platform-agnostic — ZeitZeiger failed on several datasets due to linear dependency issues with platform-dependent success, and TimeSignatR relies on within-subject renormalization — so only identifier-harmonized inputs allow a like-for-like 2-hour accuracy and RMSE comparison.

## Related tools

- **biomaRt** (Converts Ensembl gene IDs to gene names during harmonization (applied to GSE54651))
- **tauFisher** (Target pipeline whose preprocessing convention (uppercase gene names, averaging non-unique genes) the harmonization reproduces; harmonized matrices feed run_metacycle, find_periodic_genes, train_tauFisher, and test_tauFisher) — github.com/micnngo/tauFisher
- **TimeSignatR** (Cross-platform comparator (within-subject renormalization plus elastic net) whose platform generalizability motivates harmonized inputs for fair benchmarking)

## Examples

```
library(biomaRt); mart <- useEnsembl("ensembl", dataset="mmusculus_gene_ensembl"); map <- getBM(attributes=c("ensembl_gene_id","external_gene_name"), mart=mart)
```

## Evaluation signals

- No residual Ensembl or probe identifiers remain: every row of the processed matrices is an uppercase gene name, with duplicated (non-unique) genes collapsed by averaging
- GSE56931 matrices exclude the 38-hour continuous wakefulness and recovery sleep time points, and GSE157077 retains only normal-chow mice
- Harmonized matrices yield rhythmic predictor sets via find_periodic_genes with JTK_pvalue < 0.05 and top ten significant 24-hour-period genes per dataset
- After harmonization, tauFisher with one test sample achieves accuracy and RMSE similar to two-sample TimeSignatR on most mouse benchmark datasets
- ZeitZeiger failures on specific datasets are attributable to linear dependency issues rather than identifier mismatches

## Limitations

- Averaging non-unique genes discards potential isoform- or probe-specific signal when collapsing identifiers
- Harmonization alone does not fix comparator failures: ZeitZeiger failed to run on several datasets due to linear dependency issues and its success was dependent on the assay platform
- tauFisher's downstream performance remains subpar on human blood samples even with harmonized inputs, and the top-ten-diurnal-gene choice is safe but not guaranteed optimal

## Evidence

- [discussion] biomaRt converts Ensembl IDs to gene names: "For GSE54651, we converted the Ensembl gene IDs to gene names using the R package biomaRt"
- [discussion] 38-hour wakefulness time points filtered from GSE56931: "we filtered out the time points related to the 38-hour continuous wakefulness and subsequent recovery sleep"
- [discussion] ZeitZeiger platform dependence motivates harmonization: "ZeitZeiger failed to run for several of the datasets due to linear dependency issues, and its success was dependent on the assay platform"
- [intro] TimeSignatR cross-platform renormalization: "TimeSignatR30 applies within-subject renormalization and an elastic net predictor, making it generalizable between transcriptomic data from different assay platforms."
- [intro] tauFisher is platform-agnostic, so harmonized inputs transfer: "Here, we present tauFisher, a pipeline that can accurately predict circadian time from a single transcriptomic data irrespective of the assay platform."
- [readme] R environment for the harmonization/tauFisher toolchain: [evidence span withheld — anchor status: external_doc]
