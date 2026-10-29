---
name: prec-recall-curve-benchmarking
description: Use when scored enhancer–gene interactions must be quantitatively compared
  across activity setups or models using an imbalanced CRISPR perturbation gold standard,
  where AUPRC is the appropriate metric.
when_to_use_negative:
- No CRISPR perturbation gold-standard enhancer–gene labels are available for the
  dataset, so no supervised AUPRC can be computed
- The goal is TF affinity/annotation analysis (e.g. TF affinities per gene via STARE+GAZE)
  rather than benchmarking enhancer–gene interaction predictions
- Predictions are balanced-class or ranked lists where ROC-AUC, not precision-recall,
  is the requested metric
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: STARE / stare-abc
  role: Produces the ABC-scored enhancer–gene interactions (with adapted activity
    and all_tss defaults) that are benchmarked via AUPRC
  repo: https://github.com/schulzlab/STARE
- name: pROC
  role: R package used to compute the AUPRC of scored interactions against the CRISPR
    gold-standard labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:a3b52bcf9850e4efb6c714a3b1d802f93f5ae7be2425c740c06033b0421aa08e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - STARE / stare-abc
  - pROC
  - stare-abc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# prec-recall-curve-benchmarking

## Summary

Benchmarks enhancer–gene interaction predictions by computing the area under the precision-recall curve (AUPRC) of STARE/stare-abc-scored interactions against CRISPR perturbation gold-standard labels, comparing single-assay (DNase-seq, H3K27ac ChIP-seq, ATAC-seq) enhancer-activity setups against the combined DNase-seq + H3K27ac baseline.

## When to use

Use when scored enhancer–gene interactions must be quantitatively compared across activity setups or models using an imbalanced CRISPR perturbation gold standard, where AUPRC is the appropriate metric.

## When NOT to use

- No CRISPR perturbation gold-standard enhancer–gene labels are available for the dataset, so no supervised AUPRC can be computed
- The goal is TF affinity/annotation analysis (e.g. TF affinities per gene via STARE+GAZE) rather than benchmarking enhancer–gene interaction predictions
- Predictions are balanced-class or ranked lists where ROC-AUC, not precision-recall, is the requested metric

## Inputs

- Candidate-enhancer bed file with numeric activity columns (DNase-seq, H3K27ac ChIP-seq, geometric mean, ATAC-seq; all values ≥ 0)
- Gene annotation in GTF format (e.g. Gencode)
- Normalized chromatin contact data directory with chosen bin size, or '-f false' for the distance-based contact estimate
- CRISPR perturbation gold-standard enhancer–gene labels (e.g. Gasperini dataset)
- STARE_ABCpp scored interaction outputs per activity setup

## Outputs

- AUPRC value per activity setup and per model configuration
- auprc_comparison.csv assembling the four single-assay AUPRCs and the combined DNase-seq + H3K27ac baseline AUPRC

## How to apply

Build candidate-enhancer bed files with separate activity columns (DNase-seq counts, H3K27ac ChIP-seq counts, their geometric mean, and ATAC-seq counts, all ≥ 0). For each activity setup, run STARE_ABCpp with the activity column selected via -n, a 5MB window (-w), contact data via -f <contact_data_dir> -k <bin_size> (or -f false for the distance-based contact estimate c = max{d, 5000}^-1), and cutoff -t 0.02, keeping gABC defaults -q True (adapted activity) and -i all_tss. In R, use pROC to compute the AUPRC of each setup's scored interactions against the CRISPR perturbation gold-standard enhancer–gene labels, then assemble the four single-assay AUPRCs alongside the combined DNase-seq + H3K27ac baseline into auprc_comparison.csv. The evaluation is valid if at least one single-assay AUPRC is comparable to the combined baseline, since the paper found the model comparably accurate with only one activity assay, with ATAC-seq alone even surpassing the DNase-seq + H3K27ac combination on the Gasperini dataset.

## Related tools

- **STARE / stare-abc** (Produces the ABC-scored enhancer–gene interactions (with adapted activity and all_tss defaults) that are benchmarked via AUPRC) — https://github.com/schulzlab/STARE
- **pROC** (R package used to compute the AUPRC of scored interactions against the CRISPR gold-standard labels)

## Examples

```
Rscript -e 'library(pROC); p <- pr.curve(-scores$gABC, weights.class0=labels$positive, curve=TRUE); cat(p$auc.integral)'
```

## Evaluation signals

- All activity values in the candidate-enhancer bed columns are ≥ 0
- Four AUPRCs (DNase-seq alone, H3K27ac alone, geometric mean, ATAC-seq alone) plus the combined DNase-seq + H3K27ac baseline AUPRC are assembled in auprc_comparison.csv
- At least one single-assay AUPRC is comparable to the combined two-assay baseline AUPRC
- ATAC-seq alone improves AUPRC over the DNase-seq + H3K27ac combination on the Gasperini dataset
- All runs use the same gABC defaults (-q True adapted activity, -i all_tss) and cutoff -t 0.02 so setups differ only in the activity column

## Limitations

- The benchmark depends on the availability and quality of CRISPR perturbation gold-standard labels (e.g. the Gasperini dataset); datasets without such labels cannot be AUPRC-evaluated
- The distance-based contact fallback (c = max{d, 5000}^-1) is only an approximation when no Hi-C/contact data are supplied
- Findings that single-assay activity suffices, and that ATAC-seq alone surpasses the combined setup, were demonstrated on the datasets analyzed (e.g. Gasperini) and may not transfer to every tissue or assay combination

## Evidence

- [abstract] Further, we show that the model is comparably accurate with only one assay to measure enhancer activity.: "Further, we show that the model is comparably accurate with only one assay to measure enhancer activity."
- [supplementary] pROC: An open-source package for r and s+ to analyze and compare ROC curves: "pROC: An open-source package for r and s+ to analyze and compare ROC curves"
- [methods] score enhancer-gene interactions with the Activity-By-Contact (ABC) model: "score enhancer-gene interactions with the Activity-By-Contact (ABC) model"
- [methods] Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
