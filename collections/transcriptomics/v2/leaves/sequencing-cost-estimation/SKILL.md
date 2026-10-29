---
name: sequencing-cost-estimation
description: Use when a semi-profiling workflow needs a defensible dollar figure for
  sequencing only representative samples versus the whole cohort, or when judging
  whether the 1/10–1/3 cost-savings target of scSemiProfiler was achieved.
when_to_use_negative:
- The cohort will be fully single-cell sequenced regardless — cost savings from representative
  selection are irrelevant
- Sequencing rates differ substantially from the 2023 McGill Genome Centre / costpercell
  basis and accurate local pricing is required
- The study targets modalities beyond scRNA-seq (e.g., proteomics), which scSemiProfiler
  does not yet support
tools:
- name: scSemiProfiler
  role: Provides the estimate_cost utility (scSemiProfiler.utils.estimate_cost) used
    to compute sequencing costs of the semi-profiled cohort and the full single-cell
    baseline
  repo: github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:bbe7c860c190eed33b6346a717ba4037b43dd1aee5f6d31eae7e991fdc20c203
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSemiProfiler
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sequencing-cost-estimation

## Summary

Estimate and compare single-cell sequencing costs for semi-profiled versus fully-profiled cohorts using scSemiProfiler's estimate_cost utility, grounded in per-cell sequencing rates from the McGill Genome Centre and costpercell (2023). It quantifies the cost savings of active-learning-based representative selection.

## When to use

Use when a semi-profiling workflow needs a defensible dollar figure for sequencing only representative samples versus the whole cohort, or when judging whether the 1/10–1/3 cost-savings target of scSemiProfiler was achieved.

## When NOT to use

- The cohort will be fully single-cell sequenced regardless — cost savings from representative selection are irrelevant
- Sequencing rates differ substantially from the 2023 McGill Genome Centre / costpercell basis and accurate local pricing is required
- The study targets modalities beyond scRNA-seq (e.g., proteomics), which scSemiProfiler does not yet support

## Inputs

- Preprocessed bulk sequencing data for the full cohort (e.g., 124-sample COVID-19 cohort)
- List of selected representative samples chosen by the active learning module
- Sequencing rate assumptions (McGill Genome Centre and costpercell rates as of 2023)

## Outputs

- cost_summary.csv with per-sample and total sequencing cost estimates
- Cost ratio of semi-profiling versus full single-cell sequencing (e.g., $62,640 / $186,000 = 33.7% for the COVID-19 example)

## How to apply

After assembling the semi-profiled cohort (semiprofiled_cohort.h5ad), call scSemiProfiler.utils.estimate_cost to compute the total cost of single-cell sequencing the selected representatives plus bulk sequencing of the full cohort, saving the result as cost_summary.csv. In the COVID-19 case (124 samples, 28 representatives), the estimated semi-profiling cost was $62,640 versus $186,000 for full single-cell sequencing — a ratio of 0.337 (66.3% savings). Cost rates are based on 2023 figures from the McGill Genome Centre and the costpercell resource. Correctness is judged by checking that the cost summary file exists and that the semi-profiling cost ratio falls in the expected ~1/10–1/3 band relative to full profiling.

## Related tools

- **scSemiProfiler** (Provides the estimate_cost utility (scSemiProfiler.utils.estimate_cost) used to compute sequencing costs of the semi-profiled cohort and the full single-cell baseline) — github.com/mcgilldinglab/scSemiProfiler

## Examples

```
import scSemiProfiler.utils as su; su.estimate_cost(...)  # writes cost_summary.csv for the 28-representative COVID-19 cohort
```

## Evaluation signals

- semiprofiled_cohort.h5ad exists before cost computation is run
- cost_summary.csv is produced by scSemiProfiler.utils.estimate_cost
- Total semi-profiling cost falls within $60,000–$66,000 for the COVID-19 benchmark (28 representatives, 124 samples)
- Cost ratio to the $186,000 full single-cell estimate lies in the 0.30–0.37 range (33.7% expected)
- Savings align with the advertised 1/10 to 1/3 of full sequencing cost (66.3% savings in the COVID-19 example)

## Limitations

- Cost estimates rely on 2023 rates from the McGill Genome Centre and costpercell; actual pricing varies by provider and time
- Savings of 1/10 to 1/3 cost apply to the semi-profiling scenario and depend on the number of representatives selected by active learning

## Evidence

- [readme] This AI-powered semi-profiling framework significantly reduces single-cell profiling costs for large cohorts (e.g., a 66.3% savings in the example COVID-19 study).: [evidence span withheld — anchor status: external_doc]
- [readme] Cost estimates are based on rates from the McGill Genome Centre and costpercell as of 2023.: [evidence span withheld — anchor status: external_doc]
- [readme] semi-profiling, which uses deep generative learning and active learning to generate a single-cell cohort with 1/10 to 1/3 sequencing cost: "semi-profiling, which uses deep generative learning and active learning to generate a single-cell cohort with 1/10 to 1/3 sequencing cost"
- [abstract] It provides a scalable, cost-effective solution for single-cell profiling, facilitating in-depth cellular investigation in various biological domains.: "It provides a scalable, cost-effective solution for single-cell profiling, facilitating in-depth cellular investigation in various biological domains."
