---
name: differential-expression-gene-scoring
description: Use when validating single-cell drug-response predictions on a drug-treated
  scRNA-seq dataset by scoring DEGs and correlating scores with Monocle3-style pseudotime
  or sensitive/resistant probability labels.
when_to_use_negative:
- When the goal is training or predicting drug response from scratch (bulkmodel.py/scmodel.py)
  rather than post-hoc validation of an existing AnnData prediction output
- When no trajectory or time-course structure exists in the scRNA-seq data, since
  pseudotime correlation is the basis of the validation
- When the input is raw expression data rather than an scDEAL AnnData output with
  sens_label probabilities
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDEAL
  role: Produces the predicted sensitive/resistant probabilities (sens_label) and
    provides trajectory.py utilities for Monocle3-style pseudotime inference on AnnData
    objects
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Loads the AnnData .h5ad prediction results (scanpy.read_h5ad) for correlation
    analysis
- name: AnnData
  role: Container format of the scDEAL output carrying cell-level response probabilities
    used for DEG scoring and pseudotime correlation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:06e02ea4483dfd23435b1d655fcacd47179cff61659bc559b371a421db7734b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# differential-expression-gene-scoring

## Summary

Scores single-cell differential-expression (DEG) signal and correlates it with trajectory/pseudotime and predicted drug-response probabilities to validate that scDEAL predictions track drug-response development, as in the Data 6 (GSE110894, I-BET-762) case study matching Fig. 5e-f and Source Data 9-10.

## When to use

Use when validating single-cell drug-response predictions on a drug-treated scRNA-seq dataset by scoring DEGs and correlating scores with Monocle3-style pseudotime or sensitive/resistant probability labels.

## When NOT to use

- When the goal is training or predicting drug response from scratch (bulkmodel.py/scmodel.py) rather than post-hoc validation of an existing AnnData prediction output
- When no trajectory or time-course structure exists in the scRNA-seq data, since pseudotime correlation is the basis of the validation
- When the input is raw expression data rather than an scDEAL AnnData output with sens_label probabilities

## Inputs

- Pre-trained scDEAL AnnData (.h5ad) result object, e.g. adata.zip for GSE110894 with adata.obs['sens_label'] response probabilities
- scDEAL trajectory utilities (trajectory.py)
- casestudy/analysis_tarined_anndata.ipynb template notebook

## Outputs

- pseudotime_correlations.csv containing Pearson correlations (resistant, sensitive, DEG-score vs pseudotime)
- Regression plot of pseudotime versus response probability score (Fig. 5e-f analogue)

## How to apply

Load the pre-trained AnnData result for the dataset (e.g. adata.zip for GSE110894, 1419 mouse AML cells) via scanpy.read_h5ad, extracting scDEAL response probabilities from adata.obs["sens_label"] (0 = resistant, 1 = sensitive). Run Monocle3-style pseudotime trajectory inference on the cells using the scDEAL trajectory utilities (trajectory.py) operating on Scanpy/AnnData objects. Compute Pearson correlations between pseudotime and the predicted resistant and sensitive probability scores (expected r = +0.81 and r = -0.93 for Data 6), and between differential-expression (DEG) scores and pseudotime, to reproduce Fig. 5e-f. Generate a regression plot of pseudotime versus response score following the casestudy/analysis_tarined_anndata.ipynb template, and write all correlation values to pseudotime_correlations.csv.

## Related tools

- **scDEAL** (Produces the predicted sensitive/resistant probabilities (sens_label) and provides trajectory.py utilities for Monocle3-style pseudotime inference on AnnData objects) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Loads the AnnData .h5ad prediction results (scanpy.read_h5ad) for correlation analysis)
- **AnnData** (Container format of the scDEAL output carrying cell-level response probabilities used for DEG scoring and pseudotime correlation)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --checkpoint "save/sc_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
```

## Evaluation signals

- Pearson correlation between resistant probability score and pseudotime matches +0.81 within ±0.05
- Pearson correlation between sensitive probability score and pseudotime matches -0.93 within ±0.05
- pseudotime_correlations.csv exists and contains the required correlation fields (resistant, sensitive, DEG-score)
- Regression plot of pseudotime versus response score reproduces the casestudy/analysis_tarined_anndata.ipynb template (Fig. 5e-f)
- AnnData object successfully loaded via scanpy.read_h5ad with adata.obs['sens_label'] present and coded 0 = resistant, 1 = sensitive

## Limitations

- Validated only on Data 6 (GSE110894, I-BET-762 mouse AML); due to the limited number of drug-treated mouse benchmark scRNA-seq data in the public domain, trans-species reliability could not be systematically evaluated
- Correlations are specific to the I-BET-762 case study and the pre-trained checkpoints distributed in save/adata; other drugs/datasets may not reach the same coefficients

## Evidence

- [abstract] pseudotime case study finding: "We benchmark scDEAL on six scRNA-seq datasets and demonstrate its model interpretability via three case studies focusing on drug response label prediction, gene signature identification, and"
- [intro] integrated gradient signature interpretation: [evidence span withheld — anchor status: not_found]
- [methods] output format for downstream analysis: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
- [readme] single-cell prediction invocation on GSE110894: "python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128""
