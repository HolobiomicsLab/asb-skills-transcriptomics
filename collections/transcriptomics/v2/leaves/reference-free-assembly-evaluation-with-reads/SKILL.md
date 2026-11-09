---
name: reference-free-assembly-evaluation-with-reads
description: Use when a de novo transcriptome assembly (FASTA) must be quality-assessed
  but no reference genome is available, and the raw sequencing reads (FASTQ) are at
  hand.
when_to_use_negative:
- A reference genome is available, so reference-based assessment or RSEM-EVAL-style
  methods can be used instead
- The input is already a filtered or processed assembly for which the unfiltered baseline
  TransRate score cannot be established
- Only the assembly (no sequencing reads) is available, since TransRate requires the
  reads
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: TransRate
  role: Reference-free quality assessment of the assembly from reads; also provides
    the TransRate-C final filtering step and TransRate-Q re-assessment
- name: Bellerophon
  role: Pipeline wrapper that automates the TransRate/BUSCO assessment plus TPM and
    CD-HIT-EST filtering and generates the results report
  repo: github.com/JesseKerkvliet/Bellerophon
- name: CD-HIT-EST
  role: Removes highly identical redundant contigs at a 95% identity cut-off before
    quality re-assessment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:3e04508ab77ff0aab7d8e7d2c8a181ba88229105814f6366c43c4dba67b65a22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# reference-free assembly evaluation with reads

## Summary

Evaluate de novo transcriptome assembly quality without a reference genome by running TransRate on the sequencing reads and the assembly itself, optionally grounded with a BUSCO ortholog score, as done in the Bellerophon pipeline.

## When to use

Use when a de novo transcriptome assembly (FASTA) must be quality-assessed but no reference genome is available, and the raw sequencing reads (FASTQ) are at hand.

## When NOT to use

- A reference genome is available, so reference-based assessment or RSEM-EVAL-style methods can be used instead
- The input is already a filtered or processed assembly for which the unfiltered baseline TransRate score cannot be established
- Only the assembly (no sequencing reads) is available, since TransRate requires the reads

## Inputs

- De novo transcriptome assembly in FASTA format (e.g. Trinity output)
- Paired-end RNA-Seq reads in FASTQ format (left/right files)

## Outputs

- TransRate quality score for the unfiltered assembly (ground quality score)
- TransRate-Q scores for TPM- and CD-HIT-EST-filtered assemblies
- TransRate-C filtered transcriptome assembly
- BUSCO completeness assessment of the filtered assembly
- Automatically generated Bellerophon results report

## How to apply

Run TransRate with only the paired sequencing reads and the assembled transcriptome as inputs, since TransRate is truly reference-free and maps reads back to the contigs to score assembly quality; optionally run BUSCO (e.g. with the Insecta odb9 ortholog set) on the same assembly to establish a ground quality score. Execute this assessment before any filtering to baseline the unfiltered assembly, and re-run TransRate-Q after each filtering stage (e.g. after TPM filtering with cut-off 1 and after CD-HIT-EST clustering at 95% identity) to track score changes. Compare the resulting TransRate scores across filtering variants to guide filtering decisions — in the Bellerophon study the highest score (0.319) came from CD-HIT-EST-only filtering, but the default order was set to TPM then CD-HIT-EST because ORF-length filtering removed too many transcripts. Finish by running TransRate's filtering capabilities (TransRate-C) as the final step, which in the reported experiments removed the largest share of chimeras (42.9%).

## Related tools

- **TransRate** (Reference-free quality assessment of the assembly from reads; also provides the TransRate-C final filtering step and TransRate-Q re-assessment)
- **Bellerophon** (Pipeline wrapper that automates the TransRate/BUSCO assessment plus TPM and CD-HIT-EST filtering and generates the results report) — github.com/JesseKerkvliet/Bellerophon
- **CD-HIT-EST** (Removes highly identical redundant contigs at a 95% identity cut-off before quality re-assessment)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- The report confirms all steps completed in the default order: (1) TPM filtering then (2) CD-HIT-EST, with TransRate-C applied last
- BUSCO and TransRate produce a ground quality score for the unfiltered assembly before any filtering step runs
- TransRate scores are comparable across filter variants (e.g. CD-HIT-EST-only reached 0.319 in the published experiment)
- The final filtered assembly reflects the reported chimera-removal behavior, with TransRate-C removing the largest share of chimeras (42.9%)
- The run completes in the Vagrant VM or conda environment without backwards-compatibility problems with TransRate

## Limitations

- Bellerophon may erroneously remove contigs from alternative splicing that TransRate misclassifies as chimeras when they are unevenly expressed along their sequence
- Custom order of filtering steps is disabled in the current version of the pipeline
- No tool version numbers are stated for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester in the methods
- The Vagrant VM's disk size is not large enough for most full transcriptome analyses, so outputs should go to the shared /vagrant/ directory

## Evidence

- [intro] TransRate is truly reference‐free and only requires the sequencing reads and the assembled transcriptome.: "TransRate is truly reference‐free and only requires the sequencing reads and the assembled transcriptome."
- [methods] Busco and Transrate are used to establish a ground quality score of the unfiltered assembly.: "(a) Busco and Transrate are used to establish a ground quality score of the unfiltered assembly."
- [methods] To assess the quality of the fully filtered transcriptome, TransRate-Q and BUSCO are executed using this filtered assembly.: [evidence span withheld — anchor status: not_found]
- [results] The filter with the highest resulting TransRate score was the filter with only CD‐HIT‐EST applied: "The filter with the highest resulting TransRate score was the filter with only CD‐HIT‐EST applied (Figure 2a, experiment number 3), resulting in a TransRate score of 0.319."
- [readme] python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun: "python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun"
