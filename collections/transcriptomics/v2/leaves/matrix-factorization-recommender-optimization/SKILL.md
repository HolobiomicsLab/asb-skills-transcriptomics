---
name: matrix-factorization-recommender-optimization
description: Use when you need to fit a latent-factor recommender model to a drug–cell-line
  response matrix (IC50 or AUC) and predict drug response from kernelized expression
  features, as in GDSC/PRISM pharmacogenomic modeling.
when_to_use_negative:
- The goal is de novo prediction of sensitivity to compounds never experimentally
  screened in GDSC or PRISM — scDrug's prediction is limited by experimentally screened
  compounds
- Input is raw scRNA-seq data needing preprocessing/clustering rather than a drug-response
  matrix to factorize — use the Single-Cell Data Analysis pipeline first
- A pre-trained GDSC model suffices (CaDRReS-Sc ships one); only retrain when a different
  response dataset or scale transformation is required
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CaDRReS-Sc
  role: Recommender-system (matrix factorization) framework trained on GDSC response
    data with kernel features from CCLE expression
  repo: https://github.com/CSB5/CaDRReS-SC
- name: scDrug
  role: Wraps CaDRReS-Sc model training/prediction; invokes drug_response_prediction.py
    with --model GDSC or --model PRISM
  repo: https://github.com/ailabstw/scDrug
- name: Python
  role: Python 3.x environment (with Pandas, Numpy, TensorFlow 1.14 for CaDRReS-Sc)
    in which the factorization models are trained
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:27a647d37cca308f00fe4d93c7f2e39618632209a6f1e3c62072c45b8c836911
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSB5/CaDRReS-SC
  tools:
  - CaDRReS-Sc
  - Python
  - scDrug
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# matrix-factorization-recommender-optimization

## Summary

Train modified CaDRReS-Sc recommender (matrix-factorization) models on GDSC IC50 and PRISM AUC drug-response matrices using kernel features derived from CCLE 21Q3 expression, to learn drug–cell-line latent interactions for drug response prediction. The skill also validates model outputs by the Pearson correlation (0.615) between PRISM (1 − AUC) and GDSC (−log(IC50)) values across shared drugs.

## When to use

Use when you need to fit a latent-factor recommender model to a drug–cell-line response matrix (IC50 or AUC) and predict drug response from kernelized expression features, as in GDSC/PRISM pharmacogenomic modeling.

## When NOT to use

- The goal is de novo prediction of sensitivity to compounds never experimentally screened in GDSC or PRISM — scDrug's prediction is limited by experimentally screened compounds
- Input is raw scRNA-seq data needing preprocessing/clustering rather than a drug-response matrix to factorize — use the Single-Cell Data Analysis pipeline first
- A pre-trained GDSC model suffices (CaDRReS-Sc ships one); only retrain when a different response dataset or scale transformation is required

## Inputs

- GDSC drug response matrix (measured IC50, 226 drugs × 1074 cancer cell lines) provided by CaDRReS-Sc
- PRISM Repurposing dataset version 19Q4 (AUC responses, 1448 drugs × 480 cell lines)
- CCLE expression data version 21Q3 from the DepMap Portal (1,379 cell lines, 19,177 genes)

## Outputs

- Trained CaDRReS-Sc matrix-factorization models (10-dim GDSC, 140-dim PRISM latent spaces)
- Predicted response matrices (IC50_prediction.csv / AUC_prediction.csv)
- Drug-wise Spearman correlation coefficients between actual and predicted response
- prism_gdsc_correlation.csv containing the overall PRISM (1 − AUC) vs GDSC (−log(IC50)) Pearson correlation

## How to apply

Load the GDSC response data (measured IC50, 226 drugs × 1074 cell lines) from CaDRReS-Sc and the PRISM Repurposing dataset 19Q4 (1448 drugs × 480 cell lines), then transform response scales: GDSC IC50 to −log(IC50) and PRISM AUC to scaled 1 − AUC with D′ = 2.40D − 1.20. Build kernel features from CCLE 21Q3 expression: 17,419 common genes across 1,018 cell lines for GDSC, and 8,087 genes with ≥0.2 absolute Pearson correlation between expression and PRISM AUC for PRISM, computing log2 expression fold-change per gene and Pearson-correlation cell–cell similarity. Train the modified CaDRReS-Sc models (drug bias only — no cell bias or overall mean term — with L2 regularization) in a 140-dimensional latent space for PRISM and 10-dimensional for GDSC, learning rate 0.01, maximum 100,000 epochs. Evaluate with drug-wise Spearman correlations between actual and predicted response, dropping drugs below the first-quartile coefficient. The rationale is that the low-rank pharmacogenomic space regularizes against sparse, noisy drug screens while the drug-bias-only variant suits the response-scale transformation.

## Related tools

- **CaDRReS-Sc** (Recommender-system (matrix factorization) framework trained on GDSC response data with kernel features from CCLE expression) — https://github.com/CSB5/CaDRReS-SC
- **scDrug** (Wraps CaDRReS-Sc model training/prediction; invokes drug_response_prediction.py with --model GDSC or --model PRISM) — https://github.com/ailabstw/scDrug
- **Python** (Python 3.x environment (with Pandas, Numpy, TensorFlow 1.14 for CaDRReS-Sc) in which the factorization models are trained)

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./output -m GDSC
```

## Evaluation signals

- The overall Pearson correlation between PRISM (1 − AUC) and GDSC (−log(IC50)) across shared drugs reproduces the reported value of 0.615
- Drug-wise Spearman correlations between actual and predicted response are computed and all drugs below the first-quartile coefficient are dropped
- Training loss converges within the specified budget (learning rate 0.01, max 100,000 epochs) without divergence
- GDSC model uses exactly 10 latent dimensions and the PRISM model 140 latent dimensions, with drug bias only (no cell bias or overall mean term)
- Feature-gene counts match expectations: 17,419 common genes for GDSC and 8,087 genes passing the |Pearson r| ≥ 0.2 PRISM AUC correlation filter

## Limitations

- The prediction of drug response in scDrug is limited by experimentally screened compounds — no de novo sensitivity prediction is implemented
- Drugs whose drug-wise Spearman coefficient falls below the first quartile must be discarded, reducing the set of usable predictions
- Lack of scRNA-seq data with drug treatments for evaluation; extensive experimental validations of the pipeline are still necessary

## Evidence

- [methods] GDSC training data definition: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [methods] PRISM training data definition: "we used the PRISM Repurposing dataset (version 19Q4) as the training data, which contained the responses of 1448 drugs against 480 cell lines"
- [methods] PRISM feature-gene selection threshold: "We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes"
- [readme] Pre-trained GDSC model availability in CaDRReS-Sc: [evidence span withheld — anchor status: external_doc]
- [methods] Drug-wise correlation filtering rule: "we dropped the drugs whose drug-wise coefficient was lower than the first quartile coefficient."
