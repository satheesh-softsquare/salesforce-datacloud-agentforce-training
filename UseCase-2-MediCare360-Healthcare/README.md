# Use Case 2 — MediCare360 Healthcare: Close the Care Gap

**Domain:** Healthcare / Insurance
**Company:** MediCare360 (fictional health insurance provider)
**Difficulty:** Beginner → Intermediate → Advanced

## Business Problem

MediCare360 has member data in three separate systems — Salesforce CRM, a Claims database CSV, and a Wellness Engagement CSV. The care management team cannot identify members who are both high-claim-risk and disengaged from wellness — the exact members most likely to have a major health event. MediCare360 wants a unified Member 360 view, proactive care gap identification, and an AI agent for member support.

## What Participants Build

### Level 1 — Beginner: Ingest, Map and Unify
- Connect Salesforce CRM via DSO (Member contacts)
- Upload Claims Records CSV as a DLO (fix date and amount field types)
- Upload Wellness Engagement CSV as DLO with Transformation (formula field: EngagementTier)
- Map all streams to Individual and Sales Order DMOs
- Configure Identity Resolution (phone as primary match — emails have mismatches)
- Verify unified Member 360 profiles

### Level 2 — Intermediate: Care Gap Insights and Activation
- Write a Calculated Insight (Member_Care_Gap_Score — total claims, risk tier)
- Build Care Gap Segment: High claim risk AND Disengaged from wellness
- Activate to Salesforce CRM Campaign for care coordinator outreach
- Configure Data Action to auto-create a high-priority CRM Case on segment entry

### Level 3 — Advanced: AI Member Support Agent
- Upload MediCare360 plan benefits document
- Create Search Index (Hybrid) and Retriever
- Build an empathetic Prompt Template in Prompt Builder
- Deploy Agentforce Service Agent for member benefits and claims Q&A

## Documents

| File | Audience | Purpose |
|------|----------|---------|
| UC2_Facilitator_Guide.md | Trainer only | Full solutions, SQL, common mistakes, discussion questions |
| UC2_Participant_Guide.md | Participant | Challenge card + detailed tasks + CSV data + Think About It prompts |

## Sample Data

- crm_members.csv — 15 member Contact records
- claims_records.csv — 20 claim records (intentional email mismatches for IR learning)
- wellness_engagement.csv — 15 wellness records with WellnessScore for formula transformation

## Key Learning Points vs UC1

- Phone as primary IR match rule instead of email (teaches match rule selection)
- Two-condition AND segment (claim risk + wellness tier) vs single-condition in UC1
- Data Action creates a Case instead of a Task (healthcare workflow context)
- Formula field created at ingestion (EngagementTier) used directly in segment without CI
- Empathetic tone instruction in Prompt Template for sensitive healthcare context
