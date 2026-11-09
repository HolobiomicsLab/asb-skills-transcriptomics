---
name: splicing-prediction-benchmark-evaluation
description: 'Use when validating a splicing-status predictor for scRNA-seq reads:
  you need negative-control baselines (constant or random splice_prob) to demonstrate
  that model AUC reflects genuine prediction rather than chance.'
when_to_use_negative:
- When you already have Forseti's own prediction scores and only want its reported
  AUC (0.92 simulated, 0.88 experimental) — this skill builds the negative-control
  baselines, not the model itself
- When the input reads lack ground-truth splicing-status labels (e.g. unannotated
  alignments); AUC cannot be computed without labels
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Forseti
  role: Supplies the splice_prob prediction records (forseti_dict) that define the
    evaluation read set and ground-truth labels against which baselines are scored
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Used to implement the constant (1.0 / 0.0) and shuffled random baseline probabilities
    and to compute ROC AUC on the shared read set
  repo: https://github.com/COMBINE-lab/forseti
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:0ffd9179dc4012cefa5c761056c3d6d3213288991b98096547ee335639ff202d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# splicing-prediction-benchmark-evaluation

## Summary

Reproduces the trivial-baseline ROC comparison from the Forseti paper (Fig. 3), in which 'All spliced', 'All unspliced', and 'All random' predictors are evaluated on the same uniquely mapped PBMC1k scRNA-seq read set to confirm all attain AUC = 0.5. This establishes that no degenerate predictor approaches Forseti's reported performance.

## When to use

Use when validating a splicing-status predictor for scRNA-seq reads: you need negative-control baselines (constant or random splice_prob) to demonstrate that model AUC reflects genuine prediction rather than chance.

## When NOT to use

- When you already have Forseti's own prediction scores and only want its reported AUC (0.92 simulated, 0.88 experimental) — this skill builds the negative-control baselines, not the model itself
- When the input reads lack ground-truth splicing-status labels (e.g. unannotated alignments); AUC cannot be computed without labels

## Inputs

- Uniquely genome-mapped scRNA-seq reads (BAM filtered via samtools NH:1 flag) from PBMC1k (v3) healthy donor sample
- Ground-truth splicing-status labels per read (spliced / unspliced), drawn from Forseti forseti_dict prediction records

## Outputs

- baseline_auc_results.csv containing AUC values for 'All spliced', 'All unspliced', and 'All random' baselines (expected 0.5, 0.5, 0.5)
- ROC curve data comparable to the baseline traces shown in Fig. 3

## How to apply

Load the evaluation read set — uniquely genome-mapped PBMC1k (v3) scRNA-seq reads with ground-truth splicing-status labels, extracted from the forseti_dict prediction records produced after aligning reads to the spliceu index with STAR. Implement three baselines on the identical read set: 'All spliced' assigns splice_prob = 1.0 to every read; 'All unspliced' assigns splice_prob = 0.0 to every read; 'All random' draws splice_prob uniformly from [0, 0.5] for reads labeled unspliced and from [0.5, 1] for reads labeled spliced, per the shuffle scheme. Compute ROC AUC for each baseline with a standard ROC routine in Python and verify all three equal 0.5, matching Fig. 3 of the Forseti paper, then write the three AUC values to baseline_auc_results.csv.

## Related tools

- **Forseti** (Supplies the splice_prob prediction records (forseti_dict) that define the evaluation read set and ground-truth labels against which baselines are scored) — https://github.com/COMBINE-lab/forseti
- **Python** (Used to implement the constant (1.0 / 0.0) and shuffled random baseline probabilities and to compute ROC AUC on the shared read set) — https://github.com/COMBINE-lab/forseti

## Examples

```
from sklearn.metrics import roc_auc_score; auc = roc_auc_score(y_true, [1.0]*len(y_true))  # 'All spliced' baseline; expect AUC == 0.5
```

## Evaluation signals

- All three baseline AUC values are exactly 0.5 on the identical evaluation read set
- Baselines are computed on the same read set as Forseti (uniquely mapped PBMC1k reads with ground-truth splicing labels) so the comparison is apples-to-apples
- 'All random' uses the specified shuffle scheme: uniform [0, 0.5] probabilities for unspliced-labeled reads and [0.5, 1] for spliced-labeled reads
- Results are written to baseline_auc_results.csv with one AUC value per baseline
- No baseline AUC approaches Forseti's reported performance (mean AUC 0.92 simulated / 0.88 experimental), confirming no trivial predictor mimics the model

## Limitations

- Forseti, as a mechanistic model, cannot distinguish fundamentally ambiguous reads (polyA-tail priming with read2 within the terminal exon vs. exon-originating cDNA fragments); baselines therefore do not test performance on this irreducible ambiguity
- Baselines only confirm non-informativeness on the specific evaluated read set; they do not establish a lower-bound calibration of splice_prob values across other datasets

## Evidence

- [abstract] Forseti AUC on simulated and experimental data: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [methods] Uniquely mapped read filtering for evaluation: "we use samtools to filter the reads have unique map to the genome."
- [methods] Ambiguous reads definition relevant to predictor scope: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts."
- [readme] Evaluation script outputs available in forseti-experiments: [evidence span withheld — anchor status: external_doc]
