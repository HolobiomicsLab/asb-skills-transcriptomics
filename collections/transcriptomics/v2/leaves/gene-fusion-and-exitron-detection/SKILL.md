---
name: gene-fusion-and-exitron-detection
description: Use when RNA-seq data must be mined for gene-fusion and exitron-splicing
  events as variant inputs to neoantigen prediction, beyond the SNV/indel sources
  most tools cover.
when_to_use_negative:
- Input is WES-only data with no RNA-seq reads — fusion and exitron detection requires
  transcriptomic evidence from chimeric/spliced alignments
- A validated VCF of fusion or exitron variants already exists and the goal is only
  peptide extraction and netMHCpan/netMHCIIpan prediction downstream
- The analysis goal is canonical SNV/indel calling (GATK Mutect2/transIndel) or alternative-splicing
  detection via SplAdder — these are separate ScanNeo2 variant modules with their
  own rules
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: STAR
  role: Realigned RNA-seq reads in chimeric mode (chimSegmentMin 20, chimScoreMin
    10) to expose candidate fusion-spanning reads for Arriba
  repo: https://github.com/alexdobin/STAR
- name: Arriba
  role: Detects gene fusions from the STAR realigned alignments, with the `maxevalue`
    cutoff for chance-expected supporting reads
  repo: https://github.com/suhrig/arriba
- name: ScanExitron
  role: Identifies exitron-splicing events using `ao` (minimum supporting reads) and
    `pso` (minimum exon-exclusion rate) thresholds
  repo: https://github.com/ylab-hi/ScanExitron
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:32e48ed9a1499d96aad46ecfe516fb7cf717e1d4c5a7bcfc0d1fa00e64041b60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - Arriba
  - BWA
  - GATK Mutect2
  - HLA-HD
  - STAR
  - ScanExitron
  - ScanNeo2
  - SplAdder
  - fastp
  - netMHCpan
  - transIndel
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-fusion-and-exitron-detection

## Summary

Detects two non-canonical neoantigen sources from RNA-seq data within the ScanNeo2 workflow: gene fusions via STAR chimeric realignment followed by Arriba, and exitron-splicing events via ScanExitron, producing per-group VCFs that feed downstream MHC-I/II epitope prediction.

## When to use

Use when RNA-seq data must be mined for gene-fusion and exitron-splicing events as variant inputs to neoantigen prediction, beyond the SNV/indel sources most tools cover.

## When NOT to use

- Input is WES-only data with no RNA-seq reads — fusion and exitron detection requires transcriptomic evidence from chimeric/spliced alignments
- A validated VCF of fusion or exitron variants already exists and the goal is only peptide extraction and netMHCpan/netMHCIIpan prediction downstream
- The analysis goal is canonical SNV/indel calling (GATK Mutect2/transIndel) or alternative-splicing detection via SplAdder — these are separate ScanNeo2 variant modules with their own rules

## Inputs

- fastp-trimmed RNA-seq FASTQ reads (paired-end)
- Sample sheet at config/samples.tsv and config/config.yml (reference genome, ENSEMBL release, threads, mapq 30, basequal 20)
- ENSEMBL reference genome and STAR index
- Per-sample/group grouping definition used to merge calls into per-group VCFs

## Outputs

- Gene-fusion calls (Arriba output) with supporting-read counts and maxevalue filtering
- Exitron-splicing calls (ScanExitron output) with read support and exon-exclusion rates
- Per-group merged VCFs for fusion and exitron variant types under results/<sample>/variants/
- Downstream <variant_type>_variant_effects.tsv and <variant_type>_<mhc_class>_neoepitopes.txt tables in results/<sample>/prioritization/

## How to apply

Within ScanNeo2, first preprocess RNA-seq reads with fastp, then align them in chimeric mode with STAR (chimSegmentMin 20, chimScoreMin 10) followed by BWA realignment, and apply Arriba to the resulting alignments for gene-fusion detection; the maxevalue configuration cutoff controls how many supporting reads may have arisen by chance. In parallel, call exitron-splicing events with ScanExitron, tuning the `ao` parameter (minimum number of reads supporting the exitron, set to 3 in the article's workflow) and the `pso` parameter (minimum exon-exclusion rate, set to 0.05). Rationale: existing neoantigen-prediction tools mainly focus on SNV/indel-derived neoantigens and overlook fusions and splicing events, whereas ScanNeo2 integrates these variant classes so their peptides can be predicted for MHC-I (lengths 8-11) and MHC-II (lengths 13-15 in the integration test). Merge the per-group fusion and exitron calls as VCFs under results/<sample>/variants/ before the peptide-extraction and netMHCpan/netMHCIIpan prediction steps.

## Related tools

- **STAR** (Realigned RNA-seq reads in chimeric mode (chimSegmentMin 20, chimScoreMin 10) to expose candidate fusion-spanning reads for Arriba) — https://github.com/alexdobin/STAR
- **Arriba** (Detects gene fusions from the STAR realigned alignments, with the `maxevalue` cutoff for chance-expected supporting reads) — https://github.com/suhrig/arriba
- **ScanExitron** (Identifies exitron-splicing events using `ao` (minimum supporting reads) and `pso` (minimum exon-exclusion rate) thresholds) — https://github.com/ylab-hi/ScanExitron

## Examples

```
snakemake --use-conda --cores 8 results/<sample>/variants/
```

## Evaluation signals

- Arriba fusion candidates survive the configured maxevalue cutoff on chance-expected supporting reads; in the article's workflow gene fusions are called with suppreads 2
- Exitron calls respect both ScanExitron thresholds: ao (minimum supporting reads) set to 3 and pso (minimum exon-exclusion rate) set to 0.05
- Per-group fusion and exitron VCFs exist and are merged under results/<sample>/variants/
- Fusion- and exitron-derived peptides appear in the downstream <variant_type>_variant_effects.tsv and <variant_type>_<mhc_class>_neoepitopes.txt tables, and in the combined <mhc_class>_neoepitopes_all.txt
- STAR chimeric parameters are confirmed as chimSegmentMin 20 and chimScoreMin 10 in the STAR realignment step before Arriba runs

## Limitations

- Most existing neoantigen tools focus on SNV/indel sources and often overlook gene fusions and alternative splicing, so no external benchmark of the fusion/exitron modules against such tools is described in the text
- The integration workflow does not pin a specific Snakemake version, so rule behavior for the fusion/exitron modules may vary across Snakemake releases
- No software license is stated for Arriba in the provided context (NOASSERTION), which matters for redistribution of the deployed environment

## Evidence

- [abstract] ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants: "Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants."
- [results] STAR realignment feeds Arriba fusion detection: "the aligned reads are first realigned using STAR (Dobin et al. 2013), and then subjected to Arriba"
- [methods] ScanExitron called for exitron events: "Exitron events are called using [ScanExitron](https://github.com/ylab-hi/ScanExitron)"
- [methods] ScanExitron ao/pso parameter semantics: "The former describes the minimum number of reads that support the exitron"
- [methods] Arriba maxevalue cutoff semantics: "`maxevalue` is a cutoff for the the number of supporting reads that are expected to have occurred by chance (e-value)"
- [readme] Arriba purpose from README: [evidence span withheld — anchor status: external_doc]
