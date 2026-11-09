---
name: public-geo-dataset-retrieval
description: Use when a study needs raw or processed public transcriptomics data (e.g.
  GEO accessions, Zenodo archives, GitHub processed files) for re-analysis, meta-analysis,
  or reproduction of published findings.
when_to_use_negative:
- The data are already locally available and QC-checked — re-downloading wastes effort
  and risks version drift
- The target data are unpublished, controlled-access, or require dbGaP/DUA approval
  rather than open GEO download
- The question needs newly generated data rather than re-analysis of deposited cohorts
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: COVID19-severity GitHub repository
  role: Source of processed count and anova files for the 7 curated transcriptomics
    datasets, downloadable as a zip
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: Streamlit webtool
  role: Web app for querying the retrieved database of gene expression differences
    between severe and mild COVID-19
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:6a1ea87979cfcce046dd73f5b1d8212aa6ca6cb6685b714e7a20fe3fd77e3e12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - COVID19-severity GitHub repository
  - Streamlit webtool
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# public-geo-dataset-retrieval

## Summary

Locates and downloads publicly deposited transcriptomics datasets (GEO accessions such as GSE155454 and GSE145926, Zenodo DOIs, and processed GitHub deposits) so that published COVID-19 severity analyses can be systematically re-analysed and reproduced.

## When to use

Use when a study needs raw or processed public transcriptomics data (e.g. GEO accessions, Zenodo archives, GitHub processed files) for re-analysis, meta-analysis, or reproduction of published findings.

## When NOT to use

- The data are already locally available and QC-checked — re-downloading wastes effort and risks version drift
- The target data are unpublished, controlled-access, or require dbGaP/DUA approval rather than open GEO download
- The question needs newly generated data rather than re-analysis of deposited cohorts

## Inputs

- GEO accession identifiers (e.g. GSE155454, GSE145926, GSE164805)
- Zenodo DOI 10.5281/zenodo.7114840 archive
- Processed data files from the GitHub repository kuanrongchan/COVID19-severity (counts and anova folders)
- Supplementary Table S5 listing accession numbers

## Outputs

- Downloaded raw count matrices and processed differential-expression tables per dataset
- Local dataset manifest with accession, group sizes (severe/mild/healthy), and provenance

## How to apply

Systematically gather the accession numbers cited by the source article (here 7 RNAseq datasets: GSE155454, Bibert 2021 via 10.17632/8wxhhykfnh.2, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805, plus single-cell BALF data GSE145926) and record sample group sizes (e.g. 140 severe vs 181 mild COVID-19 patients). Prefer the authors' processed files when available — for this study these are deposited on GitHub (kuanrongchan/COVID19-severity) and Zenodo (10.5281/zenodo.7114840), with processed tables containing t-test p-values, BH-adjusted p-values, ratios and fold-changes. Download via the GEO accession page or the repository's green 'Code' button zip download, then verify counts against the study design (severe/mild/healthy group sizes per dataset) before downstream Seurat or differential expression work.

## Related tools

- **COVID19-severity GitHub repository** (Source of processed count and anova files for the 7 curated transcriptomics datasets, downloadable as a zip) — https://github.com/kuanrongchan/COVID19-severity
- **Streamlit webtool** (Web app for querying the retrieved database of gene expression differences between severe and mild COVID-19) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py

## Examples

```
git clone https://github.com/kuanrongchan/COVID19-severity && cd COVID19-severity && pip install -r requirements.txt
```

## Evaluation signals

- Number of retrieved datasets matches the study design: 7 bulk RNAseq datasets plus the single-cell BALF dataset (GSE145926, 78,666 cells)
- Per-dataset sample counts match the README (e.g. Fong 2021: 14 severe, 18 mild, 6 healthy; Overmyer 2021: 50 severe, 50 mild)
- Processed files contain the expected columns: t-test p-value, BH-adjusted p-value, ratio, and fold-change
- All accession numbers resolve on GEO/Zenodo and checksums/file dimensions match the described deposits

## Limitations

- Dataset heterogeneity across the 7 studies was noted by the authors (Supplementary Fig. S9), so retrieved cohorts may not be directly comparable
- The curated datasets did not evaluate different SARS-CoV-2 variants, so retrieved data cannot answer variant-specific questions
- Processed GitHub files are author-generated; downstream conclusions depend on their re-analysis choices

## Evidence

- [discussion] All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity: "All processed data can be publicly downloaded from GitHub at https://github.com/kuanrongchan/COVID19-severity"
- [methods] Download this repository by clicking on the green code button and downloading the zip file.: "Download this repository by clicking on the green code button and downloading the zip file."
- [readme] The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19, or severe COVID-19 and healthy subjects."
- [readme] This dataset may be found at [GSE155454](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE155454).: "Dataset compares the gene expression differences between 14 severe COVID-19, 18 mild COVID-19, and 6 healthy subjects."
- [discussion] the accession numbers are shown in Table 1 and Supplementary Table S5: "the accession numbers are shown in Table 1 and Supplementary Table S5"
