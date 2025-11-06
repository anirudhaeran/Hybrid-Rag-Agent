# Hybrid AI RAG Agent (n8n + Gemini + SQL + LightRAG) 
[**Experience the Demo**](https://phenomenal-ganache-e00ad6.netlify.app/)

This project is a complete, end-to-end intelligent AI agent built to solve a critical business problem: **how to answer questions from *all* of an organization's data, not just its documents.**

The system automatically ingests and understands both **structured data** (from Excel/Google Sheets) and **unstructured data** (from PDFs/text documents) and routes user questions to the correct knowledge base.

You can ask it:
* *"What were the total sales for 'Richard Allnutt' in Q3?"* (It will query the SQL database).
* *"Summarize the key findings from the Q3 2025 market analysis report."* (It will query the GraphRAG server).

---

## 🚀 The Core Problem & Solution

Traditional RAG (Retrieval-Augmented Generation) is excellent for text documents but fails when asked specific, quantitative questions about structured data (like "what is the total of column X").

This project solves that by implementing a **Hybrid RAG** architecture:

* **The Problem:** A user wants one simple chat interface to ask questions about all company data, whether it's in a sales spreadsheet or a 20-page PDF report.
* **The Solution:**
    1.  **Specialized Pipelines:** We treat each data type differently. Excel files are automatically ingested into a **PostgreSQL (SQL)** database, while text documents are ingested into a **LightRAG (Graph)** database.
    2.  **AI Router:** We build a central **AI Agent** (powered by Google Gemini) that acts as a smart "router." It analyzes the user's natural language question and intelligently decides *which* database to query to get the most accurate answer.

## ✨ Key Features

* **🧠 Central AI Agent:** A single "brain" (built with n8n and Gemini) that understands user intent and functions as a smart router, deciding whether to call the SQL or GraphRAG tool.
* **📈 SQL RAG for Structured Data:**
    * Automatically ingests, sanitizes, and maps Excel/Google Sheets files into new PostgreSQL tables.
    * The agent dynamically inspects the database schema (`Get Database Schema`) and then **writes and executes its own SQL queries** to answer quantitative questions.
* **📚 Graph RAG for Unstructured Data:**
    * Automatically ingests text documents (PDFs, .txt) into a self-hosted LightRAG server.
    * The agent queries this knowledge graph to answer qualitative, conceptual, or summary-based questions.
* **⚙️ Fully Automated Data Ingestion:**
    * The entire system is "live." A user simply drops a file (Excel *or* PDF) into a specific Google Drive folder.
    * n8n workflows automatically detect the new file, identify its type, and trigger the correct ingestion pipeline (SQL or GraphRAG).
* **🔄 Automated Data Synchronization:**
    * A scheduled daily workflow acts as a "garbage collector."
    * It compares the files in Google Drive against the tables in PostgreSQL and the documents in LightRAG.
    * Any data corresponding to a deleted file is automatically purged from the databases, preventing data drift and keeping the agent's knowledge current.

## 🛠️ Tech Stack

* **Automation & Agent Orchestration:** [n8n](https://n8n.io/)
* **AI Model (LLM):** [Google Gemini](https://ai.google.dev/) (via API)
* **Data Source:** [Google Drive](https://www.google.com/drive/)
* **Structured Database (SQL RAG):** [PostgreSQL](https://www.postgresql.org/)
* **Unstructured Database (Graph RAG):** [LightRAG](https://github.com/HKUDS/LightRAG)
* **Reranker:** [Cohere](https://cohere.com/) (via API)
* **Hosting:** [Google Cloud (VM)](https://cloud.google.com/) & [DigitalOcean](https://www.digitalocean.com/)
* **Containerization:** [Docker](https://www.docker.com/)

## 📊 High-Level System Flow

Here is the life-cycle of a user's question:

1.  **User Query:** A user sends a `POST` request to a single n8n webhook (e.g., `{"message": "How many units did Bob sell?"}`).
2.  **Agent Analysis:** The n8n AI Agent, powered by a custom Gemini prompt, analyzes the question. It determines this is a quantitative question about a specific person ("Bob") and a metric ("how many units").
3.  **Tool Selection (SQL):** The agent decides to use the SQL toolset.
    * **Step 3a:** It first calls the `Get Database Schema` subworkflow to get a list of all tables and columns (e.g., it finds `employee_sales_data(name, units_sold, date)`).
    * **Step 3b:** It uses this schema to **generate a new SQL query** on the fly (e.g., `SELECT SUM(units_sold) FROM employee_sales_data WHERE name = 'Bob'`).
    * **Step 3c:** It executes the query using the `Execute a SQL query in Postgres` tool.
4.  **Final Response:** The agent receives the result (e.g., `[{"sum": 120}]`), formats it into a natural language answer ("Bob sold 120 units."), and returns it to the user.

*(If the user had asked, "Summarize the sales report," the agent would have skipped to step 3 and routed the query to the `GraphRAG` tool instead.)*

## 💡 What This Project Demonstrates

* **Advanced RAG Architecture:** Implementation of a "Hybrid RAG" system, which is a modern solution for querying both structured and unstructured data.
* **End-to-End Automation:** Building a production-ready, autonomous system, not just a demo. The automated ETL pipelines and data-sync workflows show a deep understanding of data lifecycle management.
* **AI Agent Orchestration:** Skill in using an LLM (Gemini) as a "reasoning engine" to orchestrate complex workflows and make dynamic decisions (i.e., "Agentic" design).
* **DevOps & Self-Hosting:** Proficiency in deploying and managing multiple services (n8n, LightRAG, Postgres) on cloud VMs using Docker.
