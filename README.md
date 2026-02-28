# breast-cancer-geo-gene-mapper
"An R pipeline for converting raw Affymetrix microarray probes to unique gene symbols without statistical filtering. Designed to preserve full transcriptomic profiles for complete downstream analysis like GSEA."
# Unfiltered Microarray Probe-to-Gene Converter

## 📌 Overview
This repository contains an R pipeline designed to convert raw Affymetrix microarray probe IDs (specifically from the `AFFY_HG_U133_PLUS_2` platform) into unique human gene symbols. 

Crucially, this script performs **no statistical filtering** (no P-value or Log2FC cutoffs). It is specifically engineered to retain the entire transcriptomic profile (both significant and non-significant genes). This is an essential data-preparation step for downstream bioinformatics applications that require full, unfiltered gene sets, such as Gene Set Enrichment Analysis (GSEA) or comprehensive Venn diagram intersections in breast cancer transcriptomics.

## ✨ Key Features
* **Complete Gene Retention:** Converts probes to genes without dropping non-significant results.
* **Smart Deduplication:** When multiple probes map to a single gene, the script retains the single most statistically relevant representative (based on the lowest P-value) while discarding the redundant probes.
* **Automated Batch Processing:** Seamlessly iterates through multiple raw Differential Expression Analysis (DEA) CSV files from GEO datasets.
* **Powered by gprofiler2:** Utilizes the robust `gprofiler2` library for highly accurate `hsapiens` probe mapping.

## 📂 Input & Output
**Inputs:** Raw Step 3 DEA CSV files containing upwards of 54,000+ Affymetrix probes.
*(Default datasets configured in the script include GSE20685, GSE2990, GSE42568, GSE54002, and GSE41998).*

**Outputs:**
For every input file, the script generates a `*_Raw_Venn_Input.csv` file containing cleanly mapped, unique Gene Symbols ready for immediate downstream use.

## 🛠️ Prerequisites
To run this script, you will need R installed along with the following packages:

```R
install.packages("dplyr")
install.packages("gprofiler2")
