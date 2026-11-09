---
name: eclip-signal-metagene-visualization
description: Use when you have eCLIP IP and size-matched input bigWig/bedGraph signal
  tracks from ENCODE plus GENCODE annotations and need a metagene density summary
  over transcript features to infer RBP binding localization.
when_to_use_negative:
- When the question concerns single-transcript binding maps rather than averaged signal
  across a set of transcripts
- When no size-matched input background control exists for the CLIP experiment, since
  IP-to-input normalization is central to Metadensity
- When annotations for a new species/coordinate system are needed but no precomputed
  DATADIR pickle exists and building one (parse_gencode_coords_into_data) is out of
  scope
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3523
tools:
- name: Metadensity
  role: 'Main pipeline: computes background-normalized read density, windows signal
    over transcript features, and averages across transcripts for metagene plotting'
  repo: https://github.com/YeoLab/Metadensity
- name: conda
  role: Creates the Metadensity runtime environment from environment.yaml
- name: Jupyter
  role: Notebook interface for advanced metadensity usage (dataloader construction,
    plotting, comparing transcript groups)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac083
    title: 'Metadensity: a background-aware python pipeline for summarizing CLIP signals
      on various transcriptomic sites'
  source_content_hash: sha256:01bc89978847713a208408ec6fc9d80f0f9958cce4e55da076670d193be622c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YeoLab/Metadensity
  tools:
  - Jupyter
  - Metadensity
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# eclip-signal-metagene-visualization

## Summary

Compute background-aware eCLIP/STAMP metagene plots with Metadensity, normalizing IP read density against size-matched input controls and averaging windowed signal over user-defined transcript features (e.g. 5'UTR-CDS-3'UTR metagene) at near-nucleotide resolution without scaling features by length.

## When to use

Use when you have eCLIP IP and size-matched input bigWig/bedGraph signal tracks from ENCODE plus GENCODE annotations and need a metagene density summary over transcript features to infer RBP binding localization.

## When NOT to use

- When the question concerns single-transcript binding maps rather than averaged signal across a set of transcripts
- When no size-matched input background control exists for the CLIP experiment, since IP-to-input normalization is central to Metadensity
- When annotations for a new species/coordinate system are needed but no precomputed DATADIR pickle exists and building one (parse_gencode_coords_into_data) is out of scope

## Inputs

- eCLIP (or STAMP/Par-CLIP) immunoprecipitation signal tracks from ENCODE
- size-matched input (background control) signal tracks
- menifest.csv listing CLIP files with unique IDs (uid)
- config/*.ini file with genome-specific annotation paths (e.g. config/hg38.ini)
- GENOME_FA genome FASTA (e.g. hg38)
- GENCODE-derived TRANSCRIPT and FEATURE gff3 annotations (exon, CDS, UTR, introns) from canonical transcripts
- Optional transcript ID list to restrict the metagene computation (defaults to peak-containing transcripts)

## Outputs

- Metagene density plot (e.g. metagene_mature_mrna.png) showing averaged normalized signal across 5'UTR-CDS-3'UTR segments
- Per-feature windowed signal arrays saved alongside figures in the output directory
- Background-normalized IP-over-input density per transcript feature

## How to apply

First configure species-specific paths (GENOME_FA, GENCODE, TRANSCRIPT, FEATURE .gff3) in a config/*.ini file such as config/hg38.ini. Download eCLIP immunoprecipitation and size-matched input tracks from ENCODE, then build the Metadensity dataloader and manifest (menifest.csv listing CLIP files with unique UIDs). Use the metadensity.metadensity and metadensity.readdensity modules to compute read density, normalizing IP against the size-matched input background (choose 'relative information', 'substraction', or None), optionally using mean or median statistics and truncation sites instead of entire reads. Window the normalized signal over user-defined transcript features (5'UTR, CDS, 3'UTR, or custom features like branchpoints and polyA sites), preserving near-nucleotide resolution by not scaling features by length. Finally average the windowed signals across the transcript set and render with the metadensity.plotd module. Either run the vanilla CLI (run_metadensity_vanilla.py) or the Jupyter notebook workflow for advanced control.

## Related tools

- **Metadensity** (Main pipeline: computes background-normalized read density, windows signal over transcript features, and averages across transcripts for metagene plotting) — https://github.com/YeoLab/Metadensity
- **conda** (Creates the Metadensity runtime environment from environment.yaml)
- **Jupyter** (Notebook interface for advanced metadensity usage (dataloader construction, plotting, comparing transcript groups))

## Examples

```
python run_metadensity_vanilla.py -i ../test_data/menifest.csv -u SF3B4_test -o ../test_data --config=../config/hg38.ini --background_method=relative_information --stat=mean
```

## Evaluation signals

- metagene_mature_mrna.png (or equivalent output PNG) exists and displays the expected UTR/CDS-segmented metagene structure consistent with the reported SF3B4 example
- The IP signal is visibly normalized against the size-matched input background rather than showing raw read density
- Feature windows retain near-nucleotide resolution (features are not scaled to equal length in the plot)
- Output directory contains both figures and the underlying windowed signal arrays for the chosen --stat (mean/median) and --background_method settings
- The run used a valid species config (e.g. config/hg38.ini) and the manifest uid resolved to the intended CLIP dataset

## Limitations

- Requires several annotations (GENOME_FA, GENCODE, TRANSCRIPT, FEATURE) pointed to in config/*.ini; missing annotation paths will block execution
- Background methods are limited to 'relative information', 'substraction', or None; choice affects interpretability of the metagene
- Custom coordinate systems without precomputed DATADIR pickles require building the parsed data yourself via the parse_gencode_coords_into_data notebook
- Whether the CLIP is single-end (-s flag) and whether truncation sites vs. entire reads are used (--truncation) change the Metatruncate object semantics and results

## Evidence

- [abstract] Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features. Finally, the signals are averaged across a: "Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features."
- [abstract] Metadensity allows users to input features such as branchpoints and preserves the near-nucleotide resolution of CLIP technologies by not scaling the features by length.: "preserves the near-nucleotide resolution of CLIP technologies by not scaling the features by length"
- [methods] you need to download eCLIP data from the ENCODE project, create your dataloader and menifest!: "you need to download eCLIP data from the ENCODE project, create your dataloader and menifest!"
- [readme] Metadensity requires several annotations to work. You need to point those files in `config/*.ini`. see `config/hg38.ini` as an example.: "Metadensity requires several annotations to work. You need to point those files in `config/*.ini`."
- [readme] This will run the most vanilla functions of Metadensity: [evidence span withheld — anchor status: external_doc]
