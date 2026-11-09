---
name: probe-identifier-verification
description: Use when a probe-level expression matrix (e.g. GEO microarray data like
  GSE4786) yields candidate reference probes from ERgene.FindERG and you must confirm
  a specific probe ID exists in the output before normalizing.
when_to_use_negative:
- Input data have already been converted from probes to gene symbols, so probe IDs
  like 1439423_x_at no longer exist in the matrix
- The analysis goal is multi-gene normalization per MIQE guidelines rather than verification
  of a single-reference probe
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: Python library whose FindERG function returns the candidate reference probe
    identifiers to be verified
  repo: https://github.com/Starlitnightly/ERgene
- name: Pandas
  role: Loads the probe-level expression matrix as a DataFrame indexed by probe identifier
- name: Python
  role: Execution environment for running ERgene and Pandas on the probe matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:1566e04a65c8887ea923bd4bff3fb510c4015b00c9d4bdc60216ff0a1977ce33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Starlitnightly/ERgene
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# probe-identifier-verification

## Summary

Verify that microarray probe identifiers (e.g. Affymetrix-style IDs like 1439423_x_at) returned by ERgene.FindERG correspond to the intended reference-gene candidates in the input expression matrix, before using them for single-reference normalization.

## When to use

Use when a probe-level expression matrix (e.g. GEO microarray data like GSE4786) yields candidate reference probes from ERgene.FindERG and you must confirm a specific probe ID exists in the output before normalizing.

## When NOT to use

- Input data have already been converted from probes to gene symbols, so probe IDs like 1439423_x_at no longer exist in the matrix
- The analysis goal is multi-gene normalization per MIQE guidelines rather than verification of a single-reference probe

## Inputs

- Probe-level expression matrix as a pandas DataFrame (e.g. tab-separated GEO file read with pd.read_csv('gse.txt', sep='\t'), index set to the first column of probe IDs)
- FindERG output list/table of top-20 candidate reference probe identifiers
- Target probe identifier string to verify (e.g. '1439423_x_at')

## Outputs

- Confirmation that a given probe ID (e.g. 1439423_x_at) appears in the FindERG top-20 reference gene list
- Verified probe identifier suitable for ERgene.normalizationdata(Data, Ergname=...)
- Saved FindERG top-20 reference gene table (CSV/text)

## How to apply

Load the probe-level expression matrix into a pandas DataFrame indexed by probe ID (e.g. data.set_index(data.columns[0])). Run ERgene.FindERG(Data, Depth=2), which applies robust matrix block operation (every 1000 genes per block for large datasets), sorts the variance of relative factor change from smallest to largest, and returns the top 20 candidate reference genes. Check the returned list/table to confirm that the target probe (e.g. 1439423_x_at for the GSE4786 mouse cochlea dataset) is among the selected identifiers, keeping in mind FindERG returns probe IDs as-is when data are not converted from probes to genes. Only then pass the verified identifier to ERgene.normalizationdata(Data, Ergname='<probe_id>').

## Related tools

- **ERgene** (Python library whose FindERG function returns the candidate reference probe identifiers to be verified) — https://github.com/Starlitnightly/ERgene
- **Pandas** (Loads the probe-level expression matrix as a DataFrame indexed by probe identifier)
- **Python** (Execution environment for running ERgene and Pandas on the probe matrix)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 2); ans = ERgene.normalizationdata(data, '1439423_x_at')
```

## Evaluation signals

- The verified probe ID (e.g. 1439423_x_at for GSE4786) appears in the FindERG top-20 output list
- FindERG output length is 20 candidate genes (top 20 returned by the function)
- Every identifier in the FindERG output is present in the input DataFrame index (probe-level data not converted to genes)
- ERgene.normalizationdata accepts the verified probe string without a key error
- Comparing boxplots/density plots of raw (Fig. 2a,b) versus normalized (Fig. 2c,d) data shows narrowed sample spectral differences

## Limitations

- Single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated
- The README warns that 'the literature has proved that single gene standardization is not desirable, so this method should be used with Caution!!!'
- Computational depth of 3 or larger reduces efficiency and may yield no result; depth 2 (4.58 s for 1968 genes) is the tested efficient setting
- Test datasets are 'not converted by a probe', so verification applies at the probe-identifier level

## Evidence

- [other] Confirm that the probe 1439423_x_at is selected as the reference gene: [evidence span withheld — anchor status: not_found]
- [discussion] top 20 genes as a result: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] single internal reference gene standardization: "This function is used for the standardization of a single internal reference gene"
- [discussion] the test data are not converted by a probe: "The test data are not converted by a probe"
- [readme] single gene standardization caution: "However, the literature has proved that single gene standardization is not desirable, so this method should be used with Caution!!!"
