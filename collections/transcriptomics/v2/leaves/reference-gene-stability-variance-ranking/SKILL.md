---
name: reference-gene-stability-variance-ranking
description: Use when a raw gene expression matrix (tab-delimited GSE-style file or
  Pandas DataFrame) must be screened for stable endogenous reference genes by ranking
  ΔF variance, or when reproducing/validating ERgene's FindERG top-20 output.
when_to_use_negative:
- The goal is normalization of expression data by already-selected reference genes
  — use ERgene.normalizationdata / Ergene.Normalizationdata instead of FindERG screening
- The investigator intends to normalize with a single reference gene without evidence
  of invariable expression — MIQE guidelines deem this unacceptable, so a stability
  screen (or multi-gene set) is required instead
- The computational depth is set very large, where efficiency declines and the program
  may return no result
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: Python library whose FindERG function performs the inverse internal-reference
    screening; used as the ground-truth implementation to compare against
  repo: https://github.com/Starlitnightly/ERgene
- name: Pandas
  role: Builds and holds the expression DataFrame input and supports the DataFrame-based
    screening interface
- name: Numpy
  role: Declared dependency of ERgene used for matrix/variance computation
- name: upsetplot
  role: Declared ERgene dependency used to generate the UpSet plot of candidate genes
    in Jupyter notebooks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:d235732f210c9aa840e3d6044cceffb30fc30b764b5a06cd12920237933ce23c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Starlitnightly/ERgene
  tools:
  - ERgene
  - Numpy
  - Pandas
  - upsetplot
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reference-gene-stability-variance-ranking

## Summary

Implements ERgene's inverse internal-reference algorithm: build per-sample relative correction factor matrices (F1, F2), compute their difference ΔF, rank genes by the variance of ΔF across samples, and return the top 20 most stable candidate endogenous reference genes.

## When to use

Use when a raw gene expression matrix (tab-delimited GSE-style file or Pandas DataFrame) must be screened for stable endogenous reference genes by ranking ΔF variance, or when reproducing/validating ERgene's FindERG top-20 output.

## When NOT to use

- The goal is normalization of expression data by already-selected reference genes — use ERgene.normalizationdata / Ergene.Normalizationdata instead of FindERG screening
- The investigator intends to normalize with a single reference gene without evidence of invariable expression — MIQE guidelines deem this unacceptable, so a stability screen (or multi-gene set) is required instead
- The computational depth is set very large, where efficiency declines and the program may return no result

## Inputs

- Gene expression matrix as a Pandas DataFrame (rows indexed by gene, e.g. loaded from a tab-separated GSE file such as gse.txt)
- Computational depth parameter (Depth, integer; e.g. 2 or 3) controlling samples used in the computation

## Outputs

- Ranked list of the top 20 candidate endogenous reference genes (gene identifiers sorted by ascending ΔF variance)
- top20_reference_genes.csv (ranked comparison of reimplemented vs. ERgene FindERG output)
- Optional UpSet plot (upsetplot) visualizing overlaps, as generated in Jupyter notebooks

## How to apply

Load the expression matrix into a Pandas DataFrame (e.g. via pd.read_csv with sep='\t', setting the first column as index for ERgene v1.2+). Reconstruct the inverse-of-internal-reference calculation: build the per-sample relative correction factor matrices F1 and F2, compute their difference ΔF, and take the per-gene variance of ΔF across samples as the stability measure. For large gene datasets (e.g. 1968 genes), apply robust block computation, processing genes in blocks of 1000 to keep memory tractable. Sort genes by ΔF variance from smallest to largest and return the top 20 genes as candidate reference genes; the Depth parameter of FindERG controls the computational depth (depth 2 screened 1968 genes in 4.58 s, depth 3 in 17.3 s on the README example). Validate by running ERgene.FindERG on the identical matrix and comparing the reimplemented top-20 ranked list, then write it to top20_reference_genes.csv.

## Related tools

- **ERgene** (Python library whose FindERG function performs the inverse internal-reference screening; used as the ground-truth implementation to compare against) — https://github.com/Starlitnightly/ERgene
- **Pandas** (Builds and holds the expression DataFrame input and supports the DataFrame-based screening interface)
- **Numpy** (Declared dependency of ERgene used for matrix/variance computation)
- **upsetplot** (Declared ERgene dependency used to generate the UpSet plot of candidate genes in Jupyter notebooks)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- Reimplemented top-20 gene list matches ERgene.FindERG output on the same input matrix (identical ranked identifiers)
- Output contains exactly 20 candidate reference genes, sorted by ΔF variance from smallest to largest
- Runtime scales with depth as reported (1968 genes: 4.58 s at depth 2; 17.3 s at depth 3 in the README example)
- For datasets over ~1000 genes, block computation is applied with every 1000 genes taken as a block
- Output written to top20_reference_genes.csv parses as a valid ranked CSV of gene identifiers

## Limitations

- Computational efficiency declines when depth is 3 or larger, and setting the depth too large may yield no result
- ERgene's normalizationdata provides single-gene standardization, but the paper warns single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is demonstrated
- No ERgene software version stated in the article text (README mentions new version 1.2.0 with 100x speed increase); installation instructions and data accession statements are absent from the article
- Selection is similarity-based comparison to NormFinder/geNorm rather than a guarantee — ERgene, NormFinder and geNorm found similar but not identical gene sets

## Evidence

- [discussion] Screen internal reference genes with ERgene.Ergene.Finderg(Data, Depth): "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [discussion] Block calculation and top-20 return: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Top-20 sorting rule: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] Computational depth trade-off: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [abstract] Inverse-process rationale: "The principle is based on the inverse process of the internal reference method, and the robust matrix block operation makes the selection of internal reference genes faster"
- [readme] README example invocation: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0])#This is Ver1.2
	ERgene.FindERG(data,3)"
