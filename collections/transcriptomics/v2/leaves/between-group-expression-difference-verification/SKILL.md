---
name: between-group-expression-difference-verification
description: Use when single reference gene normalization (ERgene.normalizationdata)
  has been applied and you must show, via before/after boxplots and density plots,
  that the between-group spectral density difference (e.g., SHED vs salt injection)
  is reduced relative to raw data.
when_to_use_negative:
- The dataset is a large multi-gene screening task where you only need the top-20
  candidate reference gene list from ERgene.FindERG, not normalization or its validation
- You cannot demonstrate invariable expression of the chosen single reference gene
  — the article states single-gene normalization is not acceptable per MIQE guidelines
  in that case
- The computational depth for FindERG is set very large (≥3 and beyond), where efficiency
  declines and there may be no result to verify
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ERgene
  role: Provides FindERG to rank candidate reference genes and normalizationdata to
    standardize the data with a single internal reference gene whose effect the between-group
    comparison verifies
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Holds the expression data in the DataFrame format required by FindERG and
    normalizationdata
- name: Python
  role: Execution environment for running ERgene screening, normalization, and before/after
    density/boxplot verification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:6d793e3109fdfd0e47149e2cc60a345f1e386e4bc5acb79abc7ddfe2232a79b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# between-group-expression-difference-verification

## Summary

Verifies that normalization with an ERgene-screened candidate reference gene (e.g., Atp1a3) actually reduces the between-group difference in sample expression distributions, using before/after boxplots and spectral density plots (SHED-injected vs salt-injected aging mouse brain data).

## When to use

Use when single reference gene normalization (ERgene.normalizationdata) has been applied and you must show, via before/after boxplots and density plots, that the between-group spectral density difference (e.g., SHED vs salt injection) is reduced relative to raw data.

## When NOT to use

- The dataset is a large multi-gene screening task where you only need the top-20 candidate reference gene list from ERgene.FindERG, not normalization or its validation
- You cannot demonstrate invariable expression of the chosen single reference gene — the article states single-gene normalization is not acceptable per MIQE guidelines in that case
- The computational depth for FindERG is set very large (≥3 and beyond), where efficiency declines and there may be no result to verify

## Inputs

- Pandas DataFrame of gene expression values (rows/genes x columns/samples) from the ERgene example folder (aging mouse brain, SHED vs salt injection groups)
- Name of a candidate reference gene identified by ERgene.FindERG (e.g., 'Atp1a3')

## Outputs

- Normalized expression DataFrame produced by ERgene.normalizationdata
- Before/after boxplots and spectral density plots comparing SHED vs salt groups (Fig. 1a-d style)
- normalization_results.csv containing the FindERG screening ranking and the normalization verification result

## How to apply

Load the example dataset (aging mouse brain, SHED vs salt injection groups) as a Pandas DataFrame and run ERgene.FindERG, which computes the variation of the relative factor, sorts results smallest to largest, and returns the top 20 candidate reference genes; confirm the candidate (Atp1a3 in the lab test data) appears in the ranking. Call ERgene.normalizationdata with the DataFrame and that single reference gene to standardize the samples. Then quantify the between-group (SHED vs salt) difference in sample values before and after normalization, visualizing each state as boxplots and spectral density plots. The run is judged successful only if the normalized between-group difference is smaller than the raw difference; note the article's caveat that single-gene normalization is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated.

## Related tools

- **ERgene** (Provides FindERG to rank candidate reference genes and normalizationdata to standardize the data with a single internal reference gene whose effect the between-group comparison verifies) — github.com/Starlitnightly/ERgene
- **Pandas** (Holds the expression data in the DataFrame format required by FindERG and normalizationdata)
- **Python** (Execution environment for running ERgene screening, normalization, and before/after density/boxplot verification)

## Examples

```
import ERgene; ans = ERgene.normalizationdata(data, 'Atp1a3')
```

## Evaluation signals

- FindERG's ranked candidate list (top 20 genes, sorted by variation of the relative factor) contains the expected reference gene, Atp1a3, for the SHED vs salt dataset
- ERgene.normalizationdata returns a standardized DataFrame using only the single named reference gene
- Between-group (SHED vs salt) difference in normalized sample values is smaller than in the unnormalized data
- Before/after boxplots and density plots show visibly reduced spectral density difference between groups (analogous to Fig. 1a-d)
- Screening ranking and normalization verification results are written to normalization_results.csv

## Limitations

- Single-gene normalization is not acceptable per MIQE guidelines unless the investigator provides clear evidence of invariable expression; the README likewise warns it 'should be used with Caution!!!'
- ERgene's efficiency declines at computational depth 3 or larger, and setting the depth too large may yield no result at all
- No data availability or accession statement is given for the test datasets, so the verification depends on the repository's example folder

## Evidence

- [discussion] single-gene caveat: "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [discussion] normalization function role: "This function is used for the standardization of a single internal reference gene"
- [discussion] FindERG ranking behavior: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [readme] README normalization call and caution: "However, the literature has proved that single gene standardization is not desirable, so this method should be used with Caution!!!"
- [readme] README invocation example: "ans=ERgene.normalizationdata(data,'ASHGV40056316')"
