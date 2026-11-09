---
name: ecdna-breakpoint-concordance-comparison
description: Use when circlehunter has been run on ATAC-Seq samples (e.g. GBM39, PC3,
  COLO320DM) with known WGS- and FISH-validated ecDNA, and you must verify that predicted
  ligation relationships and breakpoints match the reference structures.
when_to_use_negative:
- When no independent WGS or FISH validation exists for the sample — accuracy of ecDNA
  calls for all types of focal amplifications cannot be guaranteed without such a
  reference
- When the input data are WGS rather than ATAC-Seq — use WGS-based frameworks such
  as AmpliconArchitect instead
- When the goal is single-cell ATAC-Seq ecDNA detection — this application has yet
  to be explored
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Pipeline whose reconstructed ecDNA circles, ligation relationships, and breakpoints
    are the objects being compared against validated structures
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that executes the circlehunter pipeline producing the outputs
    under evaluation
  repo: https://snakemake.readthedocs.io/
- name: ATAC-Seq
  role: Assay whose enhanced chromatin accessibility signals on ecDNA underlie the
    circlehunter calls being validated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:a2d654ff0904e8f8a1ea44df2f1004f91e225d4e45858463dbdffde92a45bfd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ATAC-Seq
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ecdna-breakpoint-concordance-comparison

## Summary

A validation skill for judging whether circlehunter-reconstructed ecDNA structures from ATAC-Seq samples faithfully reproduce WGS/FISH-validated ecDNA by comparing circular structures, ligation relationships, and breakpoints against published reference figures (e.g. Fig. 2D-F).

## When to use

Use when circlehunter has been run on ATAC-Seq samples (e.g. GBM39, PC3, COLO320DM) with known WGS- and FISH-validated ecDNA, and you must verify that predicted ligation relationships and breakpoints match the reference structures.

## When NOT to use

- When no independent WGS or FISH validation exists for the sample — accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed without such a reference
- When the input data are WGS rather than ATAC-Seq — use WGS-based frameworks such as AmpliconArchitect instead
- When the goal is single-cell ATAC-Seq ecDNA detection — this application has yet to be explored

## Inputs

- circlehunter output structures (reconstructed circles, ligation relationships, predicted breakpoints) from ATAC-Seq BAM/analysis outputs
- Reference ecDNA structures validated by WGS and FISH (e.g. published Fig. 2D-F for GBM39, PC3, COLO320DM)
- config.yaml enumerating the three GEO ATAC-Seq samples and their desired outputs

## Outputs

- Per-sample concordance assessment of ecDNA circular structures against WGS/FISH-validated references
- Comparison of ligation relationships (e.g. all six PC3 fragment ligations recovered)
- Breakpoint-level agreement report (predicted vs. validated breakpoints)

## How to apply

After running the circlehunter Snakemake pipeline (snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml) on the ATAC-Seq samples, extract the reconstructed circular structures, ligation relationships, and predicted breakpoints from the workflow outputs. For each sample, compare these against the independently validated ecDNA structures reported for that cell line (for GBM39, PC3, and COLO320DM these are the WGS- and FISH-validated structures in Fig. 2D-F). Concordance is judged at two levels: (1) ligation relationships — e.g. for the PC3 cell line's six-fragment ecDNA, all ligation relationships should be recovered; and (2) breakpoints — most breakpoints should be accurately predicted, with the simple GBM39 ecDNA fully recovered. Treat extra 'computational sub-structures' (as seen for COLO320DM) as a known artifact of short read-length sequencing rather than an outright failure, but document them as deviations from the validated structure.

## Related tools

- **circlehunter** (Pipeline whose reconstructed ecDNA circles, ligation relationships, and breakpoints are the objects being compared against validated structures) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that executes the circlehunter pipeline producing the outputs under evaluation) — https://snakemake.readthedocs.io/
- **ATAC-Seq** (Assay whose enhanced chromatin accessibility signals on ecDNA underlie the circlehunter calls being validated)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- GBM39's simple ecDNA is identified accurately, matching the WGS/FISH-validated structure
- For PC3's six-fragment ecDNA, all ligation relationships are correctly identified and most breakpoints are predicted
- Reconstructed structures visually/structurally reproduce the validated ecDNA in Fig. 2D-F for all three samples
- Extra predicted sub-structures (e.g. for COLO320DM) are explainable as short-read-length artifacts and documented as deviations
- Calls on other focal amplification classes are flagged as uncertain, since accuracy for all focal amplification types cannot be guaranteed

## Limitations

- Current benchmarking strategy has limitations; accuracy of ecDNA calls cannot be guaranteed for all types of focal amplifications
- Failure rate on complex structural variants not yet explored; future studies needed
- Short read-length sequencing may cause circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- Other classes of focal amplification with misregulated chromatin may be detected as false-positive ecDNA

## Evidence

- [results] On real samples, circlehunter accurately identified the simple GBM39 ecDNA and, for the PC3 cell line's six-fragment ecDNA, accurately identified all ligation relationships and predicted most breakpoints: [evidence span withheld — anchor status: not_found]
- [methods] workflow comparison step: [evidence span withheld — anchor status: not_found]
- [discussion] read-length limitation: "short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures"
- [readme] README run command: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] benchmarking caveat: "we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications"
