---
name: ecclip-alignment-signal-extraction
description: Use when starting from eCLIP/STAMP IP and size-matched input BAM files
  and you need per-nucleotide transcript-level diagnostic signals (read coverage or
  CIT/CIM summation) before metagene averaging.
when_to_use_negative:
- Input is already a precomputed Metadensity signal/feature table — skip extraction
  and load it directly (Heatmap_from_precomputed_data workflow)
- The goal is only plotting aggregated metagenes over branchpoints, polyA sites, miRNA-hosting
  genes or other user-defined features without recomputing signals from BAMs
- Your coordinate system/species has no prepared DATADIR and you have not yet built
  annotations via parse_gencode_coords_into_data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Metadensity
  role: Pipeline whose readdensity, truncation and metadensity modules extract per-transcript
    read-coverage and CIT/CIM signals from BAM alignments
  repo: https://github.com/YeoLab/Metadensity
- name: Jupyter
  role: Notebook environment for running the example signal-extraction workflows (e.g.
    1_Example_on_SF3B4)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac083
    title: 'Metadensity: a background-aware python pipeline for summarizing CLIP signals
      on various transcriptomic sites'
  source_content_hash: sha256:3107914898243a3e89bd417f2c43557c3d2819ebc991d700d398fbe4ddf78764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YeoLab/Metadensity
  tools:
  - Jupyter
  - Metadensity
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# ecclip-alignment-signal-extraction

## Summary

Extracts per-transcript CLIP diagnostic signals from eCLIP BAM/BIGWIG alignments using Metadensity: read-coverage density tracks (metadensity.readdensity) and crosslink-induced truncation/mutation counts (metadensity.truncation), with a precomputed WIG track as a faster alternative. This is the first pipeline stage that supplies the per-nucleotide signal table later aggregated into metagene plots.

## When to use

Use when starting from eCLIP/STAMP IP and size-matched input BAM files and you need per-nucleotide transcript-level diagnostic signals (read coverage or CIT/CIM summation) before metagene averaging.

## When NOT to use

- Input is already a precomputed Metadensity signal/feature table — skip extraction and load it directly (Heatmap_from_precomputed_data workflow)
- The goal is only plotting aggregated metagenes over branchpoints, polyA sites, miRNA-hosting genes or other user-defined features without recomputing signals from BAMs
- Your coordinate system/species has no prepared DATADIR and you have not yet built annotations via parse_gencode_coords_into_data

## Inputs

- eCLIP IP BAM/BIGWIG alignment files (e.g. from the ENCODE project)
- size-matched input (background control) BAM/BIGWIG files
- manifest .csv listing all CLIP files with unique IDs (uid)
- Species-specific annotation config file (e.g. config/hg38.ini) with GENOME_FA, GENCODE, TRANSCRIPT and FEATURE paths
- Optional: pre-computed WIG signal track as a faster alternative to BAM-derived coverage

## Outputs

- Per-transcript read-coverage density tracks (metadensity.readdensity)
- Per-nucleotide crosslink-induced truncation (CIT) and mutation (CIM) count tables (metadensity.truncation)
- Assembled per-transcript diagnostic-signal table (metadensity.metadensity) saved for later loading as precomputed data

## How to apply

Register the eCLIP IP and size-matched input files downloaded from ENCODE in a Metadensity dataloader and manifest (.csv), then point annotation paths in the species config (e.g. config/hg38.ini) so signals map onto transcript coordinates. For each transcript, compute read-coverage tracks with the metadensity.readdensity module, and tally crosslink-induced truncations and mutations per nucleotide with the metadensity.truncation module — the --truncation flag switches Metatruncate to use truncations instead of entire reads. Assemble the per-transcript diagnostic-signal table via the metadensity.metadensity module so it can be saved and later loaded as precomputed data (Heatmap_from_precomputed_data workflow). If computation speed matters, substitute a pre-computed WIG track in place of raw BAM coverage summation. Decide the background strategy explicitly: IP-to-INPUT conversion can be relative information, subtraction, or None, and transcript selection falls back to peak-containing transcripts when no transcript list is given.

## Related tools

- **Metadensity** (Pipeline whose readdensity, truncation and metadensity modules extract per-transcript read-coverage and CIT/CIM signals from BAM alignments) — https://github.com/YeoLab/Metadensity
- **Jupyter** (Notebook environment for running the example signal-extraction workflows (e.g. 1_Example_on_SF3B4))

## Examples

```
cd scripts/; python run_metadensity_vanilla.py -i ../test_data/menifest.csv -u SF3B4_test -o ../test_data --config=../config/hg38.ini --truncation
```

## Evaluation signals

- The per-transcript signal table exists on disk, is readable, and contains per-nucleotide signal entries for every processed transcript
- Coverage tracks from metadensity.readdensity match per-transcript BAM read depth at transcript coordinates resolved via the config/hg38.ini annotations
- Truncation/CIM counts from metadensity.truncation agree with the --truncation mode requested (truncations instead of entire reads)
- IP signal shows normalization against the size-matched input control using the chosen background_method (relative information, substraction, or None)
- Outputs reproduce the per-transcript signal structure shown in the example notebooks (e.g. 1_Example_on_SF3B4)

## Limitations

- Signals are species/coordinate-dependent: annotation paths must be set per species in a separate config/*.ini, and non-human coordinates require building DATADIR yourself via parse_gencode_coords_into_data
- Using raw BAM coverage is slower; only a pre-computed WIG track (or precomputed DATADIR pickle) speeds computation
- Single-end vs paired-end CLIP affects Metatruncate objects (-s flag), so incorrect endness in the manifest changes truncation extraction

## Evidence

- [full_text] The first pipeline stage extracts CLIP diagnostic signals from BAM/BIGWIG files for each transcript, using either read coverage or the summation of crosslink-induced truncations (CITs) and mutations (CIMs); a pre-computed WIG track can be used instead to speed up computation.: [evidence span withheld — anchor status: not_found]
- [abstract] Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features. Finally, the signals are averaged across a: "Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features."
- [methods] you need to download eCLIP data from the ENCODE project, create your dataloader and menifest!: "you need to download eCLIP data from the ENCODE project, create your dataloader and menifest!"
- [readme] You need to point those files in `config/*.ini`. see `config/hg38.ini` as an example. These information are genome coordinate, species dependent. So you can keep each species with a seperate `.ini`.: "These information are genome coordinate, species dependent. So you can keep each species with a seperate `.ini`."
- [readme] --truncation          Use truncation instead of the entire read: [evidence span withheld — anchor status: external_doc]
