---
name: scientific-task-formulation
description: Use when a source article describes a newly released analysis tool or
  software package (e.g. RCAT for circadian clock genes) and the agent must decompose
  the article's claims, delivery artifacts, and workflow into a reusable, executable
  skill definition.
when_to_use_negative:
- The article does not describe a released tool or package — the task is purely conceptual
  or a literature review with no artifacts to index
- The analysis goal is running an established circadian pipeline end-to-end on real
  data; this skill only formulates the task, since the MVP does not execute the workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: RCAT
  role: The packaged analysis tool whose source code and Windows distribution define
    the task being formulated; download RCAT_WIN.zip, unzip, and run for circadian
    clock gene analysis
  repo: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
- name: GitHub
  role: Distribution and provenance channel hosting the RCAT source code and packaged
    releases
  repo: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
- name: Python
  role: Runtime bundled inside RCAT_WIN.zip; the packaging of 'all needed python packages'
    removes environment setup from the user's task
  repo: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12539-021-00471-2
    title: 'Rhythmic Component Analysis Tool (RCAT): A Precise, Efficient and User-Friendly
      Tool for Circadian Clock Genes Analysis'
  source_content_hash: sha256:1a3e9ad0a3ab2a8daf4498be3798931a8fee42f966a4f2d6e7a1e317492013bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
  tools:
  - GitHub
  - Python
  - RCAT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# scientific_task_formulation

## Summary

Formulates the scientific task of a computational-methods article into an explicit, reusable skill: what tool was built, for which inputs, delivered how, and with what verification signals. Here it captures RCAT, a packaged Windows tool for circadian clock gene rhythm analysis distributed via GitHub releases.

## When to use

Use when a source article describes a newly released analysis tool or software package (e.g. RCAT for circadian clock genes) and the agent must decompose the article's claims, delivery artifacts, and workflow into a reusable, executable skill definition.

## When NOT to use

- The article does not describe a released tool or package — the task is purely conceptual or a literature review with no artifacts to index
- The analysis goal is running an established circadian pipeline end-to-end on real data; this skill only formulates the task, since the MVP does not execute the workflow

## Inputs

- Time-series gene expression data for circadian clock genes (input expected by RCAT)
- RCAT_WIN.zip package retrieved from the GitHub releases page

## Outputs

- A structured skill card linking the RCAT tool, its distribution channel, its scientific purpose, and open reproducibility gaps
- Rhythmic component analysis results for circadian clock genes (once RCAT is executed)

## How to apply

Identify the tool artifact (RCAT source code at lzbbest/Rhythmic-Component-Analysis-Tool) and its packaged distribution form (RCAT_WIN.zip, a Windows build bundling all required Python packages). Map the article's stated purpose — precise, efficient, user-friendly analysis of circadian clock genes — onto concrete inputs (time-series gene expression data) and outputs (rhythmic component estimates). Trace the deployment path: download RCAT_WIN.zip from the GitHub releases page, unzip it, and run the tool without needing to configure a Python environment, which is the paper's user-friendliness claim. Record the paper link (SharedIt: https://rdcu.be/ctKPu) and DOI 10.1007/s12539-021-00471-2 as provenance anchors, then note any missing materials (e.g. no example datasets or parameter documentation is present in the README) as open items for the reproducibility record.

## Related tools

- **RCAT** (The packaged analysis tool whose source code and Windows distribution define the task being formulated; download RCAT_WIN.zip, unzip, and run for circadian clock gene analysis) — https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
- **GitHub** (Distribution and provenance channel hosting the RCAT source code and packaged releases) — https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
- **Python** (Runtime bundled inside RCAT_WIN.zip; the packaging of 'all needed python packages' removes environment setup from the user's task) — https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool

## Examples

```
Download RCAT_WIN.zip from https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool/releases and unzip it to run RCAT locally
```

## Evaluation signals

- The skill card cites the RCAT GitHub releases URL (https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool/releases) as the canonical download location
- Delivery form is correctly recorded as RCAT_WIN.zip for a Windows environment with all Python packages bundled, per the README
- Provenance anchors resolve: DOI 10.1007/s12539-021-00471-2 and the SharedIt link https://rdcu.be/ctKPu
- Missing information (no documented input formats, parameters, or example datasets in the README) is explicitly recorded rather than fabricated
- The formulated task scope matches the article's claim — circadian clock genes analysis — not unrelated time-series domains

## Limitations

- No execution workflow is performed at this stage; the skill formulates the task rather than running RCAT (MVP scope)
- The README documents only download-and-unpack instructions ('Please download RCAT_WIN.zip and unpackage it to use RCAT'); no input file formats, parameter defaults, or example datasets are specified
- The packaged distribution is Windows-specific; macOS/Linux users must fall back to the source code, whose setup requirements are not documented in the README
- RCAT versioning must be obtained from the GitHub releases page; no fixed release tag is pinned in the article text

## Evidence

- [readme] Tool distribution form: "We provide a packaged RCAT  version (Windows environment with all needed python package), which is freely available at:"
- [readme] Deployment step: "Please download RCAT_WIN.zip and unpackage it to use RCAT."
- [intro] Tool purpose claim: "Rhythmic Component Analysis Tool (RCAT): A Precise, Efficient and User-Friendly Tool for Circadian Clock Genes Analysis"
- [readme] Provenance link: "corresponding paper('Rhythmic Component Analysis Tool (RCAT): A Precise, Efficient and User-Friendly Tool for Circadian Clock Genes Analysis') by using the following SharedIt link:"
