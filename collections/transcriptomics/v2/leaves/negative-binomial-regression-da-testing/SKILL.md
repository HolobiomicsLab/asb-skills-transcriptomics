---
name: negative-binomial-regression-da-testing
description: Use when significant condition-enriched cell communities (e.g. from EVA/ProcessEVA
  with a Bonferroni-thresholded binomial test) have been identified on a cell-cell
  kNN graph and their cell counts must be formally tested for differential abundance
  across a condition such as age or disease stage.
when_to_use_negative:
- Input is raw counts without an established cell-cell graph and condition-enriched
  cluster set — ELVAR's DA test operates on communities from EVA clustering, not on
  raw cell-type labels alone
- Counts are not overdispersed or no sample-replicate structure exists to model inter-subject
  variability — a simpler test may suffice
- Cell-type annotation FPs are the concern rather than clustering-attribute noise
  — the article notes ELVAR is more sensitive to FPs in the clustering (sample/phenotype)
  attribute than in cell-type annotations
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_2229
tools:
- name: ELVAR
  role: R-package whose DoDA function performs the negative binomial regression DA
    test on condition-enriched communities
  repo: https://github.com/aet21/ELVAR
- name: MASS
  role: R library providing negative binomial regression machinery (glm.nb-style fitting)
    used by DoDA
- name: Seurat
  role: Holds the cell metadata (condition and replicate variables) and the scRNA-Seq
    object passed to DoDA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:4e22e7f92389dd0f3b65b00266f752e3093d94d5c4ac80160af0910de9f53fbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# negative-binomial-regression-da-testing

## Summary

Fit a negative binomial regression (NBR) to the cell counts of condition-enriched cell communities to test for differential abundance, using a biological condition variable and a sample-replicate variable as covariates. In ELVAR this is implemented as the DoDA function, which reports Wald and likelihood-ratio test (LRT) statistics for the DA effect.

## When to use

Use when significant condition-enriched cell communities (e.g. from EVA/ProcessEVA with a Bonferroni-thresholded binomial test) have been identified on a cell-cell kNN graph and their cell counts must be formally tested for differential abundance across a condition such as age or disease stage.

## When NOT to use

- Input is raw counts without an established cell-cell graph and condition-enriched cluster set — ELVAR's DA test operates on communities from EVA clustering, not on raw cell-type labels alone
- Counts are not overdispersed or no sample-replicate structure exists to model inter-subject variability — a simpler test may suffice
- Cell-type annotation FPs are the concern rather than clustering-attribute noise — the article notes ELVAR is more sensitive to FPs in the clustering (sample/phenotype) attribute than in cell-type annotations

## Inputs

- Seurat object (e.g. built with FindVariableFeatures vst, ScaleData, RunPCA, FindNeighbors dims 1:8, k.param=20) with per-cell condition metadata such as age or phenotype
- Significant-cluster object from ProcessEVA identifying condition-enriched communities (Eva_partitions over alpha 0.1-0.9, resolution=1, threshold=0.0000001)
- Condition variable name (varDA, e.g. 'naive' / age attribute) and sample-replicate variable name (varREP, e.g. 'mouse') from the Seurat metadata

## Outputs

- DoDA result object containing negative binomial regression fits per condition-enriched cluster with Wald and LRT statistics for differential abundance
- Per-run DA significance calls, aggregated into sensitivity (fraction of significant DA detections) and z-statistic distributions across replicates and scenarios (e.g. Fig. 6 and Supplementary Fig. S8)

## How to apply

After selecting condition-enriched communities with ProcessEVA (binomial test with Bonferroni threshold, merging multiple enriched clusters per condition), run DoDA on the Seurat object with the significant-cluster object, passing the condition variable (e.g. 'naive' for the naive-to-mature Cd4+ T-cell shift in lung tissue) as varDA and the sample-replicate variable (e.g. 'mouse' in the Tabula Muris Senis lungCD4T data) as varREP. The negative binomial model accounts for biological inter-subject variability and overdispersed counts, so naive count comparisons are avoided. Record both the Wald and LRT statistics per cluster; run the analysis across repeated EVA partitions (~100 repeats per alpha setting) and aggregate z-statistic distributions and sensitivity (fraction of significant DA detections) to assess robustness, e.g. across batch-correction scenarios (none, Harmony, Seurat integration). Validation should reproduce the tutorial's highly significant naive-to-mature Cd4+ T-cell DA result under the no-correction scenario.

## Related tools

- **ELVAR** (R-package whose DoDA function performs the negative binomial regression DA test on condition-enriched communities) — https://github.com/aet21/ELVAR
- **MASS** (R library providing negative binomial regression machinery (glm.nb-style fitting) used by DoDA)
- **Seurat** (Holds the cell metadata (condition and replicate variables) and the scRNA-Seq object passed to DoDA)

## Examples

```
da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");
```

## Evaluation signals

- The no-batch-correction run should reproduce the tutorial's highly significant naive-to-mature Cd4+ T-cell DA result, with both Wald and LRT statistics reported by DoDA
- Sensitivity (fraction of significant DA detections) from batch-corrected runs (Harmony or Seurat integration) should rarely exceed the no-correction scenario, with generally higher significance without prior correction
- Under simulated false positives, sensitivity loss should be larger when label noise is in the phenotype/clustering attribute than in cell-type annotations
- Per-cluster Wald and LRT statistics should be recorded for every run and their distributions aggregated across ~100 EVA repeats per alpha setting
- Enriched-cluster selection entering DoDA must come from ProcessEVA with a Bonferroni-thresholded binomial test, merging multiple enriched clusters per condition

## Limitations

- ELVAR is sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect the quality of inferred communities and thus downstream DA testing
- DA significance is study-specific: the degree of significance was generally higher without prior batch correction, but the batch-correction effect varied by study
- No benchmark against miloDE was performed; the authors flag this as an open question, and ELVAR has increased computational complexity and runtimes compared to Milo and the Louvain-analog

## Evidence

- [other] Run DoDA negative binomial regression: [evidence span withheld — anchor status: not_found]
- [methods] DoDA invocation: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [intro] Confounding motivation: "biological inter-subject variability, as well as technical batch effects, can potentially confound naïve DA-analyses"
- [discussion] Sensitivity vs false positives: "it was more sensitive to putative FPs in the clustering attribute representing the sample/phenotype label."
- [discussion] DA result example: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts, such as the age-related shift from naïve to mature Cd4+ T-cells"
