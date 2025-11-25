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
