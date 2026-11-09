---
name: roc-auc-evaluation-of-read-classification
description: Use when Forseti (or a similar per-read splicing-status classifier) has
  produced spliced/unspliced probabilities on reads with known ground-truth labels,
  and you need ROC/AUC to quantify predictive accuracy.
when_to_use_negative:
- Reads still labeled 'A' (ambiguous) in the simulated setting — the reported AUCs
  apply to retained non-ambiguous reads, so including ambiguous reads with no ground
  truth would bias the estimate
- Datasets without definitive ground-truth splicing labels (e.g. single-end data without
  the paired-end read-1 labeling rules e∧e′, e·e′, i, i·e) — AUC cannot be computed
  without gold-standard labels
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Forseti
  role: Produces the per-read splicing-status predictions that are scored by ROC/AUC;
    evaluation uses the forseti_predictor master function output
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Used to parse forseti_dict, filter reads, compute ROC curves and AUC scores,
    and write forseti_eval_auc_results.csv
- name: bedtools
  role: bedtools intersect (-f 1.0 -wo -bed against exon_by_tx.bed) identifies fully
    exonic reads used to derive definitive paired-end ground-truth labels for experimental
    evaluation
- name: R
  role: Supports plotting/analysis of ROC curves and AUC comparisons as in Fig. 3c-d
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:df12e6a51237148666388ec1ce2965616a5dafec6872a2c543ccbf7cb7e736b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - R
  - bedtools
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# roc-auc-evaluation-of-read-classification

## Summary

Evaluate Forseti's per-read spliced/unspliced splicing-status predictions by computing ROC curves and AUC scores against definitive ground-truth labels from simulated or paired-end-labeled experimental read sets.

## When to use

Use when Forseti (or a similar per-read splicing-status classifier) has produced spliced/unspliced probabilities on reads with known ground-truth labels, and you need ROC/AUC to quantify predictive accuracy.

## When NOT to use

- Reads still labeled 'A' (ambiguous) in the simulated setting — the reported AUCs apply to retained non-ambiguous reads, so including ambiguous reads with no ground truth would bias the estimate
- Datasets without definitive ground-truth splicing labels (e.g. single-end data without the paired-end read-1 labeling rules e∧e′, e·e′, i, i·e) — AUC cannot be computed without gold-standard labels

## Inputs

- forseti_dict returned by forseti.modules.prediction_model.forseti_predictor with per-read splicing_status and spliced/unspliced probabilities
- Simulated evaluation read sets (one with and one without fundamental ambiguity) with ground-truth splicing labels
- Experimental evaluation read sets (mouse and human scRNA-seq) with definitive labels from paired-end read-1 alignment
- exon_by_tx.bed and toTranscriptome BAM used to construct the evaluation read sets

## Outputs

- ROC curves for each evaluation read set
- Per-set AUC scores (expected: 0.92 mean on simulated data; 0.89 mouse and 0.85 human on experimental data)
- forseti_eval_auc_results.csv summarizing evaluation AUCs

## How to apply

After running forseti_predictor on the toTranscriptome BAM (with the uniquely-mapped read-name list, spliceu.fa, t2g_3col.tsv, and the pre-trained spline_model.pkl and mlp_pa6with1mm.pkl models; snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12), parse the returned forseti_dict to obtain each read's final splicing_status and splicing probability. For simulated evaluation, retain all non-ambiguous reads (discard reads whose final splicing_status remains 'A', i.e. compatible with both spliced and unspliced transcripts) and compute ROC curves and AUCs on both simulated read sets (with and without fundamental ambiguity), expecting mean AUC ~0.92 (range 0.85-0.93). For experimental evaluation, compare predicted probabilities against definitive labels assigned from paired-end alignments of read 1 using the exon containment patterns e∧e′, e·e′, i, i·e (computed with bedtools intersect -f 1.0 -wo -bed against exon_by_tx.bed), reproducing AUCs of 0.89 (mouse) and 0.85 (human), mean 0.88. Write results (e.g. forseti_eval_auc_results.csv) and validate that computed AUCs fall within the reported ranges.

## Related tools

- **Forseti** (Produces the per-read splicing-status predictions that are scored by ROC/AUC; evaluation uses the forseti_predictor master function output) — https://github.com/COMBINE-lab/forseti
- **Python** (Used to parse forseti_dict, filter reads, compute ROC curves and AUC scores, and write forseti_eval_auc_results.csv)
- **bedtools** (bedtools intersect (-f 1.0 -wo -bed against exon_by_tx.bed) identifies fully exonic reads used to derive definitive paired-end ground-truth labels for experimental evaluation)
- **R** (Supports plotting/analysis of ROC curves and AUC comparisons as in Fig. 3c-d)

## Examples

```
from forseti.modules.prediction_model import forseti_predictor; forseti_dict = forseti_predictor(toTranscriptome_bam, unimap_names, 'spliceu.fa', 't2g_3col.tsv', 'spline_model.pkl', 'mlp_pa6with1mm.pkl', snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12)
```

## Evaluation signals

- Computed simulated-data AUCs fall within the reported range 0.85-0.93 with mean ~0.92 (README-consistent reproduction target mean 0.90)
- Experimental AUCs reproduce the reported values of 0.89 (mouse) and 0.85 (human), mean 0.88, as in Fig. 3c-d
- All reads whose final splicing_status remains 'A' (ambiguous) have been excluded from the simulated evaluation
- forseti_eval_auc_results.csv is produced and populated with per-set ROC/AUC values
- Every evaluated read has a definitive ground-truth spliced/unspliced label (simulated truth or paired-end read-1 containment pattern)

## Limitations

- Forseti is a mechanistic model and cannot distinguish fundamentally ambiguous reads (polyA tail priming with read2 in the terminal exon vs. cDNA fragments arising from an exon), so such reads must be excluded in simulation evaluation
- Reported AUCs of 0.92 (simulated) and 0.88 (experimental) are means; individual evaluation sets range from 0.85 to 0.93 on simulated data, so per-set AUCs will vary
- Experimental ground truth depends on paired-end read-1 labeling rules, so single-end datasets cannot receive definitive labels by this scheme

## Evidence

- [abstract] Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data.: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [methods] Ambiguous reads are compatible with both transcripts: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads"
- [results] Experimental AUCs across species: [evidence span withheld — anchor status: not_found]
- [other] Paired-end exon containment labeling: [evidence span withheld — anchor status: not_found]
