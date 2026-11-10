<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Transcriptomics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for computational transcriptomics: bulk and single-cell RNA-seq analysis.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-11%2C256-00284b" alt="11,256 skills">
  <img src="https://img.shields.io/badge/tools-2%2C086-00284b" alt="2,086 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-450-00284b" alt="450 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the transcriptomics skill collection of AgenticScienceBuilder (ASB): 11,256 skills over 2,086 software tools and 2 composite workflows, distilled from 450 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 9,283 of the 11,256 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/transcriptomics/v2` | ASB Transcriptomics Skill Collection v2 | 11256 | 2086 | 2026-11-09 | [10.5281/zenodo.24093100](https://doi.org/10.5281/zenodo.24093100) |

## Status

Collection `v2`, built from corpus cut `b5a519a80508`, is released on `main` at tag `transcriptomics-v2.0.0` and archived on Zenodo under the version DOI [10.5281/zenodo.24093100](https://doi.org/10.5281/zenodo.24093100). The earlier flat-layout collection `v1` stays in the repository as a legacy collection; it is not part of this release.

| Item | State |
|---|---|
| Release gate | `receipts/b5a519a80508/gate_report.json` reports `pass`: 14 of 16 checks pass and 2 are not applicable (`cut_manifest`, and `peer_review_claim` because the corpus holds no preprint or deposit DOI). [`receipts/b5a519a80508/VERIFY.md`](receipts/b5a519a80508/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/transcriptomics/v2` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/transcriptomics/v2` |
| IRI resolution (2026-10-31) | Registered at w3id.org. It redirects (HTTP 302) to a path in the asb-skill-collections registry that returns 404: the collection lives in this repository, not in the registry. |
| DOI | [10.5281/zenodo.24093100](https://doi.org/10.5281/zenodo.24093100) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093099](https://doi.org/10.5281/zenodo.24093099) (all versions) |

## What is inside

| Item | Count | Path under `collections/transcriptomics/v2/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 11,256 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 2,086 | `tools/` |
| Source papers | 450 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/b5a519a80508/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `bulk-rnaseq-differential-expression` | paired-end bulk RNA-seq reads must be adapter-trimmed, pseudoaligned to an external reference transcriptome, aggregated into raw gene counts, tested for differential expression, and interpreted with preranked pathway enrichment | 4 |
| `single-cell-clustering-annotation` | a raw single-cell RNA-seq count matrix must be quality-filtered, normalized, reduced to a PCA embedding, clustered by a neighborhood graph, tested for cluster-specific markers, and annotated against an external tissue marker reference | 6 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 9,032 |
| `noncommercial` | 1,946 |
| `restricted` | 0 |
| `unknown` | 278 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-transcriptomics
claude plugin install transcriptomics@asb-skills-transcriptomics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/transcriptomics/v2/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-transcriptomics/collections/transcriptomics/v2/leaves/bulk-rna-differential-expression-analysis <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-transcriptomics
python3 collections/transcriptomics/v2/bin/search_skills.py --query "differential expression of bulk RNA-seq" -k 3
python3 collections/transcriptomics/v2/bin/search_skills.py --target workflows --query "differential expression of bulk RNA-seq" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. bulk-rna-differential-expression-analysis
2. bulk-rnaseq-differential-expression-analysis
3. differential-expression-analysis

1. bulk-rnaseq-differential-expression
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/b5a519a80508/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `9f15448`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

443 of the 450 included papers are open access according to `corpus.yaml` (`access.is_oa`). The other 7 are closed access and listed as `link-only`.

Each leaf names its source paper in `provenance.source_papers` (11,256 of 11,256 leaves). 9,283 of the 11,256 quote it, or its repository README, in their `## Evidence` section. In the other 1,973, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 1,932 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 20 the section holds a quote in a form the gate parser does not read. 21 have no `## Evidence` section.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the released metabolomics collection.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- `indicium-adapters-transcriptomics`: the Indicium domain adapter for the same domain. Repository follows.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24093100](https://doi.org/10.5281/zenodo.24093100); its concept DOI, which covers all versions, is [10.5281/zenodo.24093099](https://doi.org/10.5281/zenodo.24093099).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
