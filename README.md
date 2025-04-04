# Entity Resolution and Deduplication Report

## Overview

This project aimed to resolve duplicate company records in a dataset imported from multiple systems. Given slight variations in company names and other attributes, the goal was to identify unique companies by grouping duplicate records. The process involved data preprocessing, candidate generation, similarity scoring, clustering, and the selection of a canonical record for each group. The final step was an exploratory analysis to validate the deduplication results.

## Data Overview

The dataset consists of **33,446 records** with **75 columns** containing details such as:

- **Company Names:** `company_name`, `company_legal_names`, `company_commercial_names`
- **Geographic Information:** `main_country`, `main_region`, `main_city`, etc.
- **Contact Information:** `primary_phone`, `emails`, `website_url`, and social media links.
- **Other Attributes:** Business descriptions, tags, and various classification codes.

Some fields (e.g., `tiktok_url`, `alexa_rank`, `ios_app_url`) have a high percentage of missing values, which informed the decision to focus on more consistently populated fields like company names for deduplication.

## Methodology

### 1. Data Loading and Preprocessing

- **Data Import:**  
  The Parquet file (compressed with Snappy) was loaded using Pandas with the `pyarrow` engine.

- **Data Cleaning:**  
  Key text fields were normalized (e.g., lowercasing, punctuation removal, whitespace trimming) to reduce variability in company names. Clean versions of `company_name`, `company_legal_names`, and `company_commercial_names` were created.

- **Blocking Key:**  
  A new column (`blocking_key`) was generated from the first two characters of the cleaned company name to reduce the comparison space during candidate generation.

### 2. Candidate Generation and Similarity Scoring

- **Candidate Generation:**  
  Using the `recordlinkage` library, candidate pairs were generated by blocking on the `blocking_key`. This significantly reduced the number of comparisons.

- **Similarity Metrics:**  
  The Jaro-Winkler algorithm was applied to compare the cleaned company names and legal names. A threshold was set to determine candidate duplicates, and a composite similarity score was computed for each pair.

### 3. Clustering and Canonical Record Selection

- **Clustering:**  
  Duplicate pairs were grouped using NetworkX by constructing a graph where each node represents a record and each edge indicates a duplicate match. Connected components of the graph represent clusters of duplicate records.

- **Canonical Records:**  
  For each cluster, the canonical record was chosen based on the most frequently occurring (mode) company name. This approach aimed to ensure that the canonical record represents the most common naming convention found within the cluster.

### 4. Exploratory Data Analysis (EDA)

A series of analyses were conducted to understand and validate the deduplication process:

- **Cluster Size Distribution:**  
  - **Summary:**  
    - **Total Clusters:** 3,497  
    - **Mean Cluster Size:** ~9.56 records  
    - **Median Cluster Size:** 5 records  
    - **Unique Clusters (Single Record):** 1,062  
    - **Duplicate Clusters (Multiple Records):** 2,435  
    - **Maximum Cluster Size:** 485 records  
  - **Visualization:**  
    A histogram (with log-scaled frequency) was generated to examine the distribution of cluster sizes, revealing that most clusters are small while a few are exceptionally large.

- **Geographic Distribution:**  
  The top 20 countries, based on the `main_country` attribute in the canonical records, were analyzed to understand the regional concentration of companies.

- **Missing Values Analysis:**  
  The top 10 columns with the most missing values were identified, showing that fields such as `tiktok_url`, `alexa_rank`, `ios_app_url`, and `naics_2022_secondary_codes` are sparsely populated. These columns were deemed less critical for the deduplication process.

## Findings and Observations

- **Cluster Analysis:**  
  The clustering yielded a broad distribution:
  - Many clusters contain only a single record, indicating unique entries.
  - The majority of duplicate clusters contain a small number of records (median = 5).
  - A few clusters have very large sizes (up to 485 records), which may warrant further investigation for potential over-aggregation or false positives.

- **Canonical Record Selection:**  
  Selecting the canonical record based on the most frequently occurring company name provided a representative entry for each group. This approach ensures consistency across duplicate records, although further refinement (e.g., combining multiple fields) could enhance the canonical record quality.

- **Data Quality Insights:**  
  The missing values analysis confirmed that several auxiliary fields (e.g., social media URLs) are not reliable for matching. Focusing on core attributes like company names and legal names proved to be a more robust strategy.

## Conclusion and Next Steps

The entity resolution process successfully identified duplicate groups and generated a deduplicated dataset with canonical records. The analysis shows that while the majority of clusters are well-formed, some clusters—especially those with an unusually high number of records—require further manual review or adjustment of matching criteria.

### Recommendations for Further Improvement:

1. **Threshold Tuning:**  
   Experiment with higher similarity thresholds or additional attributes (such as addresses or website domains) to refine the matching process.

2. **Advanced Canonicalization:**  
   Consider merging information from multiple records to create a more comprehensive canonical record rather than selecting a single record.

3. **Manual Validation:**  
   Conduct a detailed review of large clusters to ensure that false positives are minimized.

4. **Scalability:**  
   Explore distributed processing solutions if scaling to billions of records in a production environment becomes necessary.
