---
name: mhc-class-i-allele-typing-with-optitype
description: Use when tumor FASTQ/BAM reads (DNA-seq or RNA-seq) need HLA class I
  genotyping to constrain MHC-I peptide binding predictions, or when custom allele
  files must be merged into typed allele sets.
when_to_use_negative:
- The goal is HLA class II genotyping — OptiType covers class I only; HLA-HD (hlahd.sh
  with freqdata, split, dict reference data) is the class II path in ScanNeo2
- Reads are from normal samples — rows with type 'normal' in samples.tsv are excluded
  from HLA typing
- Already curated allele lists exist per sample — use the custom mode instead of re-running
  read-based yara+OptiType typing
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3337
- http://edamontology.org/topic_0625
tools:
- name: ScanNeo2
  role: Snakemake workflow that dispatches class-conditional HLA typing, merges allele
    sets across modes, and writes mhc-I.tsv
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: yara
  role: Filters DNA-seq and RNA-seq reads down to HLA reads before OptiType alignment
- name: Optitype
  role: Aligns the yara-filtered reads against a panel of HLA class I alleles to call
    class I genotypes
- name: HLA-HD
  role: Complementary class II genotyper invoked via hlahd.sh in the same hlatyping
    block; not part of the class I OptiType path
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:145937352db26d4fc3ee8362fbc5b4e10f274618b0e0f6c0559bafe375428c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - HLA-HD
  - Optitype
  - ScanNeo2
  - yara
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3337
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mhc-class-i-allele-typing-with-optitype

## Summary

Determines patient HLA class I alleles as required downstream by ScanNeo2's neoantigen prediction, by filtering tumor DNA-seq/RNA-seq reads for HLA reads with yara and aligning them against a panel of HLA class I alleles with OptiType, with per-source allele outputs.

## When to use

Use when tumor FASTQ/BAM reads (DNA-seq or RNA-seq) need HLA class I genotyping to constrain MHC-I peptide binding predictions, or when custom allele files must be merged into typed allele sets.

## When NOT to use

- The goal is HLA class II genotyping — OptiType covers class I only; HLA-HD (hlahd.sh with freqdata, split, dict reference data) is the class II path in ScanNeo2
- Reads are from normal samples — rows with type 'normal' in samples.tsv are excluded from HLA typing
- Already curated allele lists exist per sample — use the custom mode instead of re-running read-based yara+OptiType typing

## Inputs

- Tumor FASTQ or BAM sequencing reads (DNA-seq or RNA-seq) located via config/samples.tsv (rows with type 'normal' excluded)
- hlatyping block of config/config.yml specifying class (I, II, or BOTH) and MHC-I_mode as comma-separated DNA, RNA, custom
- User-supplied tab-delimited class I allele file in the custom_hla_I sample-sheet column (custom mode)

## Outputs

- results/<sample>/hla/mhc-I.tsv with per-source allele rows formatted '<source>\t<allele>' (source is DNA, RNA, or custom)
- Merged class I allele set across all listed MHC-I_mode sources, ready as input to MHC-I peptide binding/immunogenicity prediction with IEDB

## How to apply

Parse the hlatyping block of config/config.yml to confirm class I is being typed and to read the MHC-I_mode list (comma-separated DNA, RNA, custom). Read config/samples.tsv to locate the per-sample reads, using only tumor rows since rows with type 'normal' are excluded from HLA typing. For DNA or RNA modes, filter the sequencing reads for HLA reads with yara, then align the filtered reads against a panel of HLA class I alleles with OptiType; for custom mode, load the user-supplied tab-delimited allele file given in the custom_hla_I sample-sheet column. When multiple modes are given (e.g. 'DNA, RNA'), merge the allele sets across sources while keeping read-based and custom sets when combined. Write the result as results/<sample>/hla/mhc-I.tsv with each row formatted '<source>\t<allele>'. Note the article's complementary path: OptiType covers class I only, whereas HLA-HD (hlahd.sh with freqdata, split, and dict reference directories) is used for class II.

## Related tools

- **ScanNeo2** (Snakemake workflow that dispatches class-conditional HLA typing, merges allele sets across modes, and writes mhc-I.tsv) — https://github.com/ylab-hi/ScanNeo2/
- **yara** (Filters DNA-seq and RNA-seq reads down to HLA reads before OptiType alignment)
- **Optitype** (Aligns the yara-filtered reads against a panel of HLA class I alleles to call class I genotypes)
- **HLA-HD** (Complementary class II genotyper invoked via hlahd.sh in the same hlatyping block; not part of the class I OptiType path)

## Examples

```
snakemake --use-conda --configfile config/config.yml results/<sample>/hla/mhc-I.tsv
```

## Evaluation signals

- results/<sample>/hla/mhc-I.tsv exists and every row follows the '<source>\t<allele>' schema with source in {DNA, RNA, custom}
- If MHC-I_mode lists multiple comma-separated sources (e.g. 'DNA, RNA'), the TSV contains one row-set per source, with allele sets merged while keeping read-based and custom sets when combined
- Only tumor sample rows were consumed from config/samples.tsv; no 'normal' rows contributed alleles
- At workflow load, no '[config error]' is emitted for the class I DNA/RNA path (per the task card, the load-time HLA-HD reference/hlahd.sh validation applies to class II, not OptiType class I)
- Custom mode produced no yara/OptiType runs — alleles came verbatim from the custom_hla_I tab-delimited file

## Limitations

- OptiType path handles class I only; typing class II from DNA or RNA requires HLA-HD plus its freqdata, split, and dict reference directories and hlahd.sh on PATH, and ScanNeo2 emits a '[config error]' at load time if these are missing
- If the reads are exhausted for typing, the custom mode is the fallback, so correctness depends entirely on the user-supplied tab-delimited allele files
- No pinned Snakemake version is documented in the article text (README badge indicates snakemake >= 9.0.0)

## Evidence

- [results] yara filters reads, OptiType types class I alleles: "DNA-seq and RNA-seq data are filtered for HLA reads using yara (Siragusa et al. 2013) and subsequently aligned against a panel of HLA class I alleles using Optitype"
- [other] per-source allele rows written to mhc-I.tsv / mhc-II.tsv with '<source>\t<allele>' format: [evidence span withheld — anchor status: not_found]
- [results] yara filter + OptiType class I path within the Snakemake-based ScanNeo2 workflow: "We took advantage of those tools and developed a snakemake-based workflow for the detection of neoantigens from multiple sources, termed ScanNeo2"
- [results] class II path uses HLA-HD, confirming OptiType is class-I-specific: "In a similar manner, HLA-HD (Kawaguchi et al. 2017) is used for HLA class II genotyping"
