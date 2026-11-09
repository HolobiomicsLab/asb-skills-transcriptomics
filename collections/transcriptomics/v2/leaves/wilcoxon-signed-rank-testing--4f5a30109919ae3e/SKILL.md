---
name: wilcoxon-signed-rank-testing
description: Use when comparing a paired metric (e.g., recall of GTEx eQTL-supported
  enhancer-gene pairs) between two scoring methods measured on the same small set
  of units (four heart cell types plus K562) and a non-parametric paired significance
  test is needed.
when_to_use_negative:
- The samples are independent rather than paired across the same cell types — use
  an unpaired (Mann-Whitney) test instead
- More than two scoring methods must be compared simultaneously on the same units
  — a Friedman-type test or per-pair tests with correction are needed
- The comparison target is the interaction score distribution itself rather than a
  derived per-cell-type statistic like eQTL-pair recall
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: STARE / stare-abc
  role: Produces the regular ABC and adapted gABC interaction scores per cell type
    whose eQTL-pair recalls are compared by the Wilcoxon signed-rank test
  repo: https://github.com/schulzlab/STARE
- name: Cicero
  role: Co-accessibility-based baseline enhancer-gene interaction method for the same
    four heart cell types and K562, providing recall values as an additional comparison
    context
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:4d28951acb29dbe05d70bc8aeb47478db15fe9f79331b3552660581a4aa4e2ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Cicero
  - STARE
  - STARE / stare-abc
  - stare-abc
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# wilcoxon-signed-rank-testing

## Summary

Two-sided Wilcoxon signed-rank test used to compare per-cell-type eQTL-pair recall between the gABC (adapted-activity, all-TSS) and regular ABC enhancer-gene scoring methods, yielding the headline result P ≈ 0.0005 that gABC recovers significantly more eQTL-supported enhancer-gene pairs.

## When to use

Use when comparing a paired metric (e.g., recall of GTEx eQTL-supported enhancer-gene pairs) between two scoring methods measured on the same small set of units (four heart cell types plus K562) and a non-parametric paired significance test is needed.

## When NOT to use

- The samples are independent rather than paired across the same cell types — use an unpaired (Mann-Whitney) test instead
- More than two scoring methods must be compared simultaneously on the same units — a Friedman-type test or per-pair tests with correction are needed
- The comparison target is the interaction score distribution itself rather than a derived per-cell-type statistic like eQTL-pair recall

## Inputs

- Per-method, per-cell-type top-300,000 enhancer-gene interaction lists scored with STARE_ABCpp (regular ABC and adapted gABC configurations: -q True, -i all_tss)
- High-confidence GTEx eQTL-gene pairs fine-mapped with CAVIAR, CaVEMaN, and DAP-G
- Recall values per method per cell type (four heart cell types plus K562)

## Outputs

- Two-sided Wilcoxon signed-rank p-value comparing ABC vs gABC eQTL-pair recall (expected ≈ 0.0005)
- Per-cell-type recall comparison table written to eqtl_recall_comparison.tsv

## How to apply

First compute the paired observations: intersect each method's top-ranked interactions (top 300,000 per cell type from STARE_ABCpp output, run with -t 0 so all interactions within the 5 MB window around the 5' TSS are retained) with high-confidence GTEx eQTL-gene pairs fine-mapped by CAVIAR, CaVEMaN, and DAP-G, and record recall per method per cell type. Pair the recalls across the same cell types (four heart cell types plus K562 as baseline context alongside Cicero co-accessibility). Run the two-sided Wilcoxon signed-rank test on the paired ABC-vs-gABC recall differences and record the p-value. The comparison is considered successful if gABC recall is significantly higher than ABC, with Wilcoxon signed-rank P ≈ 0.0005, and the per-cell-type recall table is written to eqtl_recall_comparison.tsv. Because the sample is small and paired, the signed-rank test on per-cell-type differences is appropriate rather than a t-test on raw interaction counts.

## Related tools

- **STARE / stare-abc** (Produces the regular ABC and adapted gABC interaction scores per cell type whose eQTL-pair recalls are compared by the Wilcoxon signed-rank test) — https://github.com/schulzlab/STARE
- **Cicero** (Co-accessibility-based baseline enhancer-gene interaction method for the same four heart cell types and K562, providing recall values as an additional comparison context)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(abc_recall, gabc_recall, alternative='two-sided')
```

## Evaluation signals

- The recorded two-sided Wilcoxon signed-rank p-value is ≈ 0.0005 with gABC recall exceeding ABC recall
- The test is applied to paired per-cell-type recall values (four heart cell types plus K562), not to raw interaction counts
- Recall is computed consistently per fine-mapping method (CAVIAR, CaVEMaN, DAP-G) and the significance holds across all of them and all eQTL datasets
- The per-cell-type recall table eqtl_recall_comparison.tsv is produced as the test's supporting output
- Each method's interaction set is restricted to the top 300,000 scored interactions per cell type before intersection with eQTL pairs

## Limitations

- The paired sample is small (five cell types), so the test's power and generalisability are limited to this comparison
- The p-value summarises relative ranking of the two methods on eQTL-pair recall; it does not quantify per-cell-type effect sizes
- Ties or zero differences in per-cell-type recall between methods cannot contribute to the signed-rank statistic and should be checked before interpreting the result

## Evidence

- [results] The gABC interactions recovered significantly more eQTL-gene pairs than regular ABC interactions across all fine-mapping methods (CAVIAR, CaVEMaN, DAP-G): [evidence span withheld — anchor status: not_found]
- [results] at P-value ≈ 0.0005 (Wilcoxon signed-rank test): [evidence span withheld — anchor status: not_found]
- [readme] STARE is a framework that combines two tasks, which can also work independently: "STARE is a framework that combines two tasks, which can also work independently"
- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has: [evidence span withheld — anchor status: not_found]
