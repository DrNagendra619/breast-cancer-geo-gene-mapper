# ==============================================================================
# FINAL SCRIPT: CONVERT RAW FILES -> UNIQUE GENES (NO STATISTICAL FILTERING)
# ==============================================================================

# 1. Load Libraries
library(gprofiler2)
library(dplyr)

# 2. Set Directory
setwd("D:/DOWNLOADS") 

# 3. Define the EXACT Input Files (Step 3 Raw Files)
files_to_process <- c(
  "GSE20685_Step3_DEG_Subtypes.csv",
  "GSE2990_Step3_DEA_Results.csv",
  "GSE42568_Step3_DEA_Results_Fixed.csv",
  "GSE54002_Step3_DEA_Results.csv",
  "GSE41998_Step3_DEA_Results.csv"
)

# 4. Define Platform (Affymetrix U133 Plus 2.0)
platform_id <- "AFFY_HG_U133_PLUS_2" 

print("--- STARTING RAW CONVERSION (NO P-VALUE/LogFC FILTERS) ---")

for (filename in files_to_process) {
  
  if (file.exists(filename)) {
    print(paste("📂 Processing:", filename))
    
    # A. Read the Raw File (54,000+ rows)
    df <- read.csv(filename)
    
    # B. Fix Column Name (The first column is the Probe ID)
    colnames(df)[1] <- "Probe_ID"
    
    # C. Convert ALL Probes to Gene Symbols
    #    (Querying all 54,000 probes - this may take 30-60 seconds per file)
    print("   ... Converting Probes to Genes (Please wait) ...")
    gost_result <- gconvert(query = df$Probe_ID, 
                            organism = "hsapiens", 
                            target = platform_id, 
                            filter_na = TRUE)
    
    # D. Merge Gene Names back
    df_mapped <- df %>%
      inner_join(gost_result[, c("input", "name")], by = c("Probe_ID" = "input")) %>%
      rename(Gene_Symbol = name)
    
    # E. REMOVE DUPLICATES (Strictly reducing duplicates, not filtering significance)
    #    Logic: If we have 3 probes for 'BRCA1', we just keep the one that appears first 
    #    (or the one with the lowest P-value if available, purely to pick the 'best' representative).
    #    We do NOT remove rows because P-value is > 0.05.
    
    if ("P.Value" %in% colnames(df_mapped)) {
      # Sort by P-value so the "duplicate removal" keeps the most relevant probe for that gene
      df_mapped <- df_mapped %>% arrange(P.Value)
    }
    
    # The command that obeys her instruction: "Remove Duplicates"
    df_unique <- df_mapped %>%
      distinct(Gene_Symbol, .keep_all = TRUE)
    
    # F. Save the Final Output
    output_filename <- paste0(gsub(".csv", "", filename), "_Raw_Venn_Input.csv")
    write.csv(df_unique, output_filename, row.names = FALSE)
    
    print(paste("✅ Created:", output_filename))
    print(paste("   Raw Probes (Start):", nrow(df)))
    print(paste("   Unique Genes (Final):", nrow(df_unique)))
    print("------------------------------------------------")
    
  } else {
    print(paste("⚠️ FILE NOT FOUND:", filename))
  }
}

print("🎉 DONE! These files contain ALL genes (Significant + Non-Significant).")