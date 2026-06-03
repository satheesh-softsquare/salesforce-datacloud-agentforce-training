# Use Case 1 — NovaMart Retail: Know Your Customer

**Domain:** Retail / E-Commerce
**Company:** NovaMart (fictional D2C retail brand)
**Difficulty:** Beginner → Intermediate → Advanced

## Business Problem

NovaMart has customer data scattered across three systems — Salesforce CRM, a Loyalty Program CSV, and a Purchase History ERP export. The same customer exists as multiple records with no shared unique ID. The goal is to unify all three into a single golden customer profile using Data Cloud, and then act on it.

## What Participants Build

### Level 1 — Beginner: Ingest, Map and Unify
- Connect Salesforce CRM via DSO (Data Source Object)
- Upload Loyalty Members CSV as a DLO (Data Lake Object)
- Upload Purchase History CSV as a DLO with Transformation (formula field: SpendCategory)
- Map all streams to Individual and Sales Order DMOs
- Configure and run Identity Resolution (exact email + fuzzy name + phone match)
- Verify unified customer profiles

### Level 2 — Intermediate: Insights, Segmentation and Activation
- Write a Calculated Insight (RFM scoring — spend, recency, frequency)
- Build a dynamic segment: High-Value Lapsed Customers
- Activate segment to a Salesforce CRM Campaign
- Configure a Data Action to auto-create a CRM Task on segment entry

### Level 3 — Advanced: AI Agent with RAG
- Upload NovaMart policy document (return, warranty, delivery, loyalty)
- Create a Search Index with Hybrid search
- Configure a Retriever in Einstein Studio
- Build a grounded Prompt Template in Prompt Builder
- Deploy an Agentforce Service Agent that answers policy questions

## Documents

| File | Audience | Purpose |
|------|----------|---------|
| UC1_Facilitator_Guide_NovaMart.docx | Trainer only | Full solutions, step-by-step instructions, SQL, common mistakes, discussion questions |
| UC1_Participant_Handout_NovaMart.docx | Participant | Challenge card (pages 1-2) + detailed tasks + sample CSV data + Think About It prompts |

## Sample Data

Three CSV files are embedded in the Participant Handout:
- crm_contacts.csv — 15 CRM Contact records
- loyalty_members.csv — 15 Loyalty Program records (intentional email mismatches)
- purchase_history.csv — 20 transaction records

## Key Learning Points

- Difference between DSO and DLO in real project scenarios
- Why Identity Resolution needs multiple match rules (exact + fuzzy)
- How formula fields work at ingestion time in Data Cloud
- Difference between Activation (batch sync) and Data Action (event-driven)
- How RAG grounding prevents AI hallucination in Agentforce agents
