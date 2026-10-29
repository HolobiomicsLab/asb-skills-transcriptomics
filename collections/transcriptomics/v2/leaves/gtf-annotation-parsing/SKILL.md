---
name: gtf-annotation-parsing
description: Use when a set of lead sQTL variants must be mapped to annotated intron/gene
  coordinates to test whether sQTLs are enriched in post-transcriptionally spliced
  introns.
when_to_use_negative:
- When the goal is sQTL association mapping itself rather than annotation-based enrichment
  analysis — run the sqtlseeker2-nf pipeline instead
- When no intron-level annotation or splicing kinetics data are available, since co-
  vs post-transcriptional classification cannot be built
- For gene-expression (eQTL) enrichment, which would require gene expression QTL regions
  rather than intron classes
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
tools:
- name: sQTLseekeR2
  role: Consumes the annotation-derived gene location file and reports lead sQTL variants
    that are intersected with the ps/cs intron sets
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow/Docker pipeline that wraps sQTLseekeR2 and produces the sQTL results
    intersected against parsed annotation regions
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine used to launch sqtlseeker2-nf with Docker after annotation-derived
    inputs are prepared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:bd1e967f268c1f281a63495e8bded4736e28299776683912a8a13798784a3597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gtf-annotation-parsing

## Summary

Parse genomic annotation (GTF-style gene/intron coordinates) into the gene location file required by the sqtlseeker2-nf pipeline and into co- vs post-transcriptionally spliced intron sets, so lead sQTL variants can be intersected and tested for enrichment by Fisher's exact test.

## When to use

Use when a set of lead sQTL variants must be mapped to annotated intron/gene coordinates to test whether sQTLs are enriched in post-transcriptionally spliced introns.

## When NOT to use

- When the goal is sQTL association mapping itself rather than annotation-based enrichment analysis — run the sqtlseeker2-nf pipeline instead
- When no intron-level annotation or splicing kinetics data are available, since co- vs post-transcriptional classification cannot be built
- For gene-expression (eQTL) enrichment, which would require gene expression QTL regions rather than intron classes

## Inputs

- Gene annotation (GTF/GFF-style) with gene and intron coordinates
- Gene location file for the pipeline: chr/start/end/geneId columns
- Lead sQTL variants from sQTLseekeR2 nominal/permuted results
- ENCODE nuclear/cytosolic RNA-seq splicing kinetics across 13 cell lines (for co- vs post-transcriptional intron classification)
- eQTL catalog to exclude sQTLs that are also eQTLs

## Outputs

- Gene location file (chr/start/end/geneId) ready for sqtlseeker2-nf
- Two classified intron sets: post-transcriptionally spliced (ps) and co-transcriptionally spliced (cs) introns
- Per-intron-set variant counts and Fisher's exact test odds ratios and p-values (all sQTLs; sQTLs that are not eQTLs)

## How to apply

Parse the gene annotation to produce one of the four sqtlseeker2-nf inputs: a gene location file with chr/start/end/geneId columns. Separately, classify each tested intron from the annotation as co-transcriptionally (cs) or post-transcriptionally (ps) spliced using ENCODE nuclear/cytosolic RNA-seq splicing kinetics across the 13 cell lines. After running the sQTLseekeR2 pipeline in nominal mode (win=5000 bp cis window, fdr=0.05, min_md=0.05) and permuted mode (max_perm=1000), intersect the lead sQTL variants with the ps and cs intron sets. Compute Fisher's exact test of sQTL counts in ps vs cs introns, separately for all sQTLs and for the subset of sQTLs that are not eQTLs. The rationale is that annotation-derived intron categories define the region classes in which regulatory hotspots for splicing are sought.

## Related tools

- **sQTLseekeR2** (Consumes the annotation-derived gene location file and reports lead sQTL variants that are intersected with the ps/cs intron sets) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow/Docker pipeline that wraps sQTLseekeR2 and produces the sQTL results intersected against parsed annotation regions) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine used to launch sqtlseeker2-nf with Docker after annotation-derived inputs are prepared)

## Examples

```
NXF_VER=22.04.0 ./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Gene location file matches the required schema exactly: chr/start/end/geneId columns
- Enrichment odds ratio for sQTLs in ps introns exceeds 1 (expected OR = 1.47 for all sQTLs; OR = 1.67 for sQTLs that are not eQTLs)
- Reproduce reported variant fractions: 9.2% of variants in ps introns are sQTLs versus 6.6% in cs introns
- Fisher's exact test significance reaches p < 10-16 for ps-intron enrichment
- Intron classification covers every tested intron from the annotation, each assigned to exactly one of ps or cs

## Limitations

- Co- vs post-transcriptional splicing classification depends on ENCODE nuclear/cytosolic data from 13 cell lines, so coverage is limited to introns observable in those data
- Whether paired sQTL/eQTL effects arise from the same variant or two variants in LD cannot be resolved, complicating the non-eQTL sQTL subset
- Results can be confounded by different preprocessing and statistical methodologies when comparing sGene and eGene counts

## Evidence

- [abstract] sQTLs tend to be located in post-transcriptionally spliced introns, which would function as hotspots for splicing regulation.: "sQTLs tend to be located in post-transcriptionally spliced introns, which would function as hotspots for splicing regulation."
- [methods] sqtlseeker2-nf ~ A pipeline for splicing QTL mapping: "sqtlseeker2-nf ~ A pipeline for splicing QTL mapping"
- [methods] ./nextflow run guigolab/sqtlseeker2-nf -with-docker: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
- [readme] Test for association between splicing ratios and genetic variants in *cis* (nominal pass): "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
