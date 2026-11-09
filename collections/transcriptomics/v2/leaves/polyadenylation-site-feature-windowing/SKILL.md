---
name: polyadenylation-site-feature-windowing
description: Use when eCLIP/IP signal should be visualized around polyadenylation
  sites (e.g. reproducing Supplementary Figs. S2/S3-style poly(A)-site metagenes)
  rather than around generic UTR/CDS transcript regions.
when_to_use_negative:
- The goal is the standard 5'-UTR-CDS-3'-UTR metagene rather than a user-defined site
  such as polyA or branchpoints (use Metadensity's default feature model).
- A precomputed DATADIR pickle does not exist for the genome/coordinate set and no
  custom feature can be built (see parse_gencode_coords_into_data notebook first).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: Metadensity
  role: pipeline that windows CLIP signal on user-defined features (here polyA sites)
    and normalizes IP against size-matched inputs
  repo: https://github.com/YeoLab/Metadensity
- name: metadensity.readdensity
  role: computes per-window read densities used for the poly(A)-site metagene
- name: metadensity.metadensity
  role: computes the averaged, background-normalized metadensity signal across transcripts
- name: metadensity.plotd
  role: renders the metagene density plot figure
- name: Jupyter
  role: notebook environment in which the custom polyA feature is built and the plot
    is generated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac083
    title: 'Metadensity: a background-aware python pipeline for summarizing CLIP signals
      on various transcriptomic sites'
  source_content_hash: sha256:544ce0f2f0eb88f1af819e339b7916a654f94e408d640303b3aa7b951e24b8d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YeoLab/Metadensity
  tools:
  - Jupyter
  - Metadensity
  - metadensity.metadensity
  - metadensity.plotd
  - metadensity.readdensity
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# polyadenylation-site-feature-windowing

## Summary

Define fixed-width windows around polyadenylation (polyA) sites on pre-mRNAs as a custom Metadensity feature so eCLIP read densities can be summarized as a background-aware metagene plot, complementing the standard 5'-UTR-CDS-3'-UTR model.

## When to use

Use when eCLIP/IP signal should be visualized around polyadenylation sites (e.g. reproducing Supplementary Figs. S2/S3-style poly(A)-site metagenes) rather than around generic UTR/CDS transcript regions.

## When NOT to use

- The goal is the standard 5'-UTR-CDS-3'-UTR metagene rather than a user-defined site such as polyA or branchpoints (use Metadensity's default feature model).
- A precomputed DATADIR pickle does not exist for the genome/coordinate set and no custom feature can be built (see parse_gencode_coords_into_data notebook first).

## Inputs

- POLYA annotation of polyA sites and signals from polyASite (gff3-style, set in config/*.ini)
- hg38 genome coordinate annotations (GENOME_FA, GENCODE, TRANSCRIPT, FEATURE) referenced via config/hg38.ini
- ENCODE eCLIP IP BAM files plus size-matched input background controls, loaded via the Metadensity dataloader/manifest (e.g. menifest.csv)

## Outputs

- Poly(A)-site metagene density plot (per-window averaged IP-vs-input normalized signal)
- Per-window density value table underlying the metagene plot

## How to apply

Configure Metadensity with hg38 annotation paths in config/*.ini, including the POLYA entry pointing to polyASite atlas annotations of polyA sites and signals. Build the custom polyA-site feature following the 0_customize_feature_example_intronic_polyA tutorial by defining fixed-width windows around poly(A) site coordinates on pre-mRNAs; preserve the native feature geometry instead of scaling features by length, which retains the near-nucleotide resolution of CLIP. Compute per-window read densities with metadensity.readdensity/metadensity.metadensity, normalizing the immunoprecipitated libraries against size-matched input background controls. Average the normalized signal across the transcript set and render the metagene with metadensity.plotd in a Jupyter notebook, then export the figure and per-window density value table.

## Related tools

- **Metadensity** (pipeline that windows CLIP signal on user-defined features (here polyA sites) and normalizes IP against size-matched inputs) — https://github.com/YeoLab/Metadensity
- **metadensity.readdensity** (computes per-window read densities used for the poly(A)-site metagene)
- **metadensity.metadensity** (computes the averaged, background-normalized metadensity signal across transcripts)
- **metadensity.plotd** (renders the metagene density plot figure)
- **Jupyter** (notebook environment in which the custom polyA feature is built and the plot is generated)

## Examples

```
python run_metadensity_vanilla.py -i ../test_data/menifest.csv -u SF3B4_test -o ../test_data --config=../config/hg38.ini
```

## Evaluation signals

- Rendered poly(A)-site metagene qualitatively matches the enrichment patterns in Supplementary Figures S2 and S3 (no numeric threshold is given in the section text).
- Feature windows are fixed-width around poly(A) site coordinates, i.e. the axis preserves native geometry and is not length-scaled.
- Plotted signal is IP normalized against size-matched input background, not raw IP counts.
- The exported per-window density table values agree with the y-extent of the rendered metadensity.plotd figure.

## Limitations

- No numeric acceptance threshold is reported for matching Supplementary Figures S2/S3; validation is qualitative.
- POLYA annotation is marked 'Essential to run: NO' in the README, so polyA data must be supplied explicitly in config/*.ini.
- For coordinates without precomputed DATADIR pickle files, users must build data themselves (parse_gencode_coords_into_data notebook).

## Evidence

- [abstract] Metadensity allows users to input features such as branchpoints and preserves the near-nucleotide resolution of CLIP technologies by not scaling the features by length.: "Metadensity allows users to input features such as branchpoints and preserves the near-nucleotide resolution of CLIP technologies by not scaling the features by length."
- [abstract] Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features.: "Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features."
- [methods] Create customized features: intronic polyadenylation, miRNA hosting genes: "Create customized features: intronic polyadenylation, miRNA hosting genes"
- [readme] POLYA            | annotation of polyA sites and signals from polyASite                                     | https://polyasite.unibas.ch/atlas | NO: [evidence span withheld — anchor status: external_doc]
- [readme] You need to point those files in `config/*.ini`. see `config/hg38.ini` as an example.: "You need to point those files in `config/*.ini`. see `config/hg38.ini` as an example."
