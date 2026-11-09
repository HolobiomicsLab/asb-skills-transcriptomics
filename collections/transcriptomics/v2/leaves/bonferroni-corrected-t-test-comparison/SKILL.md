---
name: bonferroni-corrected-t-test-comparison
description: Use when comparing per-step removal (or depletion) rates of a spiked/known-positive
  class against the background class across replicates, and multiple filter steps
  require multiplicity correction.
when_to_use_negative:
- When only a single filtering step (not multiple) is being compared, so no Bonferroni
  correction over steps is needed
- When the positive class is not experimentally planted/tracked per step, so per-step
  fate of chimeras cannot be recorded
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: Pipeline whose per-step (TPM, CD-HIT-EST, TransRate-C) chimera-removal counts
    provide the data for the t test comparison
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Provides the TransRate-C filtering step whose chimera-vs-non-chimera removal
    rate is one of the compared conditions
- name: CD-HIT-EST
  role: Redundancy filter (95% identity) whose per-step removal rate is one of the
    compared conditions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:ba70f53305b691cf719e83b37b146c6b3215dfe74ad935a880d7601bfb8c9f82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# bonferroni-corrected-t-test-comparison

## Summary

Statistical validation procedure from the Bellerophon chimera-removal study: compare per-filter chimera removal percentages against non-chimera removal percentages across replicate spiked assemblies using an unpaired t test followed by Bonferroni correction.

## When to use

Use when comparing per-step removal (or depletion) rates of a spiked/known-positive class against the background class across replicates, and multiple filter steps require multiplicity correction.

## When NOT to use

- When only a single filtering step (not multiple) is being compared, so no Bonferroni correction over steps is needed
- When the positive class is not experimentally planted/tracked per step, so per-step fate of chimeras cannot be recorded

## Inputs

- Per-repeat counts of planted chimeras removed by each filter step (TPM, CD-HIT-EST, TransRate-C) from five spiked-assembly runs
- Per-repeat percentages of non-chimeric sequences removed by each corresponding filter step
- Number of planted chimeras per run (e.g., 500)

## Outputs

- Per-step chimera removal percentages with mean and standard deviation across repeats (e.g., 97.04% ± 0.61)
- Mean non-chimera removal percentages per step (e.g., 69.89% ± 0.03 overall; 60.77% at the TPM step)
- Unpaired t test statistics with Bonferroni-adjusted p values per filter step

## How to apply

Repeat the spiking experiment five times: plant 500 synthetic chimeras (random pairs from the H. subflexa RNA-Seq dataset concatenated with 30-70% overlap) into the de novo assembly, then run the Bellerophon pipeline with default filtering (TPM cut-off 1, CD-HIT-EST at 95% identity, TransRate filtering, default order TPM then CD-HIT-EST). For each filtering step (TPM, CD-HIT-EST, TransRate-C), track the fate of the 500 planted chimeras and record how many are removed, while computing the mean percentage of non-chimeric sequences removed by the corresponding step across the five repeats. For each step, compare the per-step chimera removal percentage against the mean non-chimera removal percentage using an unpaired t test, then apply a Bonferroni correction across the multiple filter steps tested. The comparison is judged significant when the corrected p value is small (in the source study, adjusted p = 5.83 × 10−6 for 97.04% ± 0.61 chimera removal vs 69.89% ± 0.03 for other sequences).

## Related tools

- **Bellerophon** (Pipeline whose per-step (TPM, CD-HIT-EST, TransRate-C) chimera-removal counts provide the data for the t test comparison) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Provides the TransRate-C filtering step whose chimera-vs-non-chimera removal rate is one of the compared conditions)
- **CD-HIT-EST** (Redundancy filter (95% identity) whose per-step removal rate is one of the compared conditions)

## Examples

```
t.test(chimera_pct, nonchimera_pct, alternative='greater'); p.adjust(p.values, method='bonferroni')
```

## Evaluation signals

- Overall chimera removal is approximately 97% (about 485 of 500 chimeras, 97.04% ± 0.61) versus roughly 70% removal of other sequences (69.89% ± 0.03)
- The TPM step alone discards about 96% of chimeras (95.72%) versus 60.77% of non-chimeric sequences
- The comparison yields a significant Bonferroni-adjusted p value (source study: adjusted p = 5.83 × 10−6)
- The experiment is repeated five times and per-step removal percentages are averaged over repeats before testing

## Limitations

- Bellerophon may erroneously remove contigs from alternative splicing that are misclassified as chimeras (uneven expression along the sequence can make TransRate flag them), which could bias the non-chimera removal baseline
- The result is specific to the default filtering order (TPM then CD-HIT-EST); custom order of filtering steps is disabled in this version
- Removal rates vary by dataset (between 40% and 91.9% of chimeras in other assemblies), so replicate-specific percentages may differ

## Evidence

- [other] Track the fate of the 500 planted chimeras and record how many are removed by each filtering step (TPM, CD-HIT-EST, TransRate-C).: [evidence span withheld — anchor status: not_found]
- [other] Compare the per-step chimera removal percentage against the mean non-chimera removal percentage with an unpaired t test followed by a Bonferroni correction.: [evidence span withheld — anchor status: not_found]
- [methods] Bellerophon filters the assembly using a TPM filter with a default cut-off score of 1.: [evidence span withheld — anchor status: not_found]
- [results] the TPM filtering step first removed 227 (8.5%) of the chimeras: "the TPM filtering step first removed 227 (8.5%) of the chimeras"
