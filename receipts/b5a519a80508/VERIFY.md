# Verifying the receipt of cut b5a519a80508

This directory holds the receipts of the transcriptomics v2 release, cut `b5a519a80508`.
`gate_report.json` is a strict release-gate receipt: it binds every file under
`collections/transcriptomics/v2/` (the workflows included) and the policy that measured them. Verification
measures the collection again and compares the result with the receipt; it writes nothing.

## What the receipt was written with

- Tooling: HolobiomicsLab/asb-skill-collections at commit `95cb2a07189cfc2776c644ad283cf9f621b9ca96`, the commit the CI workflows
  pin. `gate_report.json` was regenerated on 2026-11-10 and verifies with it. `source.json` names
  `b8c22a6bc`, the tooling commit that assembled the collection. Gate policy `asbb-content-gate/1.4`.
- `promote_logic_source: agentic_science_builder.release.promote` in `gate_report.json`: the gate
  loaded the promotion module of the AgenticScienceBuilder package. Verify in an environment where the
  `agentic_science_builder` package is importable. Where it is not, the gate loads a vendored module
  instead, the policy digest differs, and verification reports a policy mismatch.
- Python with PyYAML, and the AgenticScienceBuilder package installed in the same environment. The
  receipt was verified with Python 3.11.15 and the package taken from the pipeline repository at commit `8c22ee66`.

## Commands

From a directory that holds a checkout of this repository (asb-skills-transcriptomics) at the commit that
contains this file:

```bash
git clone https://github.com/HolobiomicsLab/asb-skill-collections
git -C asb-skill-collections checkout 95cb2a07189cfc2776c644ad283cf9f621b9ca96
python3 -m venv .venv-verify
.venv-verify/bin/python -m pip install pyyaml
.venv-verify/bin/python -m pip install -e <path to a checkout of AgenticScienceBuilder>
cd asb-skill-collections
../.venv-verify/bin/python scripts/release_gate.py ../asb-skills-transcriptomics/collections/transcriptomics/v2 --verify --report ../asb-skills-transcriptomics/receipts/b5a519a80508/gate_report.json
```

A verified receipt prints a line starting with
`ASBB receipt verification: verified (exit 0)` and the command exits 0. Exit 1 names the
receipt fields that differ (binding, policy, inventory, checks); exit 2 is a usage error.

## The other receipts

- `source.json`: the ASB, tooling and corpus commits, the cut identifier, the view and the
  digest of each stage script. Its `output_tree_sha256` is the assembled tree before the
  workflows landed; `gate_report.json` binds the tree as committed.
- `checks.json`, `gate.txt`, `lint.txt`, `check_license_tiers.txt`, `check_provenance_tiers.txt`,
  `check_tools_index.txt`, `surface.txt` and `verify.txt`: the seven checks run on the committed
  collection. `gates.txt` is the summary of the same checks on the assembled tree, before the
  workflows landed; `join.txt`, `collect.txt`, `bootstrap.txt` and `same_paper.txt` are the logs
  of the assembly stages.
- `evidence_trim_receipt.json`: each evidence span the collector kept or shortened, by
  location (leaf and body line, or tool), DOI, lengths and SHA-256 of the UTF-8 span. It holds
  no span text; a reader who has the paper can match a row by its digest.
- `collect_receipt.json`, `corpus_join_receipt.json`, `bootstrap_receipt.json`, `mapping.json`,
  `same_paper_pairs.json` and `corpus.yaml`: the inputs and outputs of the assembly stages.
- `pins_report.json` and `verdicts/`: how each workflow stage pin resolved to a skill of this
  collection, and the verification verdict of each workflow.
