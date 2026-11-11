---
name: transcriptomics-router
description: Use for any transcriptomics how-to question (which method, tool, parameters or pipeline to use), before answering from memory. Searches 11,256 evidence-grounded transcriptomics skills distilled from method papers and their code, 2 end-to-end workflows and 2,086 tool records, then reads the one that fits.
license: CC-BY-4.0
metadata:
  skills_count: 11256
  leaf_dir: leaves
  retrieval: bin/search_skills.py
  indexes:
  - skills_index.json
  - workflows/workflows_index.json
  - tools_index.json
schema_version: 0.2.0
---

# ASB Transcriptomics router

**11,256 evidence-grounded skills**, each derived from a
method paper (journal article or preprint) and its code, with 2 end-to-end workflows and 2,086 tool records.
They ship beside this router as data, in `leaves/<slug>/SKILL.md`. **Do not
list `leaves/` and do not read the indexes whole.** Search, then read the one
file you need.

## 1. Search

Run the commands as written, from any working directory; do not `cd`.
If a command still shows a `${...}` variable instead of an absolute path, the
host did not fill it in: replace the variable with the absolute directory of
this file, `<collection root>/skills/_router`.

```bash
# one atomic skill
python3 "${CLAUDE_SKILL_DIR}/../../bin/search_skills.py" --query "<the user's task>" -k 10
# an end-to-end workflow for a whole analysis goal
python3 "${CLAUDE_SKILL_DIR}/../../bin/search_skills.py" --query "<the user's goal>" --target workflows -k 3
# a tool record (licence tier, source DOIs)
python3 "${CLAUDE_SKILL_DIR}/../../bin/search_skills.py" --query "<a tool name or task>" --target tools -k 5
```

Each hit prints the absolute `read:` path; a skill hit also its licence tier and source DOI. Narrow with `--tool <name>` or `--edam <iri-substring>`.
Python 3.9 or later, standard library only, no network; where `python3` is missing, use `python` or `py -3`.

## 2. Apply

Read the `read:` path of the chosen hit, `leaves/<slug>/SKILL.md`. Its
frontmatter carries `tools` and `metadata.tools` (what to install or invoke) and `provenance.source_papers`
(source paper DOIs). Find verbatim evidence lines in the body, then follow the
procedure.
In the answer, name the skill and give its source DOI and its licence tier.

## 3. Ground (recommended)

Before trusting a parameter, threshold or default, verify it against the
source-paper DOIs of the leaf, listed in its `provenance.source_papers` frontmatter.

## Licence tiers

Each skill hit prints `tier`, the licence tier of the tool it relies on (the
skill text itself is CC-BY-4.0). Check it before presenting a candidate.
`open`: commercial use allowed. `noncommercial`: academic or non-commercial use
only; get explicit confirmation of a non-commercial purpose before applying it.
`restricted`: a licence was established and it constrains reuse (for example a
vendor licence); say so before commercial use or redistribution. `unknown`: no
licence for the tool was established; say so, and treat the tool as all rights
reserved until the user has checked it.

## Report and contribute

`transcriptomics/v2` improves from use. Offer a report yourself, once per
problem, when a procedure failed or no longer matches the tool, the search
returned nothing usable and the work was done unaided, the user names a
method paper whose DOI the search does not find, the leaves had to be
composed by hand, or a command of this collection failed. Name the route:

1. this skill is wrong or could be better;
2. a skill is missing: request it, or propose yours;
3. a paper is not referenced: propose it;
4. something technical failed: report it.

After a no, do not raise it again. Draft the report, show the user its exact
title, labels and body, and post only on an explicit yes. Never paste paths,
credentials or sample names.

```bash
python3 "${CLAUDE_SKILL_DIR}/../../bin/report_issue.py" skill --skill <slug> --kind defect --symptom "<what happened>"
```

Routes: `skill`, `missing`, `paper`, `fault` (`--help` lists the fields). It
redacts and prints the report and a prefilled address. With `gh` and the
user's yes, run `gh issue create --repo HolobiomicsLab/asb-skill-collections`; otherwise give the user
the address.

Reports go to the public registry; a maintainer carries them into the private
domain repository.
