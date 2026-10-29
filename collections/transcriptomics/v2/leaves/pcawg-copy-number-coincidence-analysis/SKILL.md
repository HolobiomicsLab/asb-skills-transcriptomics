---
name: pcawg-copy-number-coincidence-analysis
description: Use when validating ecDNA/amplification calls predicted from ATAC-Seq
  against an independent whole-genome sequencing cohort (PCAWG) by testing whether
  the same genes show copy-number gains.
when_to_use_negative:
- When no independent whole-genome copy-number cohort (e.g., PCAWG) is available for
  the tumor types of interest, so coincidence cannot be tested
- When ecDNA calls have not yet been generated and gene-overlap annotation has not
  been performed upstream
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: circlehunter
  role: Produces the ecDNA calls from ATAC-Seq data whose amplified gene set is intersected
    with PCAWG copy-number gains
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Runs the circlehunter pipeline that generates the upstream ecDNA calls used
    in this analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:4e5d4abb1a8945ec1b82cd9a5454d47fe9d548ca3b3504163f23d975390fae68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# PCAWG copy-number coincidence analysis

## Summary

Intersect ecDNA-amplified genes with PCAWG copy-number-gain calls and statistically test their coincidence, providing orthogonal whole-genome validation that ATAC-Seq-derived ecDNA calls carry amplified oncogenes.

## When to use

Use when validating ecDNA/amplification calls predicted from ATAC-Seq against an independent whole-genome sequencing cohort (PCAWG) by testing whether the same genes show copy-number gains.

## When NOT to use

- When no independent whole-genome copy-number cohort (e.g., PCAWG) is available for the tumor types of interest, so coincidence cannot be tested
- When ecDNA calls have not yet been generated and gene-overlap annotation has not been performed upstream

## Inputs

- ecDNA calls from the circlehunter Snakemake pipeline (task_005 output) for all analyzed samples
- Gene annotation of regions overlapped by ecDNA calls (251 amplified genes; 37 known oncogenes)
- PCAWG copy-number-gain calls (per-gene gain status)

## Outputs

- Coincidence significance between ecDNA-amplified genes and PCAWG copy-number gains (P = 1.619e-15)
- Gene-level table figure4_gene_analysis.csv with per-gene ecDNA amplification status, oncogene annotation, and all five P values including the PCAWG coincidence P value

## How to apply

After circlehunter ecDNA calls are produced by the Snakemake pipeline, annotate the genes overlapped by each ecDNA region to obtain the amplified gene set (251 genes across the 94 public ATAC-Seq datasets, averaging 10 genes per ecDNA). Obtain PCAWG copy-number-gain calls and intersect them with the ecDNA-amplified gene set. Compute the statistical coincidence between genes gained in PCAWG samples and genes amplified on predicted ecDNAs, yielding a significance value (P = 1.619e-15 in this study). Record the coincidence P value alongside other gene-level statistics (oncogene copy-number, expression, gene-effect, and mutation-enrichment P values) in the per-gene output table, figure4_gene_analysis.csv.

## Related tools

- **circlehunter** (Produces the ecDNA calls from ATAC-Seq data whose amplified gene set is intersected with PCAWG copy-number gains) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Runs the circlehunter pipeline that generates the upstream ecDNA calls used in this analysis)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- The amplified-gene set derived from ecDNA overlaps contains 251 genes, of which 37 are known oncogenes
- The coincidence test between ecDNA-amplified genes and PCAWG copy-number gains yields P = 1.619e-15
- figure4_gene_analysis.csv contains per-gene amplification status, oncogene annotation, and all five P values (copy number P = 8.754e-16, expression P = 2.047e-8, gene effect P = 8.408e-5, PCAWG coincidence P = 1.619e-15, mutation enrichment P = 4.986e-2)

## Limitations

- Current benchmarking of ecDNA calls has limitations and accuracy for all types of focal amplifications cannot be guaranteed, which propagates into the gene set tested against PCAWG
- Most analyzed samples were cell lines and cancer-type proportions deviated significantly, so PCAWG coincidence may not generalize across tumor types
- Other focal amplification classes with misregulated chromatin may be detected as false-positive ecDNA, contaminating the amplified-gene set

## Evidence

- [results] PCAWG coincidence significance: [evidence span withheld — anchor status: not_found]
- [abstract] ecDNA gene content: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [methods] pipeline execution: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] benchmarking limitation: "it is important to note that the current benchmarking strategy has limitations, and we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications"
