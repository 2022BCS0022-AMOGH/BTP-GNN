### Unified Code Quality Dataset

This repository contains the implementation for building a **unified code quality dataset** by combining three distinct datasets—**Maintainability, Vulnerability, and Defectiveness**—at **method-level granularity**. Each method is enriched with a **readability score** computed using the Buse–Weimer tool (`rsm.jar`). The pipeline is modular, reproducible, and designed for transparency in data processing.

***

### Project Structure

- `audit_datasets.py`  
  - Performs a comprehensive audit of all input datasets.
  - Reports counts, file sizes, encodings, sample snippets, schema checks, nulls, duplicates, Java parsing health, and tool version checks.
  - Outputs: `audit_report.json`.

- `method_extraction/`  
  - **Core:** Java-based method extraction using `JavaParser`.
    - `java/MethodExtractorCLI.java`: Extracts method records as JSONL.
    - `javaparser_bridge.py`: Compiles and runs the Java extractor, handles streaming and subprocess management.
    - `output_utils.py`: Writes output as stream pickles and logs progress.
  - **Dataset-specific scripts:**
    - `extract_methods_from_maintainability.py`
    - `extract_methods_from_defectiveness.py`
    - `extract_methods_from_vulnerability.py`
    - Capped versions (e.g., `_5k`) for rapid prototyping.

- `score_methods_readability.py`  
  - Scores method readability using `rsm.jar`.
  - Handles batch processing, error fallback, and stratified sampling.
  - Filters non-finite scores (NaN/inf).
  - Outputs: scored pickles (e.g., `maintainability_methods_scored_balanced_5k.pkl`).

- `unify_datasets.py`  
  - Merges three scored datasets into a single master dataset.
  - Preserves dataset labels, source tracking, and method metadata.
  - Outputs:
    - `master_dataset.csv`
    - `unification_report.json`
    - `filtered_out_methods.json`

***

### Key Features

- **Method-level granularity:** All methods are extracted and labeled at the method level.
- **Label preservation:** Each method retains its original dataset label (maintainability, vulnerability, defectiveness).
- **Readability scoring:** Methods are scored using the Buse–Weimer tool, with controls for batch size and stratification.
- **Robust extraction:** Handles edge cases like method-only snippets and streaming deadlocks.
- **Reproducibility:** All scripts are versioned, and outputs are logged for auditability.

***

### Usage

1. **Audit datasets:**  
   ```bash
   python audit_datasets.py --data-dir /path/to/datasets
   ```

2. **Extract methods:**  
   - Run the appropriate extraction script for each dataset.
   - Example:  
     ```bash
     python extract_methods_from_maintainability.py --input-dir /path/to/maintainability --output-file maintainability_methods.pkl
     ```

3. **Score readability:**  
   ```bash
   python score_methods_readability.py --input-file maintainability_methods.pkl --output-file maintainability_methods_scored.pkl --max-methods 5000
   ```

4. **Unify datasets:**  
   ```bash
   python unify_datasets.py --maintainability maintainability_methods_scored_5k.pkl --vulnerability vulnerability_methods_scored_5k_final.pkl --defectiveness defectiveness_methods_scored_balanced_5k.pkl --output master_dataset.csv
   ```

***

### Output Files

- `audit_report.json`: Audit results for all datasets.
- `*.pkl`: Extracted and scored method records.
- `master_dataset.csv`: Unified dataset with 14 columns and 15,000 rows.
- `unification_report.json`: Summary of the unification process.
- `filtered_out_methods.json`: Methods filtered out during unification (empty in this release).

***

### Schema

| Column Name               | Description                                                                 |
|---------------------------|-----------------------------------------------------------------------------|
| `sample_id`               | Sequential sample ID (e.g., `sample_000001`).                               |
| `method_code`             | Source code of the method.                                                  |
| `vulnerability_label`     | Vulnerability label (0/1 or NA).                                            |
| `maintainability_label`   | Maintainability label (0/1 or NA).                                          |
| `defect_label`            | Defectiveness label (0/1 or NA).                                            |
| `readability_score`       | Readability score (float or NA).                                            |
| `has_defect_label`        | Flag indicating if defect label is present.                                 |
| `has_vulnerability_label` | Flag indicating if vulnerability label is present.                            |
| `has_maintainability_label`| Flag indicating if maintainability label is present.                         |
| `has_readability_label`   | Flag indicating if readability label is present.                            |
| `source_dataset`          | Source dataset (`maintainability`, `vulnerability`, `defectiveness`).        |
| `method_name`             | Name of the method.                                                         |
| `class_name`              | Name of the class.                                                          |
| `source_folder`           | Original source folder (e.g., `h_javas/l_javas`, `vulnerable_code/fixed_code`). |

***
