---
name: crosslink-induced-truncation-quantification
description: Use when starting from eCLIP/STAMP IP and size-matched input BAM files
  and you need nucleotide-resolution crosslink signal (truncation/mutation events)
  rather than raw read coverage, e.g. to build Metatruncate objects for metagene plots.
when_to_use_negative:
- When the desired signal is total read-coverage density from BAM alignments rather
  than crosslink sites — use the metadensity.readdensity module instead of truncation
  quantification.
- When a pre-computed WIG signal track already exists, since Metadensity can load
  it directly to speed up computation instead of re-deriving events from BAMs.
- When inputs are not CLIP IP/input sequencing data (e.g. already aggregated feature
  tables); Metadensity expects BAM/BIGWIG CLIP files registered in a dataloader and
  manifest.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Metadensity
  role: Provides the truncation/Metatruncate machinery for tallying CITs and CIMs
    per nucleotide and background-normalizing against size-matched inputs
  repo: https://github.com/YeoLab/Metadensity
- name: Jupyter
  role: Notebook interface for running the truncation workflow (e.g. 1_Example_on_SF3B4)
    and validating per-nucleotide signal tables
  repo: https://github.com/algaebrown/Metadensity
- name: Docker
  role: Prebuilt Metadensity environment on DockerHub for reproducible installation
    of the truncation pipeline
  repo: https://hub.docker.com/repository/docker/algaebrown/metadensity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac083
    title: 'Metadensity: a background-aware python pipeline for summarizing CLIP signals
      on various transcriptomic sites'
  source_content_hash: sha256:2e31d464bc4119491097757fa41a9347ebae2fa73e3899f0b3f84abd2302c2e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YeoLab/Metadensity
  tools:
  - Docker
  - Jupyter
  - Metadensity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# crosslink-induced-truncation-quantification

## Summary

Quantifies per-nucleotide crosslink-induced truncations (CITs) and crosslink-induced mutations (CIMs) from CLIP BAM alignments using Metadensity's truncation module, producing a diagnostic-signal table per transcript that can be background-normalized against size-matched inputs for metagene analysis.

## When to use

Use when starting from eCLIP/STAMP IP and size-matched input BAM files and you need nucleotide-resolution crosslink signal (truncation/mutation events) rather than raw read coverage, e.g. to build Metatruncate objects for metagene plots.

## When NOT to use

- When the desired signal is total read-coverage density from BAM alignments rather than crosslink sites — use the metadensity.readdensity module instead of truncation quantification.
- When a pre-computed WIG signal track already exists, since Metadensity can load it directly to speed up computation instead of re-deriving events from BAMs.
- When inputs are not CLIP IP/input sequencing data (e.g. already aggregated feature tables); Metadensity expects BAM/BIGWIG CLIP files registered in a dataloader and manifest.

## Inputs

- eCLIP IP BAM file (from ENCODE)
- size-matched input (background control) BAM file
- manifest .csv listing CLIP files with a unique UID per dataset
- config .ini file with species-specific annotation paths (e.g. config/hg38.ini: GENOME_FA, GENCODE, TRANSCRIPT, FEATURE)
- list of transcript IDs (optional; peak-containing transcripts used if omitted)

## Outputs

- Per-transcript, per-nucleotide table of crosslink-induced truncation (CIT) and mutation (CIM) counts
- Metatruncate signal objects that can be background-normalized (IP vs size-matched input) and assembled via metadensity.metadensity for metagene plots

## How to apply

Register the ENCODE eCLIP IP and size-matched input BAM files in a Metadensity dataloader and manifest (.csv with a UID per CLIP), and point the config to species-specific annotations (e.g. config/hg38.ini) so signals map onto GENCODE transcript coordinates. For each transcript, tally crosslink-induced truncation and mutation counts per nucleotide via the metadensity.truncation module (Metatruncate objects); set the --single end flag for single-end CLIP since it affects Metatruncate objects, and pass --truncation to use truncation sites instead of the entire read. Choose a background method (--background_method relative information, substraction, or None) to convert IP to INPUT, and a --stat (mean or median) with optional --normalization to average signal per transcript. The resulting per-transcript diagnostic-signal table can be saved and later reloaded as precomputed data for plotting.

## Related tools

- **Metadensity** (Provides the truncation/Metatruncate machinery for tallying CITs and CIMs per nucleotide and background-normalizing against size-matched inputs) — https://github.com/YeoLab/Metadensity
- **Jupyter** (Notebook interface for running the truncation workflow (e.g. 1_Example_on_SF3B4) and validating per-nucleotide signal tables) — https://github.com/algaebrown/Metadensity
- **Docker** (Prebuilt Metadensity environment on DockerHub for reproducible installation of the truncation pipeline) — https://hub.docker.com/repository/docker/algaebrown/metadensity

## Examples

```
python run_metadensity_vanilla.py -i ../test_data/menifest.csv -u SF3B4_test -o ../test_data --config=../config/hg38.ini --truncation
```

## Evaluation signals

- The per-transcript diagnostic-signal table exists, is readable, and contains per-nucleotide signal entries for each processed transcript
- Outputs match the worked example notebook (e.g. 1_Example_on_SF3B4) when run on the provided SF3B4 test data
- Toggling --truncation changes the signal from full-read coverage to truncation-site counts, confirming the truncation path is active
- Setting the correct --single end flag is reflected in how Metatruncate objects are built (single- vs paired-end read handling)
- Background normalization against size-matched inputs yields IP-vs-INPUT signal via one of the accepted --background_method values (relative information, substraction, None)

## Limitations

- Whether the CLIP library is single-end must be declared explicitly (-s/--single end), as it directly affects Metatruncate objects; a wrong flag mis-builds truncation signals
- Requires species-dependent annotations pointed to in config/*.ini (GENOME_FA, GENCODE, TRANSCRIPT, FEATURE are essential); missing DATADIR for new coordinates requires building one via parse_gencode_coords_into_data
- Background method choices are limited to relative information, substraction, or None; 'None' skips IP-to-INPUT normalization

## Evidence

- [methods] tally crosslink-induced truncation and mutation counts per nucleotide using the metadensity.truncation module: [evidence span withheld — anchor status: not_found]
- [readme] truncation flag in vanilla CLI: [evidence span withheld — anchor status: external_doc]
- [readme] single-end affects Metatruncate objects: [evidence span withheld — anchor status: external_doc]
- [abstract] background normalization against size-matched inputs: "Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features."
