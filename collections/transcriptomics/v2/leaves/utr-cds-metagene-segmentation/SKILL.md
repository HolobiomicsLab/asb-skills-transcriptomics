---
name: utr-cds-metagene-segmentation
description: Use when you need a canonical 5'-UTR-CDS-3'-UTR metagene of CLIP (eCLIP/STAMP)
  binding density on mature mRNA, rather than Metadensity's custom pre-mRNA feature
  models such as branchpoints or polyadenylation sites.
when_to_use_negative:
- When the target features are non-canonical pre-mRNA sites such as branchpoints,
  intronic polyadenylation sites, miRNA-hosting genes, or snoRNA/lncRNA loci — Metadensity
  supports these but this skill covers only the mature 5'UTR-CDS-3'UTR model
- When you need per-feature scaling by length (e.g. transcriptome-wide scaled coverage
  maps) — Metadensity deliberately preserves near-nucleotide resolution instead of
  scaling features by length
- When you lack a size-matched input control for the eCLIP experiment, since IP signal
  normalization against background is central to the metagene computation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3524
tools:
- name: Metadensity
  role: Computes background-normalized CLIP read density and windows it over 5'UTR/CDS/3'UTR
    features to build the mature-mRNA metagene
  repo: https://github.com/YeoLab/Metadensity
- name: Jupyter
  role: Advanced usage environment for running Metadensity notebooks and rendering
    the metagene with metadensity.plotd
  repo: https://metadensity.readthedocs.io/en/latest/tutorial.html
- name: conda
  role: Installs the Metadensity environment from environment.yaml before running
    the metagene computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac083
    title: 'Metadensity: a background-aware python pipeline for summarizing CLIP signals
      on various transcriptomic sites'
  source_content_hash: sha256:a5a25e2b2262fc84ddace18e6d445e5dbb5d7a3749e2ba80a7f678468cf44624
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
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# utr-cds-metagene-segmentation

## Summary

Compute the canonical mature-mRNA 5'-UTR-CDS-3'-UTR metagene model with Metadensity — the regular metagene reported in Supplementary Fig. S1 — by windowing input-normalized eCLIP signal over UTR/CDS transcript features at near-nucleotide resolution and averaging across transcripts.

## When to use

Use when you need a canonical 5'-UTR-CDS-3'-UTR metagene of CLIP (eCLIP/STAMP) binding density on mature mRNA, rather than Metadensity's custom pre-mRNA feature models such as branchpoints or polyadenylation sites.

## When NOT to use

- When the target features are non-canonical pre-mRNA sites such as branchpoints, intronic polyadenylation sites, miRNA-hosting genes, or snoRNA/lncRNA loci — Metadensity supports these but this skill covers only the mature 5'UTR-CDS-3'UTR model
- When you need per-feature scaling by length (e.g. transcriptome-wide scaled coverage maps) — Metadensity deliberately preserves near-nucleotide resolution instead of scaling features by length
- When you lack a size-matched input control for the eCLIP experiment, since IP signal normalization against background is central to the metagene computation

## Inputs

- eCLIP IP and size-matched input signal tracks downloaded from ENCODE
- Manifest .csv file listing all CLIP files with a unique uid per CLIP dataset
- config/*.ini file (e.g. hg38.ini) pointing to GENOME_FA and GENCODE-derived FEATURE (exon/CDS/UTR) and TRANSCRIPT gff3 annotations
- Optional transcript ID list; if omitted, peak-containing transcripts are used

## Outputs

- Mature-mRNA metagene plot (e.g. metagene_mature_mrna.png) segmented into 5'-UTR, CDS, and 3'-UTR regions
- Windowed, background-normalized read-density arrays underlying the metagene

## How to apply

Configure Metadensity with paths to the hg38 genome FASTA (GENOME_FA) and the GENCODE-derived FEATURE/TRANSCRIPT gff3 annotations containing exon, CDS, and UTR entries in a config/*.ini file (e.g. config/hg38.ini). Download eCLIP IP and size-matched input tracks from ENCODE, build the dataloader and manifest (.csv), and use the metadensity.metadensity module to compute read density normalized against the size-matched input background (background_method options: relative information, subtraction, or None). Window the normalized signal over the user-defined 5'UTR, CDS, and 3'UTR features, preserving near-nucleotide resolution by NOT scaling features by length. Average the windowed signals across the transcript set (mean or median) and render with the metadensity.plotd module, saving the mature-mRNA metagene figure. Either supply a transcript list or default to peak-containing transcripts via the -t option.

## Related tools

- **Metadensity** (Computes background-normalized CLIP read density and windows it over 5'UTR/CDS/3'UTR features to build the mature-mRNA metagene) — https://github.com/YeoLab/Metadensity
- **Jupyter** (Advanced usage environment for running Metadensity notebooks and rendering the metagene with metadensity.plotd) — https://metadensity.readthedocs.io/en/latest/tutorial.html
- **conda** (Installs the Metadensity environment from environment.yaml before running the metagene computation)

## Examples

```
python run_metadensity_vanilla.py -i ../test_data/menifest.csv -u SF3B4_test -o ../test_data --config=../config/hg38.ini
```

## Evaluation signals

- metagene_mature_mrna.png exists and displays the expected UTR/CDS-segmented metagene structure, consistent with the reported Metadensity output (e.g. the SF3B4 example)
- The IP signal in the arrays reflects normalization against the size-matched input (background_method set to relative information or subtraction, not None)
- The x-axis segmentation is 5'UTR → CDS → 3'UTR with features NOT scaled by length (near-nucleotide resolution preserved)
- The --stat choice (mean or median) was applied consistently when averaging signals across the transcript set

## Limitations

- Requires species-dependent genome coordinate annotations (GENOME_FA, GENCODE gff3, canonical-transcript FEATURE file) configured in config/*.ini; new coordinates need a DATADIR built via parse_gencode_coords_into_data
- Feature annotations (TRANSCRIPT, FEATURE) must be generated from GENCODE plus the UCSC canonical transcript list via gencode_canon_filtering.sh or downloaded pre-computed
- Background normalization choice matters: relative information, subtraction, or None are the supported options and the result depends on this decision

## Evidence

- [abstract] Metadensity allows users to input features such as branchpoints and preserves the near-nucleotide resolution of CLIP technologies by not scaling the features by length.: "Metadensity allows users to input features such as branchpoints and preserves the near-nucleotide resolution of CLIP technologies by not scaling the features by length."
- [abstract] Metadensity normalizes IP with size-matched inputs then windows in user-defined features and averages across transcripts.: "Metadensity normalizes immunoprecipitated libraries with background controls, such as size-matched inputs, then windowing in various user-defined features. Finally, the signals are averaged across a"
- [intro] You need to point those files in `config/*.ini`. see `config/hg38.ini` as an example.: "You need to point those files in `config/*.ini`. see `config/hg38.ini` as an example."
- [methods] you need to download eCLIP data from the ENCODE project, create your dataloader and menifest!: "you need to download eCLIP data from the ENCODE project, create your dataloader and menifest!"
- [readme] gff3 annotation from GENCODE, containing only "exon", "CDS", "UTR" and created "introns": [evidence span withheld — anchor status: external_doc]
