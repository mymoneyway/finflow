# FinFlow

FinFlow is a personal finance automation system using n8n, Google Sheets, and Looker Studio.  
It imports bank statements (Excel and later PDF), normalizes transactions, and builds visual dashboards including Sankey money flow charts.

## Architecture

```mermaid

flowchart LR

    A1[Gmail</br>Incoming emails with statements] 
    A2[Google Drive Uploads</br>Manual or automatic file drop]

    A1 --> B[n8n Ingestion</br>Detect attachments, filter, save]
    A2 --> B[n8n Ingestion</br>Detect new files, filter, save]

    B --> C[Google Drive Raw</br>Bank statements storage]

    C --> D[n8n Parsing</br>Excel and PDF parsing]

    D --> E[Google Sheets</br>Normalized transactions]

    E --> F[Looker Studio</br>Dashboards and Sankey charts]


```
