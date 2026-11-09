---
name: chromatin-3d-model-coordinate-conversion
description: Use when a cudaMMC simulation output file containing 3D coordinates of
  chromatin beads must be turned into a parseable structural file, particularly for
  whole-chromosome models whose bead counts exceed legacy PDB format limits.
when_to_use_negative:
- Input is raw ChIA-PET data (anchors, PET clusters, singletons) rather than a cudaMMC
  model output file — these are simulation inputs, not coordinate structures to convert
- A downstream tool strictly requires legacy PDB ATOM/PDB format and cannot parse
  mmCIF — the paper deliberately chose mmCIF because PDB cannot hold the bead counts
  of whole-chromosome models
- The structure of interest is an experimental protein/nucleic-acid structure with
  real atoms, not a bead-based chromatin model from cudaMMC/3D-GNOME
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: cudaMMC
  role: GPU-enhanced multiscale Monte Carlo simulator whose 3D chromatin model output
    provides the bead coordinates that the converter writes into mmCIF atom_site records
  repo: https://github.com/SFGLab/cudaMMC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:527ada2ff3715b0dfe2e1b97d4f07c8d1edb9335ad0d40377902268b143f6304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - cudaMMC
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# chromatin-3d-model-coordinate-conversion

## Summary

Converts the 3D bead coordinates of a cudaMMC multiscale Monte Carlo chromatin model into a PDBx/mmCIF structure file (model.cif), where each chromatin bead is written as an atom_site record. mmCIF is chosen over legacy PDB format because it supports a higher number of beads (pseudo-atoms) per structure, which is required for high-resolution whole-chromosome models.

## When to use

Use when a cudaMMC simulation output file containing 3D coordinates of chromatin beads must be turned into a parseable structural file, particularly for whole-chromosome models whose bead counts exceed legacy PDB format limits.

## When NOT to use

- Input is raw ChIA-PET data (anchors, PET clusters, singletons) rather than a cudaMMC model output file — these are simulation inputs, not coordinate structures to convert
- A downstream tool strictly requires legacy PDB ATOM/PDB format and cannot parse mmCIF — the paper deliberately chose mmCIF because PDB cannot hold the bead counts of whole-chromosome models
- The structure of interest is an experimental protein/nucleic-acid structure with real atoms, not a bead-based chromatin model from cudaMMC/3D-GNOME

## Inputs

- cudaMMC 3D model output file containing bead coordinates for the simulated chromosomes (filename labeled via the -n flag, e.g. chr14_test)
- Optional metadata from the cudaMMC config.ini settings (chromosome/region, e.g. chr14 or 'chr14:1:2500000')

## Outputs

- model.cif — a PDBx/mmCIF structure file with well-formed atom_site coordinate records, one per chromatin bead

## How to apply

Start from the cudaMMC output file produced by a documented simulation run (e.g. './build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test'), where the -n label ('chr14_test') is embedded in the output file names. Parse that file to extract the 3D coordinates of the beads along the multiscale chromatin chain (segment, anchor, and subanchor levels). Write the converter so that each bead is emitted as a single PDBx/mmCIF atom_site record carrying its XYZ coordinates. Select mmCIF rather than legacy PDB format because mmCIF can represent a higher number of beads per structure, which is necessary for high-resolution whole chromosome models. Validate by parsing the resulting model.cif: acceptance is a successful, well-formed parse of the atom_site records for all beads; the paper reports no numeric quality threshold beyond that.

## Related tools

- **cudaMMC** (GPU-enhanced multiscale Monte Carlo simulator whose 3D chromatin model output provides the bead coordinates that the converter writes into mmCIF atom_site records) — https://github.com/SFGLab/cudaMMC

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- The resulting model.cif parses successfully as a valid PDBx/mmCIF structure (the paper reports no numeric quality threshold, so acceptance is a successful parse)
- Every bead in the cudaMMC output has a corresponding atom_site record — atom_site record count matches the bead count of the model
- XYZ coordinate values in the atom_site records match the 3D coordinates parsed from the cudaMMC model output file
- Output file naming follows the -n label convention used by cudaMMC (e.g. chr14_test-derived model.cif)

## Limitations

- No numeric quality threshold is reported in the paper; validation is limited to checking that model.cif parses as well-formed mmCIF with atom_site coordinate records for all beads
- Legacy PDB format cannot represent the bead counts of high-resolution whole-chromosome models, which is exactly why mmCIF is used — PDB-format-only consumers will not work
- The provided README does not document the converter's own CLI or options; the converter's invocation details are not grounded in the available documentation

## Evidence

- [full_text] mmCIF supports higher bead counts: [evidence span withheld — anchor status: not_found]
- [readme] cudaMMC simulation run producing the model output: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] -n label defines output file names: "-n    label to be used in names of output files"
- [readme] cudaMMC context and model source: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
