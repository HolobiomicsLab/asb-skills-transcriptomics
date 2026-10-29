---
name: cudammc-simulation-output-parsing
description: Use when you have a cudaMMC output file (labeled with the -n flag, e.g.
  'chr14_test') containing 3D coordinates of beads along the multiscale chromatin
  chain and need to produce a structural file (model.cif) readable by structure-aware
  tools.
when_to_use_negative:
- The task requires legacy PDB-format output and the model contains more beads than
  PDB can represent — mmCIF is chosen precisely because PDB cannot hold the bead count
  of high-resolution whole chromosome models.
- The downstream analysis consumes raw ChIA-PET inputs (anchors, PET clusters, singletons)
  rather than a reconstructed 3D structure — those are cudaMMC inputs, not outputs
  to convert.
- You need to modify the simulation itself (e.g. use_motif_orientation, max_pet_length,
  subanchor heatmap settings) — that is configuration of cudaMMC, not parsing of its
  output.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cudaMMC
  role: Generates the multiscale Monte Carlo chromatin 3D model output file whose
    bead coordinates are parsed and converted to mmCIF
  repo: https://github.com/SFGLab/cudaMMC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:9ad66bf0899ea19c1b7c9a5fd1de17d4223974310fd8078cb43717a86ff02403
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - cudaMMC
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cudammc-simulation-output-parsing

## Summary

Parse the 3D bead-coordinate model output produced by a cudaMMC multiscale Monte Carlo simulation run and convert it into PDBx/mmCIF format, chosen over legacy PDB because mmCIF can represent a higher number of beads per structure, which is necessary for high-resolution whole chromosome models.

## When to use

Use when you have a cudaMMC output file (labeled with the -n flag, e.g. 'chr14_test') containing 3D coordinates of beads along the multiscale chromatin chain and need to produce a structural file (model.cif) readable by structure-aware tools.

## When NOT to use

- The task requires legacy PDB-format output and the model contains more beads than PDB can represent — mmCIF is chosen precisely because PDB cannot hold the bead count of high-resolution whole chromosome models.
- The downstream analysis consumes raw ChIA-PET inputs (anchors, PET clusters, singletons) rather than a reconstructed 3D structure — those are cudaMMC inputs, not outputs to convert.
- You need to modify the simulation itself (e.g. use_motif_orientation, max_pet_length, subanchor heatmap settings) — that is configuration of cudaMMC, not parsing of its output.

## Inputs

- cudaMMC 3D model output file (produced by ./build/cudaMMC with -s config.ini, -o output directory, -n label such as chr14_test) containing 3D coordinates of beads along the multiscale chromatin chain

## Outputs

- model.cif — a PDBx/mmCIF structure file with well-formed atom_site coordinate records for every bead of the modeled chromatin structure

## How to apply

Take the cudaMMC 3D model output produced by a run such as './build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test', where the -n label ('chr14_test') is used in the names of the output files. Parse the model output file to extract the 3D coordinates of the beads along the multiscale chromatin chain. Write each bead as an atom_site record in PDBx/mmCIF format; mmCIF is selected over legacy PDB because it supports a higher number of beads (pseudo-atoms) per structure, which is required for high-resolution whole chromosome models. Run the converter on the model output file to write model.cif. Acceptance is a successful, well-formed parse of the .cif atom_site records for all beads — the paper reports no numeric quality threshold.

## Related tools

- **cudaMMC** (Generates the multiscale Monte Carlo chromatin 3D model output file whose bead coordinates are parsed and converted to mmCIF) — https://github.com/SFGLab/cudaMMC

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- model.cif parses as a valid mmCIF structure (the paper reports no numeric quality threshold; acceptance is a successful parse of the .cif file)
- Every bead from the cudaMMC model output appears as an atom_site record — no beads dropped
- atom_site records contain well-formed 3D coordinate values (x, y, z) for each pseudo-atom bead
- Output file naming follows the -n label convention from the cudaMMC run (e.g. label 'chr14_test' used in the output file names)

## Limitations

- The paper reports no numeric quality threshold for the converted structure — validation is limited to successful parsing of the .cif, not structural accuracy of the model.
- Beads are pseudo-atoms, not real atoms; legacy PDB format cannot represent the bead count of high-resolution whole chromosome models, so downstream consumers must support mmCIF.
- The output is a model of chromatin beads whose coordinates depend on cudaMMC settings (e.g. freq_dist_power, genomic_dist_scale, loop_density) — conversion does not validate the underlying simulation.

## Evidence

- [methods] mmCIF output converter tool was chosen over PDB because mmCIF can represent a higher number of beads per structure, which is necessary for high-resolution whole chromosome models: [evidence span withheld — anchor status: not_found]
- [methods] cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test: "cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [abstract] Open-source software and manual and sample data are freely available on https://github.com/SFGLab/cudaMMC.: "Open-source software and manual and sample data are freely available on https://github.com/SFGLab/cudaMMC."
- [readme] label to be used in names of output files: "-n    label to be used in names of output files"
