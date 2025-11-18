## Use Speckit to create constitution file

```
Follow instructions in speckit.constitution.prompt.md.
Task:
Generate a complete GitHub Speckit constitution file for a project named FinFlow  a personal finance automation platform built using n8n, Google Sheets, Google Drive, Gmail, and Looker Studio.

Project Overview
FinFlow is an end-to-end ETL system for personal finance analytics. It automates collection, parsing, normalization, and visualization of bank transaction data from multiple sources.

Core Functions
Import bank statements from Gmail (PDF/Excel attachments)
Import files uploaded to Google Drive
Store raw statements
Parse Excel and PDF into structured normalized transactions
Save cleaned data into Google Sheets
Build Looker Studio dashboards including Sankey money-flow charts

Architecture Description
Use the following diagram to describe the architecture section of the constitution file:
flowchart LR
    A1[Gmail</br>Incoming emails with statements]
    A2[Google Drive Uploads</br>Manual or automatic file drop]
    A1 --> B[n8n Ingestion</br>Detect attachments, filter, save]
    A2 --> B[n8n Ingestion</br>Detect new files, filter, save]
    B --> C[Google Drive Raw</br>Bank statements storage]
    C --> D[n8n Parsing</br>Excel and PDF parsing]
    D --> E[Google Sheets</br>Normalized transactions]
    E --> F[Looker Studio</br>Dashboards and Sankey charts]

2. Repository Structure Definition
Document and generate structure for:

/docs
    /architecture
    /etl
    /setup
/n8n
    /workflows
/scripts
    /parsers
    /utils
/config

3. ETL Pipeline Explanation
Include detailed sections for:

Ingestion layer (Gmail + Drive trigger logic)
Raw storage
Parsing: Excel -> JSON, PDF -> JSON
Normalization rules: categories, merchant cleaning, date formats
Data model for Google Sheets
Data push pipelines
Dashboard layer (Looker Studio setup)

4. n8n Setup Guide
Explain:

Required credentials
Required workflows
Trigger conditions
How to deploy/import workflows
Error handling strategy
Retry logic
Logging and monitoring

5. Configuration Files
Specify any .json, .env, .n8n export formats required.

6. Contribution Guidelines

Naming conventions
Versioning of workflows
How to propose ETL changes
Testing flows
Documentation standards

7. Roadmap Template
Include placeholders for:

Future PDF OCR support
Merchant classification ML model
Automatic category learning
Household budgeting templates
Multi-bank dashboards
```
