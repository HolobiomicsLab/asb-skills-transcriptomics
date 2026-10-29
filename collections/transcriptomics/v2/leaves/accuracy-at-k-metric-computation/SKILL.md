---
name: accuracy-at-k-metric-computation
description: Use when evaluating a cell type annotation or classification method that
  outputs ranked scores over a controlled vocabulary (e.g., Cell Ontology terms),
  and you need top-k correctness (Accuracy@3/Accuracy@5) rather than only AUROC.
when_to_use_negative:
- The classifier emits only a single hard label per cell (no ranked scores over Cell
  Ontology terms), so top-k correctness cannot be computed
- The task is distinguishing topologically identical unseen cell types (siblings sharing
  the same parents in the Cell Ontology graph) — a known OnClass limitation that distorts
  top-k comparisons
- The annotation target is free text rather than the controlled Cell Ontology vocabulary
  (controlled-vocabulary labels assumed; nlp_mapping disabled)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Produces pred_Y_seen and pred_Y_all scores over all Cell Ontology terms (use_normalize=True),
    which are ranked to compute Accuracy@3/Accuracy@5
  repo: https://github.com/wangshenguiuc/OnClass
- name: ACTINN
  role: Baseline neural-network classifier whose top-k accuracy is compared against
    OnClass on the same Muris droplet splits
  repo: https://github.com/mafeiyang/ACTINN
- name: Python
  role: Implementation language for the metric computation over prediction score matrices
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:10169d73de674d90ee8679471c524a3af388c6311bd9812c5e3da0b1659ea04c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - ACTINN
  - OnClass
  - Python
  - TensorFlow
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# accuracy-at-k-metric-computation

## Summary

Computation of Accuracy@3 and Accuracy@5 to benchmark cell type annotation methods: a prediction counts as correct when the true Cell Ontology label appears among the top-3 or top-5 ranked predicted terms. Used alongside AUROC over unseen terms to compare OnClass against ACTINN, SVM, LR, and reject-option baselines on Muris droplet splits.

## When to use

Use when evaluating a cell type annotation or classification method that outputs ranked scores over a controlled vocabulary (e.g., Cell Ontology terms), and you need top-k correctness (Accuracy@3/Accuracy@5) rather than only AUROC.

## When NOT to use

- The classifier emits only a single hard label per cell (no ranked scores over Cell Ontology terms), so top-k correctness cannot be computed
- The task is distinguishing topologically identical unseen cell types (siblings sharing the same parents in the Cell Ontology graph) — a known OnClass limitation that distorts top-k comparisons
- The annotation target is free text rather than the controlled Cell Ontology vocabulary (controlled-vocabulary labels assumed; nlp_mapping disabled)

## Inputs

- Ground-truth AnnData test labels under the 'cell_ontology_id' key (controlled Cell Ontology IDs)
- Ranked prediction score matrices from OnClass (pred_Y_all / pred_Y_seen) and from baseline classifiers (ACTINN, SVM, LR, and their reject-option variants) on the same test cells
- Test split specification with controlled proportion of unseen Cell Ontology terms (70% and 30% unseen runs)

## Outputs

- Accuracy@3 over test cells (OnClass target: 0.56 at 30% unseen terms)
- Accuracy@5 over test cells (OnClass target: 0.60 at 30% unseen terms)
- Per-method comparison table of Accuracy@k and unseen-term AUROC across the 30%/70% unseen splits

## How to apply

Run each annotator (OnClass via OnClassModel.Predict with use_normalize=True, plus baselines ACTINN, SVM, LR and reject variants SVM(reject), LR(reject), Cell BLAST(reject), DOC(reject)) on identical Muris droplet train/test splits with controlled unseen-term proportions (30% and 70% unseen). For OnClass, use pred_Y_all, which propagates scores to all Cell Ontology terms including unseen ones, so top-k retrieval can hit labels absent from training. For each test cell, rank the predicted term scores descendingly and mark a hit if the 'cell_ontology_id' ground-truth label is within the top 3 (Accuracy@3) or top 5 (Accuracy@5); average over cells. Accept the benchmark run when OnClass attains Accuracy@3 = 0.56 and Accuracy@5 = 0.60 at 30% unseen terms, and complement the metric with AUROC over unseen terms (OnClass 0.87 vs baselines below 0.67 at 70% unseen).

## Related tools

- **OnClass** (Produces pred_Y_seen and pred_Y_all scores over all Cell Ontology terms (use_normalize=True), which are ranked to compute Accuracy@3/Accuracy@5) — https://github.com/wangshenguiuc/OnClass
- **ACTINN** (Baseline neural-network classifier whose top-k accuracy is compared against OnClass on the same Muris droplet splits) — https://github.com/mafeiyang/ACTINN
- **Python** (Implementation language for the metric computation over prediction score matrices) — https://github.com/wangshenguiuc/OnClass

## Examples

```
pred_Y_seen, pred_Y_all = model.Predict(test_feature, use_normalize=True); top3 = np.argsort(-pred_Y_all, axis=1)[:, :3]; acc3 = np.mean([test_labels[i] in top3[i] for i in range(len(test_labels))])
```

## Evaluation signals

- Accuracy@3 at 30% unseen Cell Ontology terms equals approximately 0.56 and Accuracy@5 approximately 0.60 for OnClass on Muris droplet
- Every test cell contributes a top-k hit determined by ranking pred_Y_all scores, so the number of ranked candidates per cell exceeds 5 and includes unseen ontology terms
- Identical train/test splits (same controlled unseen-term proportions of 30% and 70%) are used for OnClass and every baseline (ACTINN, SVM, LR, reject variants) before comparing Accuracy@k
- Metric results are consistent with the companion unseen-term AUROC trend: OnClass 0.87 vs baselines below 0.67 at 70% unseen
- Ground-truth labels are read from the 'cell_ontology_id' AnnData key with nlp_mapping disabled, so metric denominators match the controlled-vocabulary test set exactly

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which can cap achievable Accuracy@k when correct answers are ontologically interchangeable
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so Accuracy@k is only meaningful over the terms the ontology covers
- Metric values are benchmark-specific (Muris droplet, 30%/70% unseen proportions) and should not be transferred to other datasets without rerunning the splits

## Evidence

- [full_text] Accuracy@3/Accuracy@5 targets on 30% unseen split: [evidence span withheld — anchor status: not_found]
- [discussion] Score propagation enables top-k retrieval of unseen terms: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [methods] Prediction step that yields the ranked scores used for Accuracy@k: [evidence span withheld — anchor status: not_found]
- [intro] Benchmark framing against baselines: "Through careful benchmarking, we found that our method outperformed existing methods at annotating unseen Cell Ontology terms"
- [readme] Baseline tool producing comparison predictions: [evidence span withheld — anchor status: external_doc]
