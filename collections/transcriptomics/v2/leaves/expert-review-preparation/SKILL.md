---
name: expert-review-preparation
description: 'Use when an article package pairs a tool paper with a GitHub release
  and the goal is an expert review: connect each stated claim (e.g. a packaged Windows
  RCAT release) to the actual repository artifact and log gaps.'
when_to_use_negative:
- The goal is to actually run RCAT on circadian clock gene time-series data — this
  skill only inspects and reviews the package; no workflow is executed in the MVP.
- The repository documentation already contains complete run commands and test datasets,
  leaving nothing to assess via claim-to-artifact gap analysis.
- The input is raw circadian data rather than an indexed article package; RCAT analysis
  itself is out of scope for this skill.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RCAT
  role: 'Subject tool of the review: the packaged Windows release (RCAT_WIN.zip) whose
    claims are checked against the repository artifact'
  repo: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12539-021-00471-2
    title: 'Rhythmic Component Analysis Tool (RCAT): A Precise, Efficient and User-Friendly
      Tool for Circadian Clock Genes Analysis'
  source_content_hash: sha256:b0321038bd810b0ba7c7dbdbd8851e40d621954323813cd67a7ac3daafbd3392
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool
  tools:
  - Python
  - RCAT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# expert_review_preparation

## Summary

Prepares an expert review of a published scientific software article package (here, the RCAT tool for circadian clock genes analysis) by connecting article claims to repository artifacts and recording what is missing for reproducibility. It exists so a reviewer can judge whether the packaged release matches the paper's stated claims.

## When to use

Use when an article package pairs a tool paper with a GitHub release and the goal is an expert review: connect each stated claim (e.g. a packaged Windows RCAT release) to the actual repository artifact and log gaps.

## When NOT to use

- The goal is to actually run RCAT on circadian clock gene time-series data — this skill only inspects and reviews the package; no workflow is executed in the MVP.
- The repository documentation already contains complete run commands and test datasets, leaving nothing to assess via claim-to-artifact gap analysis.
- The input is raw circadian data rather than an indexed article package; RCAT analysis itself is out of scope for this skill.

## Inputs

- Enriched article index (tools, workflow_steps, filter_steps, findings) with DOI and repo URLs
- Repository README for lzbbest/Rhythmic-Component-Analysis-Tool
- GitHub releases page expected to host RCAT_WIN.zip
- Source cards (task_id, research_question, finding, workflow, tools)

## Outputs

- Expert-review record mapping article claims to repository artifacts
- Gap list of missing reproducibility information (run commands, test data, expected outputs)
- Verification status of repo URL, release asset (RCAT_WIN.zip), and SharedIt link

## How to apply

Start from the enriched article index — DOI, tool entries (RCAT), repo URLs, findings, and workflow steps — and verify each claim against the repository documentation. For RCAT, the intro claims 'a packaged RCAT version (Windows environment with all needed python package)' available at the GitHub releases page; check that the README's lzbbest/Rhythmic-Component-Analysis-Tool repository matches, that RCAT_WIN.zip is the described release asset, and that the SharedIt link (https://rdcu.be/ctKPu) resolves to the paper. Record any missing information explicitly: the README documents no concrete run command, no test input data, and no expected outputs, so reproducibility of the analysis itself cannot be verified from the package alone. In this MVP no workflow is executed — the review is an inspection-and-gap-report step, not a tool run.

## Related tools

- **RCAT** (Subject tool of the review: the packaged Windows release (RCAT_WIN.zip) whose claims are checked against the repository artifact) — https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool

## Evaluation signals

- The repo URL in the index resolves to the repository named in the README (lzbbest/Rhythmic-Component-Analysis-Tool).
- The intro's claim of a 'packaged RCAT version (Windows environment with all needed python package)' is matched by an RCAT_WIN.zip asset on the GitHub releases page.
- Every finding traced from the intro has at least one verbatim evidence span from the article or README.
- The gap list explicitly records that the README provides no run command, test input, or expected output for RCAT.
- No claims are asserted beyond what the README and index sections support (e.g. do not invent execution instructions).

## Limitations

- The README documents only how to obtain the tool (download RCAT_WIN.zip and unpackage it); no usage instructions, test datasets, or expected outputs are provided, so functional reproducibility cannot be verified from the package.
- In the MVP, no workflow is executed — the review is limited to inspecting indexed article artifacts and repository documentation.
- The packaged release targets a Windows environment with all needed Python packages; behavior on other platforms is not described in the provided context.

## Evidence

- [intro] Packaged Windows release claim: "We provide a packaged RCAT version (Windows environment with all needed python package), which is freely available at: https://github.com/lzbbest/Rhythmic-Component-Analysis-Tool/releases"
- [intro] Release acquisition workflow step: "Please download RCAT_WIN.zip and unpackage it to use RCAT."
- [readme] README scope of repository: "This page is all about source code of RCAT."
