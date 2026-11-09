---
name: single-cell-comparative-regulatory-analysis
description: Use when you have four single-cell datasets (one scRNA-seq and one scATAC-seq
  from each of two conditions/populations) and need to identify subpopulation-specific
  differential regulatory relations between transcription factors and target genes.
when_to_use_negative:
- You only have a single condition/sample — sc_compreg tests differential regulation
  between two linked subpopulations, not regulation within one sample
- More than one donor per condition — the method does not yet accommodate multiple
  donors under each condition (extension noted as pending in the discussion)
- Expression/accessibility matrices are not log2-transformed and have not been preprocessed
  into PeakName_intersect.txt / peak_gene_prior_intersect.bed via sc_compreg_process_data_.sh
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: scCompReg
  role: R package installed from source whose sc_compreg function computes TFRP scores,
    fits the regression model, and computes the likelihood-ratio statistic for differential
    TF-TG regulation
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment for the scCompReg package and the sc_compreg call
- name: Bedtools
  role: Linux dependency used in preprocessing peak data (cnmf_process_data.sh / sc_compreg_process_data_.sh)
- name: Homer
  role: Linux dependency for motif/peak preprocessing in the sc-compReg pipeline
- name: PECA
  role: Related prior software for inferring context-specific gene regulatory networks
    from paired expression and chromatin accessibility data, from the same lab
  repo: SUwonglab/PECA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:30aa1ae4db236297b0609c8ef339a0b33dbd445e76e0285419f62ee506f67d69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - Homer
  - PECA
  - R (>= 3.6.0)
  - sc-compReg
  - scCompReg
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# single-cell-comparative-regulatory-analysis

## Summary

Implements the sc-compReg workflow for comparing gene regulatory networks between two conditions from paired single-cell RNA-seq and ATAC-seq data, constructing a cell-specific transcription factor regulatory potential (TFRP) index and testing differential TF-TG regulation across linked subpopulations.

## When to use

Use when you have four single-cell datasets (one scRNA-seq and one scATAC-seq from each of two conditions/populations) and need to identify subpopulation-specific differential regulatory relations between transcription factors and target genes.

## When NOT to use

- You only have a single condition/sample — sc_compreg tests differential regulation between two linked subpopulations, not regulation within one sample
- More than one donor per condition — the method does not yet accommodate multiple donors under each condition (extension noted as pending in the discussion)
- Expression/accessibility matrices are not log2-transformed and have not been preprocessed into PeakName_intersect.txt / peak_gene_prior_intersect.bed via sc_compreg_process_data_.sh

## Inputs

- Paired single-cell RDS objects (sample1.rds, sample2.rds) containing O1/E1/O1.idx/E1.idx/symbol1/peak.name1 and O2 counterparts; expression and accessibility matrices must be log2-transformed
- PeakName_intersect.txt (path passed as peak.name.intersect.dir)
- peak_gene_prior_intersect.bed (path passed as peak.gene.prior.dir)
- Species motif prior: prior_data/motif_human.rds or prior_data/motif_mouse.rds
- motif_file.rds loaded via mfbs_load(motif.target.dir)
- Consistent cluster assignments in scRNA-seq and scATAC-seq (from cnmf or elsewhere)

## Outputs

- compreg.output$hub.tf[[i]] written to tf_<i>.txt: hub transcription factors per subpopulation
- compreg.output$diff.net[[i]] written to diff_net_<i>.txt: TF-target regulatory edges with differential-regulation scores
- One tf_<i>.txt and one diff_net_<i>.txt file per subpopulation i = 1..n.pops

## How to apply

First identify linked subpopulations via joint clustering (e.g. coupled nonnegative matrix factorization via cnmf) and pass consistent cluster assignments (O1.idx, E1.idx, O2.idx, E2.idx) to sc_compreg along with log2-transformed gene expression and chromatin accessibility matrices. Preprocess peak names with sc_compreg_process_data_.sh to generate PeakName_intersect.txt and peak_gene_prior_intersect.bed, then call sc_compreg with these paths plus the species motif prior (motif_human.rds or motif_mouse.rds) and the motif target file loaded via mfbs_load. Within sc_compreg, TFRP scores per TF, target gene, and subpopulation are computed by combining TF motif activity with the peak-gene coupling matrix and chromatin accessibility evidence, then simple linear regression models the dependency of TFRP on TG and a likelihood-ratio statistic tests whether the regression model P(TFRP|TG) is the same across the two conditions; p-value computation and FDR control are provided for the statistic. Finally loop over compreg.output$n.pops and write compreg.output$hub.tf[[i]] and compreg.output$diff.net[[i]] to tab-delimited files with write.table(row.names=FALSE, quote=FALSE, sep='\t').

## Related tools

- **scCompReg** (R package installed from source whose sc_compreg function computes TFRP scores, fits the regression model, and computes the likelihood-ratio statistic for differential TF-TG regulation) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment for the scCompReg package and the sc_compreg call)
- **Bedtools** (Linux dependency used in preprocessing peak data (cnmf_process_data.sh / sc_compreg_process_data_.sh))
- **Homer** (Linux dependency for motif/peak preprocessing in the sc-compReg pipeline)
- **PECA** (Related prior software for inferring context-specific gene regulatory networks from paired expression and chromatin accessibility data, from the same lab) — SUwonglab/PECA

## Examples

```
compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, sample2$E2, sample2$O2.idx, sample2$E2.idx, sample2$symbol2, sample2$peak.name2, motif$motif.name, motif$motif.weight, motif$match2, motif.file, peak.name.intersect.dir, peak.gene.prior.dir, sep.char=' ')
```

## Evaluation signals

- Exactly one tf_<i>.txt and one diff_net_<i>.txt file exist for every subpopulation i = 1..compreg.output$n.pops
- Each output file is a readable tab-delimited table (write.table with row.names=FALSE, quote=FALSE, sep='\t')
- diff_net tables contain TF-target regulatory edges with their differential-regulation scores
- Input matrices were log2-transformed before calling sc_compreg
- Likelihood-ratio p-values are computed with FDR control across tested regulatory relations

## Limitations

- Differential regulations discovered by comparing one CLL donor to one healthy control will need to be replicated by additional patients and controls
- The method does not yet accommodate multiple donors under each of the two conditions; extending it is described as an important next step
- sc-compReg does not yet support analysis incorporating bulk HiChIP data; that extension is planned
- Requires R >= 3.6.0 and Linux tools Bedtools and Homer; example data must be fetched with git lfs

## Evidence

- [abstract] Here, we propose sc-compReg as a method for the comparative analysis of gene expression regulatory networks between two conditions: "Here, we propose sc-compReg as a method for the comparative analysis of gene expression regulatory networks between two conditions"
- [readme] R version requirement: "* Operating System: Linux or MacOS
* R (>= 3.6.0)
* Bedtools (Linux)
* Homer (Linux)"
- [readme] sc_compreg call with motif inputs: "compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, ..."
- [readme] Output writing loop: "for (i in 1:compreg.output$n.pops) { write.table(compreg.output$hub.tf[[i]], paste(path, 'tf_', i, '.txt', sep=''), row.names = F, quote = F, sep = '\t')"
- [readme] Log2-transformed input requirement: "* **log2-transformed** gene expression matrices of samples 1 and 2
* **log2-transformed** chromatin accessibility matrices of samples 1 and 2"
- [methods] Motif prior loading: "motif = readRDS('prior_data/motif_human.rds')"
- [discussion] Multiple-donor limitation: "an important next step will be to extend the approach to accommodate multiple donors under each of the two conditions to be compared"
