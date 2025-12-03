# pdf_read_uc

# PDF Data Sorting & ML Enablement using Databricks Unity Catalog

## 📌 Project Overview

This repository contains the architecture, design, and implementation approach for **sorting and structuring PDF data in Databricks Unity Catalog** to enable advanced Machine Learning (ML) use cases such as document classification, semantic search, Retrieval Augmented Generation (RAG), and analytics.

The solution is built using an enterprise-grade Azure data platform stack:

* **Databricks with Unity Catalog** – Governance and data processing
* **Azure Data Factory (ADF)** – Orchestration and ingestion
* **Azure DevOps** – CI/CD and release automation
* **ADLS Gen2** – Raw PDF storage

---

## 🎯 Objectives

* Ingest PDF documents into Unity Catalog governed storage
* Extract structured content from PDFs
* Enable multiple sorting strategies for ML pipelines
* Maintain lineage and governance
* Support scalable ML feature engineering

---

## 🏗️ High-Level Architecture

```
PDF Files (ADLS Gen2)
        ↓
Azure Data Factory
        ↓
Unity Catalog Volume (Raw PDFs)
        ↓
Databricks Processing Job
        ↓
Silver Layer - Structured Pages
        ↓
Gold Layer - Features / Embeddings
        ↓
ML Pipeline / Vector Search
```

---

## 📂 Repository Structure

```
.
├── ingestion/
│   ├── adf_pipeline.json
│   └── databricks_job_ingest.py
├── processing/
│   ├── pdf_extraction.py
│   └── pdf_chunking.py
├── unity_catalog/
│   ├── schema.sql
│   └── table_definitions.sql
├── ml/
│   ├── embeddings_pipeline.py
│   └── model_training.py
├── ci_cd/
│   ├── azure-pipelines.yml
│   └── deploy.sh
├── docs/
│   └── architecture.png
└── README.md
```

---

## 🔧 Sorting Strategies Implemented

### 1. Page-Level Sorting

Used for NLP and classification use cases.

```sql
CREATE TABLE uc_catalog.silver.pdf_pages (
  document_id STRING,
  page_number INT,
  content STRING,
  extracted_ts TIMESTAMP
);
```

Sorting:

```sql
ORDER BY document_id, page_number;
```

---

### 2. Semantic Sorting (ML-Optimized)

Used for RAG and AI search systems.

```sql
CREATE TABLE uc_catalog.gold.pdf_semantic_index (
  document_id STRING,
  chunk_id STRING,
  text STRING,
  embedding ARRAY<FLOAT>,
  page_number INT
);
```

Sorting based on:

* Relevance score
* Cosine similarity
* Topic clustering

---

### 3. Metadata-Based Sorting

Used for operational and compliance analytics.

```sql
CREATE TABLE uc_catalog.silver.pdf_metadata (
  document_id STRING,
  document_type STRING,
  issue_date DATE,
  department STRING
);
```

Sorting:

```sql
ORDER BY issue_date DESC;
```

---

## 🧠 ML Use Case Mapping

| Use Case                | Strategy   |
| ----------------------- | ---------- |
| Document Classification | Page-based |
| RAG Chatbot             | Semantic   |
| Compliance Automation   | Metadata   |
| Knowledge Search        | Hybrid     |

---

## 🛠️ Technology Stack

| Layer       | Tool                      |
| ----------- | ------------------------- |
| Ingestion   | Azure Data Factory        |
| Processing  | Databricks (PySpark)      |
| Storage     | ADLS Gen2 + Unity Catalog |
| Governance  | Unity Catalog             |
| CI/CD       | Azure DevOps              |
| ML Tracking | MLflow                    |

---

## 📈 Optimization Practices

* Z-Ordering on frequently queried columns

```sql
OPTIMIZE uc_catalog.silver.pdf_pages
ZORDER BY (document_id, page_number);
```

* Delta Lake versioning
* Unity Catalog lineage tracking
* Immutable document versioning

---

## 🔄 CI/CD Integration

Azure DevOps pipeline includes:

* Linting & validation
* Databricks job deployment
* Unity Catalog schema migration
* Automated testing

Sample pipeline file:

```
azure-pipelines.yml
```

---

## 🚀 How to Deploy

1. Clone the repository
2. Configure ADLS and Unity Catalog
3. Update ADF pipeline parameters
4. Trigger Databricks job
5. Validate tables in Unity Catalog

---

## 🔍 Future Enhancements

* OCR integration for scanned PDFs
* Auto-tagging using NLP models
* Document summarization
* Real-time ingestion support

---

## 👨‍💻 Maintainers

Data Platform Engineering Team

For questions or enhancements, create an issue or submit a pull request.

---

## 📜 License

This project is licensed under the MIT License.




# **PDF Ingestion from ADLS Gen2 to Unity Catalog – Technical Guide**

## **1. Overview**

This document provides a step-by-step guide for copying PDF files from **Azure Data Lake Storage (ADLS Gen2)** into **Databricks Unity Catalog** using a Databricks Notebook. This enables teams to manage, govern, and prepare PDF assets for ML/AI workloads.

---

## **2. Prerequisites**

Before starting, ensure the following:

* Unity Catalog is enabled for your workspace.
* A Unity Catalog volume is created for storing PDFs.
* ADLS Gen2 is mounted or accessible via `abfss://` path.

### **Create Unity Catalog Volume**

```sql
CREATE VOLUME IF NOT EXISTS main.ml_schema.pdf_volume;
```

This creates a managed location:

```
/Volumes/main/ml_schema/pdf_volume/
```

---

## **3. Architecture Overview**

```
ADLS Gen2 (PDF Files)
        ↓
Databricks Notebook (Copy Job)
        ↓
Unity Catalog Volume
        ↓
Optional: Delta Metadata Table for PDF Governance & ML
```

---

## **4. Method 1 – BinaryFile Copy (Recommended for ML Use Cases)**

This method reads PDFs as binary content and writes them into Unity Catalog.

### **4.1 Read PDF Files from ADLS**

```python
adls_path = "abfss://container@storageaccount.dfs.core.windows.net/source-pdfs/"
uc_volume_path = "/Volumes/main/ml_schema/pdf_volume/"

df = spark.read.format("binaryFile") \
    .option("pathGlobFilter", "*.pdf") \
    .load(adls_path)
```

### **4.2 Write PDFs into Unity Catalog Volume**

```python
df.write \
  .mode("overwrite") \
  .format("binaryFile") \
  .save(uc_volume_path)
```

### **4.3 Verify Files**

```python
display(dbutils.fs.ls("/Volumes/main/ml_schema/pdf_volume"))
```

---

## **5. Method 2 – Direct File Copy (Simple Migration)**

```python
source_path = "abfss://container@storageaccount.dfs.core.windows.net/source-pdfs/"
target_path = "/Volumes/main/ml_schema/pdf_volume/"

files = dbutils.fs.ls(source_path)

for file in files:
    if file.name.endswith(".pdf"):
        dbutils.fs.cp(file.path, target_path + file.name)
```

---

## **6. Creating a Delta Metadata Table (Optional, ML-Ready)**

This table stores metadata + binary PDF content for analytics and ML pipelines.

```python
spark.read.format("binaryFile") \
     .load("/Volumes/main/ml_schema/pdf_volume") \
     .write \
     .mode("overwrite") \
     .saveAsTable("main.ml_schema.pdf_documents")
```

### **6.1 Metadata Stored**

| Column           | Description                  |
| ---------------- | ---------------------------- |
| path             | Path of the PDF file         |
| modificationTime | Last updated time            |
| length           | File size in bytes           |
| content          | PDF content in binary format |

---

## **7. Benefits of Storing PDFs in Unity Catalog**

* Centralized governance and access controls
* ML-ready storage format for document intelligence
* Versioning and lineage with Delta Lake
* Easy integration with Databricks Model Serving and AI workflows

---

## **8. Optional Extensions**

You can extend this workflow with:

* Text extraction (OCR / PyPDF2 / Azure Document Intelligence)
* Vector embeddings for search and RAG
* Automated ingestion using ADF or Databricks Jobs

---

## **9. Summary**

This solution provides an enterprise-grade approach to ingesting, governing, and preparing PDF data for ML and analytics within Databricks Unity Catalog.

Use the provided notebook snippets to automate PDF ingestion and optionally build a metadata layer for advanced AI/ML use cases.





A few clarifications to better tailor the approach:

What type of PDFs are you working with?

PDFs that already contain tables (like reports, invoices, financial statements)?
Text-heavy PDFs that you want to structure into tabular format?
Scanned/image-based PDFs that need OCR?


What's the desired output structure?

One row per page with extracted text?
Actual table extraction (if PDFs contain tables)?
Custom schema based on document type?


Scale and automation?

Batch processing of multiple PDFs?
Scheduled pipeline or one-time extraction?


Tools preference?

Open source libraries (PyPDF2, pdfplumber, tabula-py)?
Azure Document Intelligence (formerly Form Recognizer)?
Other commercial OCR services?

