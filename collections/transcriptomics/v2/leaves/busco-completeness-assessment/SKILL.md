---
name: busco-completeness-assessment
description: Use when benchmarking a de novo transcriptome assembly (e.g. Trinity-assembled
  H. subflexa pheromone gland) with BUSCO ortholog counts, especially to verify that
  chimera/redundancy filtering reduces duplicated BUSCO groups.
when_to_use_negative:
- A reference genome is available for the organism, making reference-based quality
  assessment preferable to reference-free BUSCO benchmarking
- The goal is transcript quantification or differential expression rather than assembly
  quality/completeness assessment
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: BUSCO
  role: Benchmark universal single-copy orthologs (Insecta odb9) to score assembly
    completeness and duplication before and after filtering
- name: TransRate
  role: Used alongside BUSCO to establish a ground quality score of the unfiltered
    assembly and to filter chimeric/low-evidence contigs
- name: Bellerophon
  role: Pipeline whose filtering steps (TPM cut-off 1, CD-HIT-EST 95% identity) produce
    the filtered assembly that BUSCO re-assesses
  repo: github.com/JesseKerkvliet/Bellerophon
- name: Trinity
  role: De novo assembler that produced the H. subflexa pheromone gland transcriptome
    assessed by BUSCO
- name: CD-HIT-EST
  role: Removes highly identical redundant contigs at 95% identity prior to BUSCO
    re-assessment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:312c12f0eae90d9cb46322b275dd3fb507e3099136e2c261044ec3c9c98dcbc8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# busco-completeness-assessment

## Summary

Assess de novo transcriptome completeness with BUSCO against the Insecta odb9 ortholog set, tracking present, duplicated, and missing groups before and after assembly filtering (e.g. Bellerophon) to quantify redundancy removal and completeness loss.

## When to use

Use when benchmarking a de novo transcriptome assembly (e.g. Trinity-assembled H. subflexa pheromone gland) with BUSCO ortholog counts, especially to verify that chimera/redundancy filtering reduces duplicated BUSCO groups.

## When NOT to use

- A reference genome is available for the organism, making reference-based quality assessment preferable to reference-free BUSCO benchmarking
- The goal is transcript quantification or differential expression rather than assembly quality/completeness assessment

## Inputs

- De novo assembled transcriptome in FASTA format (e.g. Trinity assembly of H. subflexa pheromone gland RNA-Seq reads)
- Insecta odb9 ortholog set (BUSCO lineage dataset)

## Outputs

- BUSCO completeness report with counts of present, duplicated, and missing ortholog groups (e.g. present 1,420; 85.8%, duplicated 182 after filtering)
- Report file (e.g. bellerophon_report.txt) recording pre- vs post-filtering BUSCO metrics for comparison

## How to apply

Run BUSCO on the unfiltered de novo assembly using the Insecta odb9 ortholog set to establish a ground quality score, as done for the Trinity-assembled H. subflexa pheromone gland transcriptome (BUSCO present 1,511; 91.1%, duplicated 562). After applying the filtering pipeline (TPM filter with default cut-off of 1, CD-HIT-EST at 95% identity, TransRate filtering), re-run BUSCO on the filtered assembly alongside TransRate-Q. Interpret the change in duplicated BUSCO groups as evidence of redundant/chimeric-suspect contig removal: in the reference run duplication dropped from 562 to 182 (22.9% of groups), while present groups decreased from 1,511 (91.1%) to 1,420 (85.8%), so judge success by a large duplication reduction with an acceptable completeness trade-off and reproducibility within expected run-to-run variation.

## Related tools

- **BUSCO** (Benchmark universal single-copy orthologs (Insecta odb9) to score assembly completeness and duplication before and after filtering)
- **TransRate** (Used alongside BUSCO to establish a ground quality score of the unfiltered assembly and to filter chimeric/low-evidence contigs)
- **Bellerophon** (Pipeline whose filtering steps (TPM cut-off 1, CD-HIT-EST 95% identity) produce the filtered assembly that BUSCO re-assesses) — github.com/JesseKerkvliet/Bellerophon
- **Trinity** (De novo assembler that produced the H. subflexa pheromone gland transcriptome assessed by BUSCO)
- **CD-HIT-EST** (Removes highly identical redundant contigs at 95% identity prior to BUSCO re-assessment)

## Examples

```
python Bellerophon.py --assembly Hsubflexa_pheromone_gland.fasta --left SRR8735410_R1.fastq --right SRR8735410_R2.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Duplicated BUSCO groups drop substantially after filtering (reference: 562 to 182, i.e. 22.9% of total groups)
- BUSCO present-group completeness stays within a reasonable range of the unfiltered value (reference: 1,511; 91.1% down to 1,420; 85.8%)
- BUSCO metrics reproduce within expected run-to-run variation when re-run on the same assembly
- The paired TransRate score improves from 0.247 to 0.312 for the same filtered assembly
- The run's report file bellerophon_report.txt exists and records the filtered assembly's BUSCO counts

## Limitations

- Bellerophon may erroneously remove contigs from alternative splicing that are misclassified as chimeras if they are unevenly expressed along their sequence
- BUSCO itself cannot remove redundant or chimeric contigs; it only scores them, so a low duplication count must be achieved by upstream filtering
- When there is no reference genome available, the quality of a de novo assembled transcriptome is difficult to assess beyond ortholog-based metrics
- No tool version numbers are stated for BUSCO, TransRate, CD-HIT-EST, or Trinity in the source, complicating exact reproduction

## Evidence

- [methods] Establish ground quality score of unfiltered assembly with Busco and TransRate: "(a) Busco and Transrate are used to establish a ground quality score of the unfiltered assembly."
- [methods] Assess quality of filtered transcriptome with TransRate-Q and BUSCO: [evidence span withheld — anchor status: not_found]
- [methods] Insecta odb9 ortholog set used for benchmarking: "using the Insecta odb9 ortholog set."
- [discussion] Bellerophon reduces duplicated BUSCO groups from 562 to 182: "the female H. subflexa pheromone gland transcriptome from 0.247 to 0.312"
- [intro] Bellerophon pipeline designed to filter chimeras and reduce false positives: "We developed a pipeline that is specifically aimed to filter out chimeras to reduce false‐positive gene discovery and false‐negative differentially expressed genes."
