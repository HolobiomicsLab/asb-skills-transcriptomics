---
name: gene-expression-dataframe-normalization
description: Use when a raw gene-expression matrix (e.g., tab-separated GSE-style
  text or CSV loaded as a Pandas DataFrame) needs an internal reference gene identified
  and single-gene normalization to reduce between-group sample variation.
when_to_use_negative:
- The input is not a gene-expression DataFrame (FindERG requires its data parameter
  in DataFrame format), e.g., raw count matrices in other formats not yet loaded as
  a Pandas DataFrame
- 'You need MIQE-compliant normalization: the paper states it is not acceptable to
  normalize by a single internal reference gene unless invariable expression is clearly
  demonstrated'
- The dataset is extremely large and a very large computational depth is required
  — efficiency declines at depth 3 or larger and no result may occur at too large
  a depth
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3572
tools:
- name: ERgene
  role: Python library providing FindERG (reference gene screening) and normalizationdata
    (single-gene standardization); install via 'pip install ERgene'
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame format required by FindERG and powers the normalization/screening
    computation
- name: Python
  role: Runtime environment in which the ERgene workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:3e334377ddda26e1b6a243f133942f6e39e44872bf3566c1876efd0a6acc8414
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gene-expression-dataframe-normalization

## Summary

Screen candidate endogenous reference genes from a gene-expression Pandas DataFrame using ERgene's FindERG, then standardize the data with ERgene's normalizationdata using the identified reference gene (e.g., Atp1a3 on the SHED vs salt mouse brain example).

## When to use

Use when a raw gene-expression matrix (e.g., tab-separated GSE-style text or CSV loaded as a Pandas DataFrame) needs an internal reference gene identified and single-gene normalization to reduce between-group sample variation.

## When NOT to use

- The input is not a gene-expression DataFrame (FindERG requires its data parameter in DataFrame format), e.g., raw count matrices in other formats not yet loaded as a Pandas DataFrame
- You need MIQE-compliant normalization: the paper states it is not acceptable to normalize by a single internal reference gene unless invariable expression is clearly demonstrated
- The dataset is extremely large and a very large computational depth is required — efficiency declines at depth 3 or larger and no result may occur at too large a depth

## Inputs

- Gene expression matrix as a Pandas DataFrame (genes as rows/index, samples as columns; e.g., tab-separated 'gse.txt' from GSE125792 or the ERgene example aging mouse brain SHED vs salt dataset)
- Computational depth parameter for FindERG (e.g., 2 or 3)
- Name of the identified reference gene for normalizationdata (e.g., 'Atp1a3')

## Outputs

- Ranked list of top 20 candidate endogenous reference genes (variation of relative factor sorted smallest to largest)
- Normalized expression DataFrame returned by normalizationdata
- Before/after boxplots and density plots showing reduced between-group sample spectral density difference
- normalization_results.csv containing screening ranking and normalization verification results

## How to apply

Load the expression data into a Pandas DataFrame with genes as rows (set the first column as index, as required in ERgene v1.2) and call ERgene.FindERG(data, depth), which computes the variation of the relative factor, sorts results from smallest to largest, and returns the top 20 candidate reference genes; note every 1000 genes are taken as a block for large datasets. Inspect the ranked list to confirm the candidate reference gene for the dataset (e.g., Atp1a3 for the aging mouse brain SHED vs salt example). Then call ERgene.normalizationdata(data, gene_name) to standardize the data with that single internal reference gene. Compare between-group (SHED vs salt) spectral density boxplots/density plots before and after normalization to verify the difference is reduced. Record the screening ranking and normalization verification to normalization_results.csv. Use depth 2 for speed (1968 genes screened in 4.58 s); depth 3 or larger slows computation and may return no result at excessive depth.

## Related tools

- **ERgene** (Python library providing FindERG (reference gene screening) and normalizationdata (single-gene standardization); install via 'pip install ERgene') — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame format required by FindERG and powers the normalization/screening computation)
- **Python** (Runtime environment in which the ERgene workflow is executed)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3); ans = ERgene.normalizationdata(data, 'ASHGV40056316')
```

## Evaluation signals

- FindERG's ranked output contains the expected candidate reference gene (e.g., Atp1a3 for the lab mouse brain SHED vs salt test data)
- normalizationdata yields a smaller between-group (SHED vs salt injection) sample difference than the raw, unnormalized data
- Before/after boxplots and density plots show visibly reduced between-group spectral density difference
- FindERG returns up to 20 genes sorted by variation of the relative factor from smallest to largest
- Results are successfully written to normalization_results.csv with the screening ranking and normalization verification

## Limitations

- Single-gene normalization is not acceptable per MIQE guidelines unless the investigator provides clear evidence that the reference gene is invariable; the README warns this method should be used with Caution
- Computational efficiency declines at depth 3 or larger, and setting the depth too large may produce no result
- No ERgene software version stated in the article (README notes version 1.2.0, ~100x speed increase, and requires setting the first column as index from v1.2)

## Evidence

- [discussion] FindERG computes variation of relative factor and returns top 20 genes: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] normalizationdata standardizes by a single internal reference gene: "This function is used for the standardization of a single internal reference gene"
- [discussion] single-gene normalization caution per MIQE: "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [discussion] FindERG requires DataFrame input: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [readme] README FindERG usage example: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2"
- [readme] README normalizationdata usage: "ans=ERgene.normalizationdata(data,'ASHGV40056316')"
- [discussion] efficiency declines at depth 3 or larger: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
