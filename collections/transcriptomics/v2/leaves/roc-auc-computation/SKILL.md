---
name: roc-auc-computation
description: Use when benchmarking a splicing-status predictor (e.g., Forseti splice_prob)
  with ROC AUC against trivial baselines on scRNA-seq reads carrying ground-truth
  spliced/unspliced labels.
when_to_use_negative:
- When the goal is to run or evaluate Forseti's own prediction model rather than compare
  it against trivial predictors — use the forseti_predictor workflow and the forseti-experiments
  Nextflow evaluation step instead
- When reads lack ground-truth spliced/unspliced labels (e.g., ambiguous 'A' reads
  with no verified status), since AUC cannot be computed without labels
- When probability calibration rather than ranking quality is the question — AUC is
  rank-based and cannot assess calibration of splice_prob values
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Forseti
  role: Provides the splice_prob predictions and forseti_dict records whose ROC AUC
    is computed and compared to the trivial baselines
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Implements the three trivial baselines and computes the ROC AUC in the evaluation
    script
  repo: https://github.com/COMBINE-lab/forseti
- name: samtools
  role: Filters the reads that uniquely map to the genome, defining the labeled evaluation
    read set used for ROC analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:d008bcf0676c525668bb25aa71fbe12b482705d42ab5d532840fb1d705bbf176
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# roc-auc-computation

## Summary

Compute ROC AUC for splicing-status prediction of scRNA-seq reads on a labeled evaluation set of uniquely mapped reads, including the trivial 'All spliced', 'All unspliced', and 'All random' baselines that floor at AUC = 0.5 (as in Forseti's Fig. 3 comparison).

## When to use

Use when benchmarking a splicing-status predictor (e.g., Forseti splice_prob) with ROC AUC against trivial baselines on scRNA-seq reads carrying ground-truth spliced/unspliced labels.

## When NOT to use

- When the goal is to run or evaluate Forseti's own prediction model rather than compare it against trivial predictors — use the forseti_predictor workflow and the forseti-experiments Nextflow evaluation step instead
- When reads lack ground-truth spliced/unspliced labels (e.g., ambiguous 'A' reads with no verified status), since AUC cannot be computed without labels
- When probability calibration rather than ranking quality is the question — AUC is rank-based and cannot assess calibration of splice_prob values

## Inputs

- Uniquely genome-mapped scRNA-seq reads (BAM filtered with samtools, e.g., PBMC1k v3 unique mappings)
- Ground-truth splicing-status labels (spliced/unspliced) per read
- Forseti prediction records (forseti_dict) supplying per-read splice_prob for the comparison read set

## Outputs

- baseline_auc_results.csv with the three trivial-baseline AUC values
- Per-baseline ROC curves / AUC scores on the identical evaluation read set

## How to apply

Load the evaluation read set: uniquely genome-mapped scRNA-seq reads (filtered with samtools, e.g., the PBMC1k v3 reads) with ground-truth splicing-status labels drawn from the forseti_dict prediction records. Implement the 'All spliced' baseline by assigning splice_prob = 1.0 to every read, and 'All unspliced' by assigning splice_prob = 0.0; compute the ROC AUC of each. Implement the 'All random' baseline per the shuffle scheme, drawing splice_prob uniformly from [0, 0.5] for reads labeled unspliced and from [0.5, 1] for reads labeled spliced. Evaluate all baselines on the identical read set used for the model under test (Forseti's Fig. 3 ROC analysis) and confirm that all three trivial predictors attain an AUC of exactly 0.5, demonstrating no trivial predictor approaches Forseti's performance (mean AUC 0.92 simulated / 0.88 experimental). Write the three AUC values to baseline_auc_results.csv.

## Related tools

- **Forseti** (Provides the splice_prob predictions and forseti_dict records whose ROC AUC is computed and compared to the trivial baselines) — https://github.com/COMBINE-lab/forseti
- **Python** (Implements the three trivial baselines and computes the ROC AUC in the evaluation script) — https://github.com/COMBINE-lab/forseti
- **samtools** (Filters the reads that uniquely map to the genome, defining the labeled evaluation read set used for ROC analysis)

## Evaluation signals

- All three trivial baselines ('All spliced' = 1, 'All unspliced' = 0, 'All random' shuffled) return an AUC of exactly 0.5 on the same read set
- The evaluation read set matches the Forseti Fig. 3 read set (uniquely mapped PBMC1k reads with ground-truth labels), so AUCs are directly comparable
- Forseti's AUC on the same set substantially exceeds 0.5 (paper reports mean AUC 0.92 on simulated and 0.88 on experimental data)
- baseline_auc_results.csv contains exactly three AUC values, one per trivial baseline

## Limitations

- The 'All random' baseline AUC depends on the shuffle scheme definition (uniform [0, 0.5] for unspliced-labeled vs [0.5, 1] for spliced-labeled reads); other random schemes are not covered
- Baselines are a sanity floor only; they are not informative predictors and cannot substitute for the mechanistic model
- Evaluation is restricted to uniquely mapped reads; multi-mapped reads and fundamentally ambiguous reads fall outside this comparison

## Evidence

- [abstract] Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data.: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [other] All three trivial baseline models attain an AUC of exactly 0.5: [evidence span withheld — anchor status: not_found]
- [methods] samtools filters uniquely mapped reads: "Then, we use samtools to filter the reads have unique map to the genome."
- [readme] Evaluation script outputs included in workflow_output: [evidence span withheld — anchor status: external_doc]
