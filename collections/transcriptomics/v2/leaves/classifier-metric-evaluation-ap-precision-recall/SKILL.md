---
name: classifier-metric-evaluation-ap-precision-recall
description: Use when a trained binary classifier must be scored on held-out labels
  and compared across ablated feature-set conditions (e.g., multi-omics vs DNA-only
  vs RNA-only) using AP, precision, recall, and F-measure.
when_to_use_negative:
- The model has not been trained and saved (Mode 2 requires a built model and scale
  file before metrics can be computed)
- No gold-standard polyA site labels exist for the test split, so no supervised metric
  can be computed
- The task is ranking or differential APA quantitation (e.g., TAPAS-style 3' UTR shortening/lengthening),
  not binary polyA-site classification performance
edam_operation: http://edamontology.org/operation_3918
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3512
tools:
- name: aptardi
  role: biLSTM classifier whose Mode 2 models are trained per condition (--c seed,
    --e model name, --k scale name) and scored on the test split
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons
    define the feature windows over which predictions are evaluated
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:b6b1e6bb2361154ac386a5183096d0fc1db50ba5db1246e283f1c5926118db39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3918
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Classifier metric evaluation (AP, precision, recall, F-measure)

## Summary

Evaluates a binary classifier such as aptardi's biLSTM polyA-site predictor on a held-out test split against gold-standard labels, using average precision (AP) plus precision, recall, and F-measure to compare feature-set variants.

## When to use

Use when a trained binary classifier must be scored on held-out labels and compared across ablated feature-set conditions (e.g., multi-omics vs DNA-only vs RNA-only) using AP, precision, recall, and F-measure.

## When NOT to use

- The model has not been trained and saved (Mode 2 requires a built model and scale file before metrics can be computed)
- No gold-standard polyA site labels exist for the test split, so no supervised metric can be computed
- The task is ranking or differential APA quantitation (e.g., TAPAS-style 3' UTR shortening/lengthening), not binary polyA-site classification performance

## Inputs

- Trained model file (Keras HDF5, e.g. name_model.hdf5) and matching scale file (pickle, name_scale.pk) per feature-set condition
- Held-out test split of binned features (100-base DNA-sequence and RNA-Seq coverage bins over 3' terminal exons)
- Gold-standard polyA site labels (tab-separated polya_sites.bed with 0-based column indices 0,4,1 for chromosome, strand, site)

## Outputs

- Per-condition test-set average precision, precision, recall, and F-measure
- Comparison table of metrics across feature-set variants (multi-omics, DNA-only, RNA-only, PAS ablations)

## How to apply

Split gold-standard-labeled data into train and held-out test sets; train each model variant (here, aptardi Mode 2 biLSTM trained on Human Brain Reference RNA with bin size 100 and up to 300 bins per 3' terminal exon) with a fixed seed via --c so runs are reproducible. Run inference on the test split for each condition and score predicted polyA-site bins against the gold-standard polyA sites (loaded with 0-based chromosome, strand, site column indices given to --l). Compute average precision (area under the precision-recall curve) as the headline metric, plus precision, recall, and F-measure, per condition. Judge correctness by comparing metrics against the reported reference values: multi-omics AP = 0.58 and F = 0.51, DNA-only AP = 0.41, RNA-only AP = 0.44, no-canonical-PAS AP = 0.52, canonical-PAS-only AP = 0.54. Similarity of AP between training and test sets is treated as evidence the model is not overfitting.

## Related tools

- **aptardi** (biLSTM classifier whose Mode 2 models are trained per condition (--c seed, --e model name, --k scale name) and scored on the test split) — https://github.com/luskry/aptardi
- **StringTie** (reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons define the feature windows over which predictions are evaluated)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --m --s polya_sites.bed --l 0,4,1 --c 42 --e name_model.hdf5 --k name_scale.pk --o output_dir
```

## Evaluation signals

- Test-set AP matches reported values: multi-omics AP = 0.58 (F = 0.51), DNA-only AP = 0.41, RNA-only AP = 0.44, no-canonical-PAS AP = 0.52, canonical-PAS-only AP = 0.54
- Multi-omics AP exceeds both single-omics ablations, confirming the combined-feature claim
- Saved artifacts load correctly: name_model.hdf5 as a Keras HDF5 model and name_scale.pk as a pickle
- Training-set and test-set AP estimates are similar, consistent with the reported non-overfitting finding
- Fixed-seed re-runs (--c <seed>) reproduce the same metrics

## Limitations

- aptardi's AP is roughly twice a standard transcriptome assembler's but absolute AP (~0.58) remains moderate; recall is improved over three-fold relative to an assembler
- Metrics are against 3'-sequencing-derived gold standards; PolyA-Seq sites sequenced at much greater depth may be undetectable, deflating recall
- The HBR-trained model performed modestly worse on BNLx and SHR data sets (possibly due to unstranded RNA-Seq or species differences), so metric portability across samples is not guaranteed

## Evidence

- [results] AP multi-omics vs single-omics: "The average precision of the aptardi model is twice that of a standard transcriptome assembler."
- [abstract] recall improvement: "the recall of the aptardi model (the proportion of true polyadenylation sites detected by the algorithm) is improved by over three-fold"
- [discussion] non-overfitting check: "the similarity of the AP estimates between training and testing sets demonstrate that aptardi is not prone to overfitting"
- [readme] Mode 2 training invocation: "--m/-m <machine learning mode>	Enables Mode 2, building your own model, requires reliable genomic locations of polyA sites as the gold standard labels to train model"
- [readme] seed reproducibility: "--c/-c <int>			Set seed for reproducibly building model"
