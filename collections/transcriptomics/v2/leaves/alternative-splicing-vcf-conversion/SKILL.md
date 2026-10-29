---
name: alternative-splicing-vcf-conversion
description: Use when RNA-seq alignments are available and the neoantigen-prediction
  goal requires capturing alternative-splicing-derived variants (beyond SNV/indel
  calling), i.e., when the altsplicing module's activate flag is set in config/config.yml.
when_to_use_negative:
- The variant of interest is an exitron-splicing event — use the ScanExitron module
  (exitrons.vcf.gz) instead of SplAdder conversion
- Only SNV or short-indel calling is needed — the GATK Mutect2 module on DNA/RNA BAMs
  covers this without splice-graph conversion
- RNA-seq data are absent or too low quality to support splice-junction quantification
  (SplAdder depends on informative alignments)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3308
tools:
- name: ScanNeo2
  role: Wraps the altsplicing module (VCF conversion, GRP/SRC augmentation, merging)
    inside its Snakemake workflow
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: SplAdder
  role: Detects alternative splicing events from the RNA-seq alignment whose splice-graph
    output is converted to VCF
- name: Snakemake
  role: Workflow engine that gates the module on its activate flag and orchestrates
    conversion/merge rules
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:6344fe2c035430293a7b8550a39a4a20f25ca889e7bacbc8a0083f1cc16fb20d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - Arriba
  - GATK Mutect2
  - ScanExitron
  - ScanNeo2
  - Snakemake
  - SplAdder
  - transIndel
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# alternative-splicing-vcf-conversion

## Summary

Converts SplAdder splice-graph output from RNA-seq alignments into standardized VCFs of alternative-splicing-derived neoantigen candidates within the ScanNeo2 Snakemake workflow, annotating each record with GRP (group) and SRC (caller source) INFO keys so downstream peptide extraction can trace neoantigen provenance.

## When to use

Use when RNA-seq alignments are available and the neoantigen-prediction goal requires capturing alternative-splicing-derived variants (beyond SNV/indel calling), i.e., when the altsplicing module's activate flag is set in config/config.yml.

## When NOT to use

- The variant of interest is an exitron-splicing event — use the ScanExitron module (exitrons.vcf.gz) instead of SplAdder conversion
- Only SNV or short-indel calling is needed — the GATK Mutect2 module on DNA/RNA BAMs covers this without splice-graph conversion
- RNA-seq data are absent or too low quality to support splice-junction quantification (SplAdder depends on informative alignments)

## Inputs

- RNA-seq BAM aligned against the reference genome (BWA/STAR pipeline output)
- config/config.yml with altsplicing module `activate` flag and parameters (confidence, iterations, edgelimit)
- Sample sheet defining groups
- SplAdder splice-graph output (per-group)

## Outputs

- Per-group VCFs of alternative-splicing events annotated with GRP and SRC INFO keys
- Merged altsplicing.vcf.gz under results/<sample>/variants/
- Downstream candidate peptides (8–11 aa for MHC I, 13–25 aa for MHC II) attributable to the alternative-splicing source

## How to apply

Run SplAdder on the RNA-seq BAM with confidence 3, 5 iterations, and edgelimit 250 — the confidence parameter controls how strongly input alignments are filtered before new nodes and edges are added to the splice graph. Convert the SplAdder splice-graph output into per-group VCFs representing the splicing-derived transcript junctions. Augment each per-group VCF with GRP (sample group) and SRC (caller source) INFO keys, then sort and merge the per-group files into a single altsplicing.vcf.gz under results/<sample>/variants/. This VCF feeds the downstream transcript/peptide extraction stage (8–11 mers for MHC I, 13–25 mers for MHC II), where alternative splicing contributes one of the single-source neoantigen categories ScanNeo2 reports alongside exitron-splicing, gene fusion, indels, and SNVs.

## Related tools

- **ScanNeo2** (Wraps the altsplicing module (VCF conversion, GRP/SRC augmentation, merging) inside its Snakemake workflow) — https://github.com/ylab-hi/ScanNeo2/
- **SplAdder** (Detects alternative splicing events from the RNA-seq alignment whose splice-graph output is converted to VCF)
- **Snakemake** (Workflow engine that gates the module on its activate flag and orchestrates conversion/merge rules)

## Examples

```
snakemake --use-conda --cores 8 results/<sample>/variants/altsplicing.vcf.gz
```

## Evaluation signals

- altsplicing.vcf.gz exists under results/<sample>/variants/ whenever the altsplicing module's activate flag is on
- Every VCF record carries the GRP (group) and SRC (caller source) INFO keys with SRC identifying the alternative-splicing caller
- SplAdder ran with the configured confidence (3), iterations (5), and edgelimit (250) — spot-check rule log for these parameter values
- Merged VCF is sorted and contains records pooled from all expected per-group VCFs, not a subset
- Peptides traced from this source appear among the single-source candidate categories (5671–34981 per patient in the TESLA benchmark) in the final report

## Limitations

- Somatic alternative-splicing calls depend entirely on RNA-seq alignment quality; SplAdder's confidence parameter trades sensitivity against spurious graph nodes/edges
- The module only captures canonical alternative-splicing events — exitrons require the separate ScanExitron module, and neither covers gene fusions or indels
- Per the README, SplAdder conversion is one module among several; no benchmark/validation of altsplicing-specific calls in isolation is described beyond the aggregate TESLA results

## Evidence

- [results] subjected to SplAdder (Kahles et al. 2016) for the detection of alternative splicing events: "the processed reads are subjected to SplAdder (Kahles et al. 2016) for the detection of alternative splicing events"
- [methods] confidence filters input alignments before splice graph extension: "the parameter `confidence` determines how strongly input alignments are filtered before new nodes and edges are added"
- [abstract] ScanNeo2 integrates canonical splicing among multiple variant sources: "Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants."
- [results] alternative splicing is a distinct single-source neoantigen category in the benchmark: "5671–34 981 candidate neoantigens originate from a single source, such as alternative splicing, exitron-splicing, gene fusion, indels, or SNVs"
