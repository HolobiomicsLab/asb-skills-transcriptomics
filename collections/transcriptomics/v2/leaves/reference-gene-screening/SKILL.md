---
name: reference-gene-screening
description: Use when you have a gene expression DataFrame (e.g. GEO microarray data
  like GSE125792 or GSE4786, probe-level or gene-level) and need to identify stable
  endogenous reference genes for qPCR/microarray normalization in Python.
when_to_use_negative:
- Your toolchain is R or Excel — the article notes geNorm (qbase+/Excel) and Normfinder
  (R/Excel) exist there; ERgene fills the absence of a Python screening package
- You intend to normalize with a single reference gene without evidence of invariable
  expression — MIQE guidelines make this unacceptable unless clearly demonstrated
- The dataset is too large with computational depth set too large — efficiency declines
  at depth 3+ and there may be no result
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: ERgene
  role: Python library providing FindERG (reference gene screening) and normalizationdata
    (single-gene standardization); install via pip install ERgene
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame format required by FindERG's data parameter and the
    computational basis for screening
- name: upsetplot
  role: Dependency used to render the UpSet plot of pairwise overlaps among screened
    candidate reference genes
- name: geNorm
  role: Comparator method (R/qbase+/Excel) whose selected genes were similar to ERgene's
    but with lower computational efficiency
- name: Normfinder
  role: Comparator method; ERgene's computational efficiency is nearly 90% higher
    than Normfinder's
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:43a8815b9a3516a1d00c0f5551988fb2927c6809533ab86d0fd222fa52d7dd99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Normfinder
  - Pandas
  - geNorm
  - upsetplot
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# reference-gene-screening

## Summary

Screen candidate endogenous reference genes from a gene expression matrix using ERgene's FindERG function, which applies the inverse process of the internal reference method with robust matrix block operation (1000-gene blocks) and returns the top 20 genes ranked by smallest variance of relative factor change.

## When to use

Use when you have a gene expression DataFrame (e.g. GEO microarray data like GSE125792 or GSE4786, probe-level or gene-level) and need to identify stable endogenous reference genes for qPCR/microarray normalization in Python.

## When NOT to use

- Your toolchain is R or Excel — the article notes geNorm (qbase+/Excel) and Normfinder (R/Excel) exist there; ERgene fills the absence of a Python screening package
- You intend to normalize with a single reference gene without evidence of invariable expression — MIQE guidelines make this unacceptable unless clearly demonstrated
- The dataset is too large with computational depth set too large — efficiency declines at depth 3+ and there may be no result

## Inputs

- Gene expression matrix as a pandas DataFrame (genes/probes as rows, samples as columns), e.g. GSE125792 (1968 genes, tab-separated) or probe-level GSE4786
- Computational depth parameter (integer, e.g. Depth=2 or Depth=3)

## Outputs

- Ranked list of top 20 candidate reference genes sorted by smallest variance of relative factor change
- UpSet plot (upsetplot.png) of pairwise candidate gene set overlaps when run in Jupyter
- Optional normalization via ERgene.normalizationdata(data, Ergname='<reference gene>') producing per-sample boxplots/density plots before vs after standardization

## How to apply

Load the expression matrix into a pandas DataFrame and set the first column as index (required since ERgene 1.2), then call ERgene.FindERG(data, Depth) where Depth is the computational depth — depth 2 is fastest (4.58 s for 1968 genes in the McLoughlin/GSE125792 test) while depth 3 yields a more focused candidate set (seven genes: RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19). FindERG computes the variation of the relative factor change across genes, sorts results from smallest to largest, and returns the top 20 genes. For large matrices, the robust block calculation treats every 1000 genes as one block, which is what makes selection faster than geNorm or Normfinder. Explicitly load dependencies (numpy, pandas, itertools, upsetplot) since importing the ERgene package does not attach them. In Jupyter, an UpSet plot of pairwise gene-set overlaps is generated automatically; you can validate reproducibility by computing per-gene sample-pair coverage across all pairs (e.g. 66 pairs from 12 samples) and confirming >80% coverage for the most stable genes and >50% duplication rate.

## Related tools

- **ERgene** (Python library providing FindERG (reference gene screening) and normalizationdata (single-gene standardization); install via pip install ERgene) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame format required by FindERG's data parameter and the computational basis for screening)
- **upsetplot** (Dependency used to render the UpSet plot of pairwise overlaps among screened candidate reference genes)
- **geNorm** (Comparator method (R/qbase+/Excel) whose selected genes were similar to ERgene's but with lower computational efficiency)
- **Normfinder** (Comparator method; ERgene's computational efficiency is nearly 90% higher than Normfinder's)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- Depth-2 run on GSE125792 (1968 genes) should reproduce the 20-gene panel: RPL19, RPS4X, RPL13, TUBA1A, SF3B5, RPS3a, RPL9, RPL39L, LDHA, RPS8, RPL31, FTL, RPS3, RPL22, PINLYP, CAPG, UQCRH, RPS5, RPSAP58, RPL36
- Depth-3 run on GSE125792 should return seven candidates (RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19) with runtime recorded (README example reports calculate time: 17.3s)
- Results are sorted by variance of relative factor change from smallest to largest and truncated to the top 20
- In Jupyter, an UpSet plot is generated; per-pair coverage across e.g. 66 sample pairs of 12 samples should show 10 genes above 80% coverage with >50% duplication rate
- For GSE4786, FindERG output should include probe 1439423_x_at; normalizationdata on Atp1a3 (lab mouse brain SHED vs salt) should visibly narrow between-group sample spectral difference in before/after boxplots and density plots

## Limitations

- Computational efficiency declines when the computational depth is set to 3 or larger, and no result may be produced if the depth is too large
- Single reference gene normalization is not acceptable under MIQE guidelines unless invariable expression is clearly evidenced; the README warns normalizationdata should be used with Caution
- ERgene may not be new in principle (the calculation uses a new formula), and the article reports no software version in the text, no data-accession statements, and no pip/conda instructions in the paper body (though the README gives pip install ERgene)

## Evidence

- [methods] DataFrame input requirement: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [methods] Ranking rule and top-20 cutoff: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [methods] Block calculation: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Runtime benchmark at depth 2: [evidence span withheld — anchor status: not_found]
- [discussion] Depth escalation failure mode: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [discussion] MIQE single-gene caution: "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [readme] README example invocation and output: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2
ERgene.FindERG(data,3)"
