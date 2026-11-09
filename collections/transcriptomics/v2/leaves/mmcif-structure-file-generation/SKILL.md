---
name: mmcif-structure-file-generation
description: Use when a cudaMMC (or 3D-GNOME-style) chromatin simulation has produced
  a bead-coordinate model file that must be converted into a parseable PDBx/mmCIF
  structure, especially when bead counts exceed legacy PDB limits.
when_to_use_negative:
- The downstream tool only consumes legacy PDB format and the model has few enough
  beads to fit PDB limits (mmCIF is chosen specifically because PDB cannot represent
  high bead counts).
- The goal is to inspect or post-process raw cudaMMC simulation data (anchors, PET
  clusters, singletons) rather than convert a finished 3D model to a structure file.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3176
tools:
- name: cudaMMC
  role: Produces the chromatin 3D model output file whose bead coordinates are converted
    into mmCIF; invoked as './build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/
    -n chr14_test'.
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME 3.0
  role: Web server into which cudaMMC is integrated; provides the modeling context
    whose structure outputs are represented in mmCIF.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:9243297c1e8b89dc969fd0dad53422587968fc37245d4f9f0e13f6a286c992c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME 3.0
  - cudaMMC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mmCIF-structure-file-generation

## Summary

Convert cudaMMC multiscale chromatin 3D model output into PDBx/mmCIF format by writing each bead of the modeled chromatin chain as an atom_site coordinate record. mmCIF is chosen over legacy PDB because it supports a higher number of beads per structure, required for high-resolution whole chromosome models.

## When to use

Use when a cudaMMC (or 3D-GNOME-style) chromatin simulation has produced a bead-coordinate model file that must be converted into a parseable PDBx/mmCIF structure, especially when bead counts exceed legacy PDB limits.

## When NOT to use

- The downstream tool only consumes legacy PDB format and the model has few enough beads to fit PDB limits (mmCIF is chosen specifically because PDB cannot represent high bead counts).
- The goal is to inspect or post-process raw cudaMMC simulation data (anchors, PET clusters, singletons) rather than convert a finished 3D model to a structure file.

## Inputs

- cudaMMC 3D model output file containing 3D coordinates of beads along the multiscale chromatin chain (named using the -n label, e.g. chr14_test)
- cudaMMC simulation config (config.ini) and output directory containing the run artifacts

## Outputs

- model.cif — a PDBx/mmCIF file with one atom_site coordinate record per chromatin bead

## How to apply

Start from the model file emitted by the documented simulation run './build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test', where the -n label ('chr14_test') appears in the output file names. Parse this output to extract the 3D coordinates of the beads along the multiscale chromatin chain (segment/anchor/subanchor beads placed with the configured loop_density and heatmap settings). Implement the converter so each bead is written as an atom_site record in PDBx/mmCIF format; do not emit legacy PDB records, since PDB cannot represent the number of pseudo-atoms needed for high-resolution whole chromosome models. Write the result as model.cif and validate that the file parses as a well-formed mmCIF structure with atom_site records covering all beads.

## Related tools

- **cudaMMC** (Produces the chromatin 3D model output file whose bead coordinates are converted into mmCIF; invoked as './build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test'.) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME 3.0** (Web server into which cudaMMC is integrated; provides the modeling context whose structure outputs are represented in mmCIF.)

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test  # then run the converter on the labeled model output to write model.cif
```

## Evaluation signals

- The resulting model.cif parses as a valid mmCIF structure (the paper reports no numeric quality threshold, so a successful parse is the acceptance criterion).
- Every bead in the cudaMMC model output has a corresponding well-formed atom_site coordinate record in the .cif file.
- The output file uses PDBx/mmCIF format rather than legacy PDB records, so large bead counts from whole chromosome models are representable.
- Output file names correspond to the -n label supplied to the cudaMMC run (e.g. chr14_test).

## Limitations

- The paper reports no numeric quality threshold for the converted structure; validation is limited to a successful parse of the .cif file.
- Beads are pseudo-atoms, not real atoms, so the mmCIF records represent chromatin beads and not atomic-resolution coordinates.
- Conversion applies to model output; the README documents that PET clusters should follow naming like 'GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt' for the simulation step, and input format errors upstream will propagate to the model file.

## Evidence

- [methods] mmCIF chosen over PDB: [evidence span withheld — anchor status: not_found]
- [methods] atom_site records: [evidence span withheld — anchor status: not_found]
- [readme] simulation invocation: "Example execution command:
```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```"
- [readme] output naming: "-n    label to be used in names of output files"
- [intro] multiscale beads: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
