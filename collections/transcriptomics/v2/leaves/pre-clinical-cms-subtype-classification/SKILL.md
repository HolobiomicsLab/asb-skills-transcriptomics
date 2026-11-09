---
name: pre-clinical-cms-subtype-classification
description: Use when gene expression matrices from CRC cell lines, patient-derived
  organoids, or PDXs need CMS1-CMS4 assignment and the original classifyCMS.RF classifier
  fails or leaves many samples unclassified (e.g., missing CMS4).
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g., patient
  biopsies of metastatic CRC — the authors state CMScaller is not recommended for
  such samples
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller
  is not optimized
- Very small datasets (fewer than approximately forty samples), where centering and
  scaling inescapably introduce prediction uncertainty
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CMScaller
  role: 'Core classifier: NTP-based CMS1-CMS4 prediction of pre-clinical expression
    matrices; also provides CMSgsa, subDEG, and subVolcano for evaluation and visualization'
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment; CMScaller is installed as an R package via devtools::install_github
    and run from R/RStudio
  repo: http://www.R-project.org/
- name: Biobase
  role: Provides ExpressionSet container for expression matrices and is loaded in
    the quick-start workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:fc0bdebb2c80512cecbea356a9d47a519448b7a1562ef2077af797b6d015b390
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pre-clinical-cms-subtype-classification

## Summary

Classify colorectal cancer pre-clinical models (cell lines, organoids, PDXs) into the four Consensus Molecular Subtypes (CMS1-CMS4) using CMScaller, a cancer cell-intrinsic, Nearest Template Prediction-based R classifier that works in the absence of immune and stromal expression signals.

## When to use

Use when gene expression matrices from CRC cell lines, patient-derived organoids, or PDXs need CMS1-CMS4 assignment and the original classifyCMS.RF classifier fails or leaves many samples unclassified (e.g., missing CMS4).

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g., patient biopsies of metastatic CRC — the authors state CMScaller is not recommended for such samples
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller is not optimized
- Very small datasets (fewer than approximately forty samples), where centering and scaling inescapably introduce prediction uncertainty

## Inputs

- Pre-processed gene expression matrix (centered and scaled within dataset) for CRC pre-clinical models, e.g., from GEO/ArrayExpress (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX)
- ExpressionSet object (Biobase) with samples as columns
- Boolean flag indicating RNA-seq vs. microarray platform (RNAseq parameter)

## Outputs

- Per-sample CMS1-CMS4 class predictions (data frame with prediction column from CMScaller)
- CMS informative gene set enrichment results (CMSgsa/Camera output, e.g., cam$CMS4)
- Differential gene expression results per subtype with volcano plot visualization (subDEG + subVolcano)

## How to apply

Retrieve pre-processed gene expression matrices for the pre-clinical datasets, center and scale each matrix within dataset (required by CMScaller; note this introduces prediction uncertainty for datasets with fewer than ~40 samples), and load into an ExpressionSet with Biobase. Run the CMScaller function, which uses the Nearest Template Prediction (NTP) algorithm with subtype-enriched marker templates, setting RNAseq=TRUE for RNA-seq data, to assign CMS1-CMS4 classes per sample. Optionally follow up with CMSgsa (Camera gene set analysis) and subDEG/subVolcano (limma-based differential expression) to evaluate and visualize subtype associations. Validate by checking that all four CMS groups are recovered in every dataset — in the source study, CMS4 accounted for 7.5-22% of 551 pre-clinical samples, whereas the original classifyCMS.RF with default settings left 219 unclassified, called 245 CMS2 and only 3 CMS4. This stromal-independence is the rationale: the original classifier depends on immune and stromal gene expression signals absent in pre-clinical models.

## Related tools

- **CMScaller** (Core classifier: NTP-based CMS1-CMS4 prediction of pre-clinical expression matrices; also provides CMSgsa, subDEG, and subVolcano for evaluation and visualization) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment; CMScaller is installed as an R package via devtools::install_github and run from R/RStudio) — http://www.R-project.org/
- **Biobase** (Provides ExpressionSet container for expression matrices and is loaded in the quick-start workflow)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- All four CMS groups (CMS1-CMS4) are present in every classified dataset; in the source study all 551 pre-clinical samples were classified with no unassigned samples
- Per-dataset CMS4 fraction falls within the reported 7.5-22% range across the seven pre-clinical datasets
- Comparator run of classifyCMS.RF with default settings reproduces the reported failure mode: 219 unclassified, 245 CMS2, and only 3 CMS4 calls out of 551 samples
- Prediction output is a data frame whose head() displays a 'prediction' column with CMS class labels per sample
- Input matrices were centered and scaled within dataset before prediction, as CMScaller requires

## Limitations

- Not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies; caution is warranted in highly selected datasets whose molecular distributions deviate from pCRC cohorts
- Centering and scaling of input data introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Not optimized for single-cell RNA-seq data, which is exceedingly noisy
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries
- The prediction template gene set is planned to be improved as larger, higher-quality datasets become available

## Evidence

- [abstract] Stromal-independence rationale: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] Classifier recapitulates subtypes in 551 models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Scaling requirement and small-sample caveat: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [discussion] Not for metastatic biopsies: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs"
- [readme] Core algorithm and quick-start usage: [evidence span withheld — anchor status: not_found]
- [readme] Example invocation from README: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
