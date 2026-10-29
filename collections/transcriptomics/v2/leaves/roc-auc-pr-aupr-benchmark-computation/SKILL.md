---
name: roc-auc-pr-aupr-benchmark-computation
description: Use when validating a fusion/artifact classifier (e.g., scFusion's bi-LSTM)
  on held-out scRNA-seq candidates where median ROC AUC and PR AUPR across datasets
  quantify discrimination of artifacts.
when_to_use_negative:
- Input is raw FASTQ or unaligned BAM — run scFusion's alignment and candidate-extraction
  pipeline first; ROC/AUPR scoring requires labeled fusion candidates
- Input fusion calls are unlabeled or have no artifact/real ground truth, so ROC AUC
  and PR AUPR cannot be computed
- Evaluating sensitivity for low-expression fusions or rare fusions in heterogeneous
  tumors — scFusion has known reduced sensitivity there, which these metrics alone
  will not capture
edam_operation: http://edamontology.org/operation_0306
tools:
- name: scFusion
  role: Pipeline producing the labeled chimeric-fusion candidates and deep-learning
    artifact scores that are evaluated with ROC AUC and PR AUPR
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligns paired Smart-Seq single-cell FASTQ files to hg19 (>= 2.7.2d) as the
    first step generating candidates to be scored
  repo: https://github.com/alexdobin/STAR
- name: TensorFlow
  role: Trains the bi-LSTM artifact-scoring model whose held-out predictions yield
    ROC AUC and PR AUPR
- name: pyensembl
  role: Annotates fusion candidates with gene_name and gene_type/gene_biotype fields
    to build model features
  repo: https://github.com/openvax/pyensembl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:485f7696488df65c946e7101dc2702f0f3ac0d41a4a613626432bfda30e6dd45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Keras
  - Python
  - R
  - STAR
  - TensorFlow
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# roc-auc-pr-aupr-benchmark-computation

## Summary

Compute ROC AUC and precision-recall AUPR to benchmark a bi-LSTM artifact-scoring model that distinguishes technical chimeric fusion artifacts from real fusions in scRNA-seq data, using scFusion's per-dataset held-out fusion candidates.

## When to use

Use when validating a fusion/artifact classifier (e.g., scFusion's bi-LSTM) on held-out scRNA-seq candidates where median ROC AUC and PR AUPR across datasets quantify discrimination of artifacts.

## When NOT to use

- Input is raw FASTQ or unaligned BAM — run scFusion's alignment and candidate-extraction pipeline first; ROC/AUPR scoring requires labeled fusion candidates
- Input fusion calls are unlabeled or have no artifact/real ground truth, so ROC AUC and PR AUPR cannot be computed
- Evaluating sensitivity for low-expression fusions or rare fusions in heterogeneous tumors — scFusion has known reduced sensitivity there, which these metrics alone will not capture

## Inputs

- Per-dataset held-out fusion candidate feature sets (artifact vs. real labels) from scFusion candidate extraction
- Trained bi-LSTM model (TensorFlow/Keras, e.g. tensorflow 2.8.0 / keras 2.8.0 / numpy 1.22.3)
- PCAWG fusion calls for external artifact-score validation
- Six publicly available scRNA-seq datasets (GEO accessions from refs 23, 30-34) as paired Smart-Seq FASTQ files (*_1.fastq, *_2.fastq)

## Outputs

- Per-dataset ROC AUC values
- Per-dataset precision-recall AUPR values
- Median ROC AUC (expected 0.884) and median PR AUPR (expected 0.913) across the six datasets
- Fraction of PCAWG fusions with artifact score < 0.5 (expected > 90%)
- model_evaluation_metrics.csv summarizing ROC AUC, PR AUPR, and PCAWG artifact-score results

## How to apply

After scFusion's pipeline (STAR >= 2.7.2d alignment of paired Smart-Seq FASTQs like *_1.fastq/*_2.fastq to hg19, samtools/bedtools candidate extraction, pyensembl annotation with 'gene_name' plus 'gene_type'/'gene_biotype', and the hg19 UCSC mappability filter), build and train the bi-LSTM model in TensorFlow/Keras to predict artifact vs. real fusions. Score held-out fusion candidates independently for each of the six public scRNA-seq datasets, then compute ROC AUC and precision-recall AUPR per dataset and report their medians (expected: 0.884 ROC AUC, 0.913 PR AUPR). As an additional sanity check, apply the trained model to PCAWG fusion calls and verify that >90% receive artifact scores < 0.5. Write per-dataset ROC AUC, PR AUPR, and the PCAWG artifact-score summary to model_evaluation_metrics.csv. Turning the deep-learning model off is a useful ablation because scFusion's precision drops without it, confirming the metric measures the model's contribution.

## Related tools

- **scFusion** (Pipeline producing the labeled chimeric-fusion candidates and deep-learning artifact scores that are evaluated with ROC AUC and PR AUPR) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligns paired Smart-Seq single-cell FASTQ files to hg19 (>= 2.7.2d) as the first step generating candidates to be scored) — https://github.com/alexdobin/STAR
- **TensorFlow** (Trains the bi-LSTM artifact-scoring model whose held-out predictions yield ROC AUC and PR AUPR)
- **pyensembl** (Annotates fusion candidates with gene_name and gene_type/gene_biotype fields to build model features) — https://github.com/openvax/pyensembl

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Median ROC AUC across the six public scRNA-seq datasets equals 0.884
- Median precision-recall AUPR across the six datasets equals 0.913
- Fraction of PCAWG fusions with artifact score < 0.5 exceeds 90%
- model_evaluation_metrics.csv exists and contains per-dataset ROC AUC, PR AUPR, and PCAWG artifact-score summary
- Ablation check: disabling the deep-learning model reduces scFusion's precision, confirming the metrics reflect the bi-LSTM's contribution

## Limitations

- The deep-learning model is trained on publicly available scRNA-seq artifacts; performance may not transfer to datasets with different artifact profiles
- scFusion has reduced sensitivity for fusions with low expression, common to all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples
- Omitting the hg19 mappability file (4-column UCSC file) disables the mappability filter, changing the candidate set being scored

## Evidence

- [results] Median AUC/AUPR benchmark targets: [evidence span withheld — anchor status: not_found]
- [intro] Model purpose: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [discussion] Ablation signal: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [readme] Deep-learning stack versions: "tensorflow, keras, and numpy (version 2.8.0, 2.8.0, and 1.22.3, respectively)"
- [methods] Mappability filter dependency: "If you do not provide this file, scFusion will turn off the mappability filter."
