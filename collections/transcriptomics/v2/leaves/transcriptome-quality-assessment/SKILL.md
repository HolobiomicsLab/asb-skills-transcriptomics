---
name: transcriptome-quality-assessment
description: Use when you have only sequencing reads and a de novo transcriptome assembly
  (no reference genome) and need a baseline assembly-quality score before filtering
  chimeric or low-evidence contigs.
when_to_use_negative:
- A reference genome is available for the organism — reference-based assessment is
  then more appropriate and the reference-free TransRate baseline adds less value
- You only need chimera detection without re-assessing assembly quality at each filtering
  step
- The input is raw reads without an existing de novo assembly to score
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TransRate
  role: Reference-free assembly quality assessment tool; also supplies TransRate-Q
    (re-assessment of filtered assemblies) and TransRate-C (final filtering of chimeric/low-evidence
    contigs)
- name: BUSCO
  role: Benchmarks assembly completeness using the Insecta odb9 ortholog set to establish
    a ground quality score and to quantify duplicated (chimeric-suspect) BUSCO groups
- name: Bellerophon
  role: Pipeline that orchestrates TransRate/BUSCO assessment before and after filtering
    and generates the results report
  repo: github.com/JesseKerkvliet/Bellerophon
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:cfb923dc48565d442d1a9acb26a2e74d314cd2555c640f25a7026291e3473e1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transcriptome-quality-assessment

## Summary

Reference-free quality assessment of a de novo transcriptome assembly using TransRate (with BUSCO as a ground score) prior to Bellerophon filtering, providing the baseline score against which all filtered assembly versions are judged.

## When to use

Use when you have only sequencing reads and a de novo transcriptome assembly (no reference genome) and need a baseline assembly-quality score before filtering chimeric or low-evidence contigs.

## When NOT to use

- A reference genome is available for the organism — reference-based assessment is then more appropriate and the reference-free TransRate baseline adds less value
- You only need chimera detection without re-assessing assembly quality at each filtering step
- The input is raw reads without an existing de novo assembly to score

## Inputs

- RNA-Seq sequencing reads (paired FASTQ, e.g. Illumina H. subflexa pheromone gland reads SRR8735410, SRR8735411, SRR8735412)
- De novo assembled transcriptome (Trinity-based FASTA, e.g. H. subflexa pheromone gland assembly)
- BUSCO ortholog set for benchmarking (Insecta odb9)

## Outputs

- TransRate quality score of the unfiltered assembly (ground score, e.g. 0.247)
- BUSCO completeness metrics (present, duplicated, e.g. 1511 (91.1%) present / 562 duplicated)
- TransRate-Q assessment of filtered assemblies for score comparison
- Bellerophon-generated results report (bellerophon_report.txt)

## How to apply

Run TransRate on the unfiltered assembly, providing only the RNA-Seq sequencing reads (e.g. Illumina FASTQ pairs such as SRR8735410-12 from H. subflexa pheromone glands) and the Trinity-built FASTA assembly, since TransRate is fully reference-free. In parallel, run BUSCO with the Insecta odb9 ortholog set to establish a ground quality score of the unfiltered assembly. Record the initial TransRate score (e.g. 0.247 for the H. subflexa pheromone gland assembly) and BUSCO counts (e.g. 1511 present, 562 duplicated) as the baseline. After each Bellerophon filtering step (TPM cut-off 1, CD-HIT-EST 95% identity, TransRate-C), re-run TransRate-Q and BUSCO on the filtered assembly and compare against this baseline — a successful assessment shows the score rising (e.g. 0.247 to 0.312) while duplicated BUSCO groups drop (562 to 182).

## Related tools

- **TransRate** (Reference-free assembly quality assessment tool; also supplies TransRate-Q (re-assessment of filtered assemblies) and TransRate-C (final filtering of chimeric/low-evidence contigs))
- **BUSCO** (Benchmarks assembly completeness using the Insecta odb9 ortholog set to establish a ground quality score and to quantify duplicated (chimeric-suspect) BUSCO groups)
- **Bellerophon** (Pipeline that orchestrates TransRate/BUSCO assessment before and after filtering and generates the results report) — github.com/JesseKerkvliet/Bellerophon

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Unfiltered H. subflexa baseline TransRate score ≈ 0.247; after full Bellerophon filtering the re-assessed score rises to ≈ 0.312
- BUSCO duplicated groups drop from 562 to 182 (22.9% of total groups) with Insecta odb9 after filtering
- Report file bellerophon_report.txt is generated automatically and confirms all steps completed in the default order (TPM then CD-HIT-EST, TransRate-C last)
- BUSCO present counts shift from 1511 (91.1%) to 1420 (85.8%), consistent with removal of 6,838 low-evidence and 7,240 chimeric-suspect contigs
- TransRate accepts only reads + assembly as inputs — verify no reference genome was used

## Limitations

- Bellerophon may erroneously remove contigs representing alternative splicing that TransRate misclassifies as chimeric when expression is uneven along the sequence
- Tool version numbers for BUSCO, TransRate, CD-HIT-EST, and Trinity are not stated; backwards compatibility problems with TransRate are a known concern (hence the Vagrant VM option)
- When no reference genome is available, de novo assembly quality is intrinsically difficult to assess
- Custom ordering of filtering steps is disabled in the current pipeline version

## Evidence

- [intro] TransRate is truly reference-free and only requires the sequencing reads and the assembled transcriptome.: [evidence span withheld — anchor status: not_found]
- [methods] Busco and Transrate are used to establish a ground quality score of the unfiltered assembly.: "(a) Busco and Transrate are used to establish a ground quality score of the unfiltered assembly."
- [methods] To assess the quality of the fully filtered transcriptome, TransRate-Q and BUSCO are executed using this filtered assembly.: [evidence span withheld — anchor status: not_found]
- [discussion] the female H. subflexa pheromone gland transcriptome from 0.247 to 0.312: "the female H. subflexa pheromone gland transcriptome from 0.247 to 0.312"
- [methods] Applying Bellerophon reduces duplicated BUSCO groups from 562 to 182 (per source card task_002 finding; supported by Insecta odb9 usage in methods): "using the Insecta odb9 ortholog set."
- [readme] python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun: "python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun"
