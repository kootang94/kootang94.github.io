---
title: "Designing a Whole Exome Sequencing (WES) Pipeline: From Reads to Variants"
date: 2025-05-28
categories: [WES, Pipeline]
tags: [GATK, BWA, CNNScoreVariants, VariantCalling, VEP]
layout: single
author_profile: true
read_time: true
---

## 🧠 Why Design Matters

Designing a WES pipeline is not just about chaining tools together — it’s about ensuring data quality, reproducibility, and downstream interpretability. This post outlines my pipeline design decisions for a single-sample germline variant analysis based on GATK Best Practices and personal refinements.

---

## 🧪 Input: What Are We Starting With?

- Paired-end FASTQ files (Illumina, 150bp reads)
- Reference genome: GRCh38 (with decoy and alt contigs)
- Known variant sets: dbSNP, gnomAD

---

## 🧱 Pipeline Overview

The pipeline has three main stages:

1. **Data Preprocessing**
2. **Variant Discovery**
3. **Callset Refinement**

Each is described below 👇

---

### 🔧 1. Data Preprocessing

- **Quality Control**: FastQC → check per-base quality, GC content, adapter contamination
- **Alignment**: BWA-MEM to GRCh38 → SAM to BAM using `samtools`
- **Post-Alignment QC**:
  - Sort & Mark Duplicates: `GATK MarkDuplicates`
  - Base Quality Score Recalibration (BQSR): using known sites from dbSNP

📦 **Output**: Analysis-ready BAM file

---

### 🔬 2. Variant Discovery

- **HaplotypeCaller**: Generate GVCF or raw VCF
- **SelectVariants**: Split into SNPs and INDELs
- **Filtering**:
  - **CNNScoreVariants** (deep learning–based filtering, great for single-sample data)
  - Optionally compare to hard filtering or VQSR (for cohorts)

📦 **Output**: Filtered VCF (SNPs + INDELs)

---

### 🧬 3. Callset Refinement & Annotation

- **Annotation Tools**:
  - **VEP**: Functional prediction (gene, transcript, protein impact)
  - **REVEL**: Pathogenicity for missense variants
  - **SpliceAI**: Splice site disruption predictor
  - **CADD**: Genome-wide deleteriousness score
  - **ClinVar + G2P**: Clinical significance and gene-disease associations

📦 **Final Output**: Annotated VCF + Summary Stats + Coverage Report

---

## 🗂️ Final Output Files

| File Type       | Description                        |
|------------------|------------------------------------|
| BAM              | Final aligned & recalibrated reads |
| VCF              | Filtered variants (SNPs/INDELs)    |
| Annotated VCF    | With VEP, REVEL, CADD, etc.        |
| Coverage Report  | Exon-level depth for QC            |

---

## 🔍 Next Steps

- Automate this with Snakemake or Nextflow
- Compare annotation strategies (e.g., VEP vs. ANNOVAR vs. Funcotator)
- Benchmark filters using a truth set (GIAB, etc.)

---

*If you're building or revising a WES pipeline, feel free to fork this approach or reach out!*
