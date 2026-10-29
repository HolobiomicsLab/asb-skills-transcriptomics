---
name: abc-model-ablation-benchmarking
description: Use when you need to quantify, by ablation, how much AUPRC improvement
  each individual ABC-model variant (adapted activity -q True, all-TSS gene window
  -i all_tss) contributes over the regular ABC baseline on perturbation-validated
  enhancer–gene interaction data.
when_to_use_negative:
- When you want the combined gABC performance rather than per-adaptation attribution
  — the ablation isolates single variants and cannot describe their joint effect
- When ground-truth enhancer–gene labels are not perturbation-based (e.g. no CRISPRi
  data) — AUPRC gains against such labels are not comparable to the reported values
- When you only need TF affinity summarisation on gene level — that is STARE's second,
  independent task, not the ABC-score ablation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: STARE / stare-abc
  role: Scores enhancer–gene interactions with the regular and adapted ABC model;
    the -q flag toggles adapted activity and the -i flag selects 5_tss vs all_tss
    gene assignment
  repo: https://github.com/schulzlab/STARE
- name: pROC
  role: Computes precision–recall/ROC curves and AUPRC for scored interactions against
    CRISPRi ground-truth labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:34860547c44c8ad33bfd36c30e282283be15ca942d43c6d38d7081380fc36fd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - STARE / stare-abc
  - pROC
  - stare-abc
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# abc-model-ablation-benchmarking

## Summary

Isolates the two gABC adaptations of the Activity-By-Contact model — gene-specific enhancer activity and inclusion of all annotated TSSs — and benchmarks each individually against the regular ABC score on CRISPRi validation datasets using AUPRC gains.

## When to use

Use when you need to quantify, by ablation, how much AUPRC improvement each individual ABC-model variant (adapted activity -q True, all-TSS gene window -i all_tss) contributes over the regular ABC baseline on perturbation-validated enhancer–gene interaction data.

## When NOT to use

- When you want the combined gABC performance rather than per-adaptation attribution — the ablation isolates single variants and cannot describe their joint effect
- When ground-truth enhancer–gene labels are not perturbation-based (e.g. no CRISPRi data) — AUPRC gains against such labels are not comparable to the reported values
- When you only need TF affinity summarisation on gene level — that is STARE's second, independent task, not the ABC-score ablation

## Inputs

- Three CRISPRi validation datasets (enhancer–gene perturbation ground-truth labels)
- Enhancer activity assay signal (e.g. ATAC-seq/DNase accessibility) over candidate regions
- Hi-C contact matrix (e.g. Fulco et al. 2019 average hg19 matrix average_hic.v2.191020)
- Gene annotation in GTF format (e.g. Gencode) with 5' and all annotated TSSs
- Candidate regions in BED format (--bed_file)

## Outputs

- gabc_ablation_auprc.csv with per-dataset and averaged per-adaptation AUPRC gains (~0.026 adapted activity, ~0.053 all TSSs)
- Scored enhancer–gene interaction files for each variant (baseline, -q True, -i all_tss) per dataset
- Precision–recall curves and AUPRC values computed with pROC

## How to apply

Score enhancer–gene interactions on each of the three CRISPRi validation datasets three times with STARE_ABCpp: (1) regular ABC baseline with window -w 5MB, cutoff -t 0.02, adapted activity disabled (-q False) and 5' TSS only (-i 5_tss); (2) only the adapted-activity adaptation enabled (-q True, -i 5_tss); (3) only the all-TSS adaptation enabled (-q False, -i all_tss). For each scored interaction file, compute precision–recall curves and AUPRC against the CRISPRi perturbation ground-truth labels with the R package pROC. Compute per-dataset AUPRC gains of each adaptation over the baseline, then average across the three datasets, expecting ~0.026 for adapted activity alone and ~0.053 for all annotated TSSs alone. The rationale is that the paper claims the ABC score gains accuracy from both the contact-adapted enhancer activity and considering all annotated TSSs, so a controlled single-variant-on/variant-off comparison attributes the gain to each adaptation separately.

## Related tools

- **STARE / stare-abc** (Scores enhancer–gene interactions with the regular and adapted ABC model; the -q flag toggles adapted activity and the -i flag selects 5_tss vs all_tss gene assignment) — https://github.com/schulzlab/STARE
- **pROC** (Computes precision–recall/ROC curves and AUPRC for scored interactions against CRISPRi ground-truth labels)

## Examples

```
STARE_ABCpp -w 5MB -t 0.02 -q False -i 5_tss   # baseline; rerun with -q True (adapted activity only) and with -i all_tss (all-TSS only), then compute AUPRC with pROC
```

## Evaluation signals

- gabc_ablation_auprc.csv contains both per-adaptation average AUPRC gains matching the reported values (≈0.026 for adapted activity, ≈0.053 for all TSSs) within reproducibility tolerance
- Six variant-by-dataset scored interaction files exist (3 datasets × 3 scoring runs: baseline, -q True, -i all_tss)
- Each AUPRC is computed against the same CRISPRi perturbation ground-truth labels per dataset so gains are baseline-consistent
- Per-dataset gains are recorded in the CSV before averaging, not only the final averaged values
- All baseline runs use identical parameters (-w 5MB, -t 0.02, -q False, -i 5_tss) so variant differences are attributable only to the toggled adaptation

## Limitations

- Results depend on the CRISPRi validation datasets and the Hi-C matrix used (e.g. Fulco et al. 2019 average hg19 matrix); different contact sources may shift the AUPRC gains
- The ablation only measures isolated effects — interactions between the two adaptations (adapted activity + all TSSs) are not captured by this design
- The adapted activity adjustment uses a window around candidate regions in which genes are considered, so window choices affect the gains

## Evidence

- [abstract] We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes: "We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes and also by"
- [methods] Adapted activity adjustment window: "Size of the window around your candidate regions in which genes are considered for the adapted activity adjustment"
- [methods] TSS window parameter: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [supplementary] pROC citation: "pROC: An open-source package for r and s+ to analyze and compare ROC curves"
- [readme] STARE framework description from README: "STARE is a framework that combines two tasks, which can also work independently:"
