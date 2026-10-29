---
name: whole-genome-duplication-warning
description: Use when running XClone's combine module on merged RDR and BAF results
  from scRNA-seq data and you need to check whether allele-specific CNA patterns (e.g.,
  widespread LOH plus copy gains) indicate whole-genome duplication.
when_to_use_negative:
- Input is raw scRNA-seq counts without prior RDR and BAF module outputs — the combine/WGD
  check requires both module results
- Sample is copy-number neutral (e.g., normal reference cells like ref_celltype='N');
  WGD signatures will not be present and the warning is not expected
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3518
tools:
- name: XClone
  role: Provides the combine module (run_combine) that merges RDR and BAF CNA states
    and emits the WGD warning
  repo: github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit generating the AD, DP and total read depth matrices
    consumed by the RDR/BAF modules upstream of the WGD check
  repo: https://github.com/hxj5/xcltk
- name: statsmodels
  role: Negative-binomial GLM fitting inside the RDR module feeding the combined CNA
    states
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:c9108b0cdaabac60e15a0d41b885faa0beced9796b765998a948be3ea3cd1898
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - XClone
  - scanpy
  - statsmodels
  - xcltk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# whole-genome-duplication-warning

## Summary

Detects and flags whole-genome duplication (WGD) signatures when combining XClone's RDR and BAF modules: the combine step emits four-state CNA calls and raises a WGD warning when duplication signatures are present in the per-gene-per-cell CNA states.

## When to use

Use when running XClone's combine module on merged RDR and BAF results from scRNA-seq data and you need to check whether allele-specific CNA patterns (e.g., widespread LOH plus copy gains) indicate whole-genome duplication.

## When NOT to use

- Input is raw scRNA-seq counts without prior RDR and BAF module outputs — the combine/WGD check requires both module results
- Sample is copy-number neutral (e.g., normal reference cells like ref_celltype='N'); WGD signatures will not be present and the warning is not expected

## Inputs

- RDR_Xdata: AnnData from xclone.model.run_RDR with genes-by-cells CNA-state probabilities
- BAF_merge_Xdata: AnnData from xclone.model.run_BAF with phased mega-gene allele states mapped back to gene scale
- XCloneConfig combine-module configuration (merge_loh=True, copygain_correct=False)

## Outputs

- combine_Xdata.h5ad: AnnData with per-gene-per-cell CNA state calls spanning four combined states plus smoothed probability layers
- Whole-genome-duplication warning flag emitted when duplication signatures are present

## How to apply

Run the RDR module (negative-binomial GLM with KNN and HMM smoothing) and the BAF module (three-step haplotyping with EM and dynamic programming, CNV_N_components=5, BAF_denoise=True) first, then call xclone.model.run_combine(RDR_Xdata, BAF_merge_Xdata, config_file=xconfig) with merge_loh=True and copygain_correct=False. The combine step merges copy loss/neutral/gain RDR states with BAF allele states into four combined CNA states per gene and cell. Inspect the combined output for genome-wide duplication signatures; when present, the module emits a whole-genome-duplication warning, enabling identification of corresponding subclones. The result is saved as combine_Xdata.h5ad, which should contain per-gene-per-cell four-state CNA calls plus smoothed probability layers.

## Related tools

- **XClone** (Provides the combine module (run_combine) that merges RDR and BAF CNA states and emits the WGD warning) — github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit generating the AD, DP and total read depth matrices consumed by the RDR/BAF modules upstream of the WGD check) — https://github.com/hxj5/xcltk
- **statsmodels** (Negative-binomial GLM fitting inside the RDR module feeding the combined CNA states)

## Examples

```
xconfig.merge_loh = True; xconfig.copygain_correct = False; combine_Xdata = xclone.model.run_combine(RDR_Xdata, BAF_merge_Xdata, config_file=xconfig)
```

## Evaluation signals

- combine_Xdata.h5ad exists and contains per-gene-per-cell CNA state calls across the four combined states
- Smoothed probability layers are present in the combined AnnData output
- The WGD warning is emitted when genome-wide duplication signatures are present in the CNA calls
- merge_loh=True correctly merges copy-neutral LOH states so BAF-informed loss of heterozygosity is reflected in the final calls
- On the astrocytoma demo (GBM_10XsnRNA), the output suggests whole genome duplication consistent with the published result

## Limitations

- WGD is reported as a warning-level inference; quantitative details supporting WGD inference in specific samples are not fully spelled out in the article discussion
- Detection depends on quality of upstream BAF phasing and RDR smoothing; noisy or sparse scRNA-seq data may mask genome-wide duplication signatures
- XClone is under active development; results can vary between versions, so keep software up to date per the README

## Evidence

- [full_text] WGD warning on combine: [evidence span withheld — anchor status: not_found]
- [abstract] XClone can indicate WGD: "and potentially indicate whole genome duplication, therefore enabling the discovery of corresponding subclones and the dissection of their phenotypic impacts"
- [discussion] astrocytoma WGD result: "from the astrocytoma sample, XClone showed the capability of suggesting whole genome duplication and increased burden of chromosomal instability"
- [readme] README demo repos: "The demo of XClone and results on the all processed cancer datasets are available at `xclone-data <https://github.com/Rongtingting/xclone-data>`_."
