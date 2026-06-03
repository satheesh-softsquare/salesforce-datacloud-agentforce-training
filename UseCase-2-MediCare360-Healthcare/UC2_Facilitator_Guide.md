# UC2 — Facilitator Guide | MediCare360 Healthcare
## Data Cloud & Agentforce Training

> ⚠️ **THIS DOCUMENT IS FOR FACILITATOR USE ONLY. DO NOT SHARE WITH PARTICIPANTS.**

---

| Field | Details |
|---|---|
| **Domain** | Healthcare / Insurance |
| **Company** | MediCare360 (fictional health insurance provider) |
| **Difficulty** | Beginner → Intermediate → Advanced |
| **Version** | v1.0 — June 2025 |

---

## Table of Contents

1. [Use Case Overview](#1-use-case-overview)
2. [Sample Data](#2-sample-data)
3. [Level 1 — Beginner](#3-level-1--beginner)
4. [Level 2 — Intermediate](#4-level-2--intermediate)
5. [Level 3 — Advanced](#5-level-3--advanced)
6. [Common Mistakes & How to Help](#6-common-mistakes--how-to-help)
7. [Discussion Questions](#7-discussion-questions-for-the-team-session)
8. [Architecture Summary](#8-architecture-summary)

---

## 1. Use Case Overview

### 1.1 Business Story

MediCare360 is a mid-sized health insurance provider covering 2 lakh members across Tamil Nadu and Karnataka. They offer individual health plans, family floater plans, and corporate group health policies.

Their member data lives in three separate systems:

- **Salesforce CRM (Service Cloud)** — member profiles, contact details, and support case history managed by the member services team.
- **Claims Database (CSV export)** — all insurance claims filed by members, including claim amount, status, and diagnosis category. Exported monthly by the finance team.
- **Wellness Engagement Records (CSV export)** — tracks whether a member completed their annual health checkup, wellness app logins, and preventive care participation. Managed by the wellness team.

The core problem: A member named Kavitha Rajan calls the support center asking about her claim status. The agent has to open three different screens across two systems to piece together her profile. Meanwhile, the care management team has no way to identify members who are both high-risk (frequent claims) and disengaged from wellness programs — the exact members most likely to have a major health event and cost the company the most.

MediCare360 wants to use Salesforce Data Cloud to build a unified Member 360 view — and then use that unified view to proactively identify at-risk members, alert care coordinators, and deploy an AI agent that can answer member queries about their benefits and coverage.

---

### 1.2 What the Participant Will Build

| Level | What They Build | Data Cloud Features | Outcome |
|---|---|---|---|
| 🟢 **Beginner** | Ingest 3 data sources, map to DMOs, resolve member identity across systems | DSO, DLO, DLO+Transform, DMO, Identity Resolution | One unified Member 360 profile per member |
| 🟡 **Intermediate** | Build a Care Gap Calculated Insight to identify high-risk disengaged members, create a segment, activate to CRM for care coordinator outreach, trigger Data Action | Calculated Insights, Segments, Activation, Data Actions | Care coordinator automatically assigned to high-risk members. CRM task created for outreach. |
| 🔴 **Advanced** | Upload plan benefit documents, build Search Index and Retriever, configure Prompt Builder, deploy Agentforce Service Agent for member support | Search Index, Retriever, Prompt Builder, Agentforce Agent | AI agent answers member questions about coverage, claims process, and wellness benefits grounded in plan documents |

---

### 1.3 Systems & Access Required

> **Pre-Requisites — Share this with participants before the session**
> 1. Salesforce Developer Org or SDO with Data Cloud provisioned and enabled.
> 2. Agentforce enabled in the org (Service Agent, Einstein Studio, Prompt Builder).
> 3. Data Cloud Admin or Data Cloud Operator permission set assigned to the user.
> 4. Sample CSV files provided in this document.
> 5. For Advanced level: plan benefit document content provided in Section 5 — copy into a `.txt` or Word file and upload.

---

## 2. Sample Data

> *Three CSV files are used in this use case. Key intentional design choices:*
> - *The Claims CSV uses `MemberPhone` as the linking field — not email. Some members have slightly different email formats across systems.*
> - *The Wellness CSV has a `WellnessScore` field (0–100) that needs to be transformed into a `EngagementTier` formula field at ingestion.*
> - *These mismatches and transformations are the learning anchors for this use case.*

---

### 2.1 File 1 — CRM Member Profiles (`crm_members.csv`)

**Source:** Salesforce CRM Service Cloud — connected via DSO.

> *In the exercise, participants will use the Salesforce CRM DSO connector to pull the Contact object. This CSV shows the data shape.*

**Sample rows:**

| MemberId | FirstName | Email | Phone | PlanType | City | Note |
|---|---|---|---|---|---|---|
| M001 | Kavitha | kavitha.rajan@gmail.com | 9841001122 | Family Floater | Chennai | |
| M002 | Suresh | suresh.kumar@yahoo.com | 9790112233 | Individual | Coimbatore | |
| M003 | Preethi | preethi.s@outlook.com | 9884223344 | Corporate | Bangalore | ⚠️ Email differs in Claims |
| M004 | Arumugam | arumugam.v@gmail.com | 9865334455 | Individual | Madurai | |
| M005 | Divya | divya.nair@gmail.com | 9944445566 | Family Floater | Chennai | |

**Full CSV — save as `crm_members.csv`:**

```csv
MemberId,FirstName,LastName,Email,Phone,City,PlanType,MemberSince
M001,Kavitha,Rajan,kavitha.rajan@gmail.com,9841001122,Chennai,Family Floater,2019-06-15
M002,Suresh,Kumar,suresh.kumar@yahoo.com,9790112233,Coimbatore,Individual,2020-03-22
M003,Preethi,Sundaram,preethi.s@outlook.com,9884223344,Bangalore,Corporate,2021-08-10
M004,Arumugam,Velayutham,arumugam.v@gmail.com,9865334455,Madurai,Individual,2018-11-05
M005,Divya,Nair,divya.nair@gmail.com,9944445566,Chennai,Family Floater,2022-01-30
M006,Rajesh,Babu,rajesh.babu@hotmail.com,9876556677,Salem,Corporate,2020-07-18
M007,Meenakshi,Sundaram,meenakshi.s@gmail.com,9788667788,Trichy,Individual,2019-04-25
M008,Karthikeyan,Raja,karthikeyan.r@gmail.com,9655778899,Bangalore,Family Floater,2021-12-01
M009,Anitha,Krishnan,anitha.k@yahoo.com,9543889900,Chennai,Individual,2023-02-14
M010,Balamurugan,Selvam,balamurugan.s@gmail.com,9432990011,Coimbatore,Corporate,2020-09-08
M011,Lakshmi,Prabhu,lakshmi.p@outlook.com,9321001122,Madurai,Individual,2022-05-19
M012,Senthil,Nathan,senthil.n@gmail.com,9210112233,Chennai,Family Floater,2018-03-07
M013,Vijayalakshmi,Mohan,vijaya.m@gmail.com,9109223344,Bangalore,Corporate,2021-10-22
M014,Murugesan,Pillai,murugesan.p@hotmail.com,9998334455,Trichy,Individual,2019-08-14
M015,Saranya,Devi,saranya.d@gmail.com,9887445566,Chennai,Family Floater,2023-06-01
```

---

### 2.2 File 2 — Claims Records (`claims_records.csv`)

**Source:** Finance system — ingested as a DLO via CSV upload.

> *Important: Some email addresses here differ slightly from the CRM (e.g., `preethi.sundaram@outlook.com` vs `preethi.s@outlook.com`). Phone number is the reliable match field. This is intentional — participants must configure IR to handle this.*

**Full CSV — save as `claims_records.csv`:**

```csv
ClaimId,MemberEmail,MemberPhone,ClaimDate,ClaimAmount,DiagnosisCategory,ClaimStatus,HospitalName
CL001,kavitha.rajan@gmail.com,9841001122,2024-01-15,45000,Cardiac,Approved,Apollo Chennai
CL002,suresh.kumar@yahoo.com,9790112233,2024-02-20,12000,Orthopaedic,Approved,PSG Coimbatore
CL003,preethi.sundaram@outlook.com,9884223344,2023-11-08,78000,Oncology,Approved,Manipal Bangalore
CL004,arumugam.v@gmail.com,9865334455,2024-03-05,8500,General,Approved,Govt Hospital Madurai
CL005,divya.nair@gmail.com,9944445566,2023-12-18,22000,Cardiac,Pending,Fortis Chennai
CL006,rajesh.babu@hotmail.com,9876556677,2024-01-28,5500,General,Approved,Salem Medical
CL007,meenakshi.s@gmail.com,9788667788,2024-04-10,95000,Oncology,Approved,KIMS Trichy
CL008,karthikeyan.r@gmail.com,9655778899,2023-10-22,31000,Cardiac,Approved,Columbia Asia Bangalore
CL009,anitha.k@yahoo.com,9543889900,2024-02-14,6500,General,Rejected,Apollo Chennai
CL010,balamurugan.s@gmail.com,9432990011,2024-03-30,18000,Diabetes,Approved,PSG Coimbatore
CL011,lakshmi.p@outlook.com,9321001122,2023-09-15,42000,Orthopaedic,Approved,Meenakshi Madurai
CL012,senthil.n@gmail.com,9210112233,2024-01-05,67000,Cardiac,Approved,Fortis Chennai
CL013,vijaya.mohan@gmail.com,9109223344,2023-08-20,125000,Oncology,Approved,Manipal Bangalore
CL014,murugesan.p@hotmail.com,9998334455,2024-04-25,9000,General,Pending,Govt Hospital Trichy
CL015,saranya.d@gmail.com,9887445566,2024-05-01,15000,Diabetes,Approved,Apollo Chennai
CL016,kavitha.rajan@gmail.com,9841001122,2023-08-10,38000,Cardiac,Approved,Apollo Chennai
CL017,meenakshi.s@gmail.com,9788667788,2023-06-15,88000,Oncology,Approved,KIMS Trichy
CL018,senthil.n@gmail.com,9210112233,2023-04-22,52000,Cardiac,Approved,Fortis Chennai
CL019,karthikeyan.r@gmail.com,9655778899,2024-02-28,27000,Diabetes,Approved,Columbia Asia Bangalore
CL020,balamurugan.s@gmail.com,9432990011,2023-11-11,21000,Diabetes,Approved,PSG Coimbatore
```

---

### 2.3 File 3 — Wellness Engagement (`wellness_engagement.csv`)

**Source:** Wellness program system — ingested as DLO with Transformation.

> *Key transformation required: `WellnessScore` (0–100 integer) must be transformed into a derived field `EngagementTier`:*
> - *Score >= 70 → 'Active'*
> - *Score >= 40 → 'Moderate'*
> - *Score < 40 → 'Disengaged'*
>
> *Also: `LastCheckupDate` must be cast to Date type. `AnnualCheckupDone` is Y/N text — participants should map it as-is (no transformation needed, just note it for use in CI).*

**Full CSV — save as `wellness_engagement.csv`:**

```csv
WellnessId,MemberPhone,MemberEmail,LastCheckupDate,AnnualCheckupDone,WellnessAppLogins,WellnessScore,PreventiveCareParticipation
W001,9841001122,kavitha.rajan@gmail.com,2024-02-10,Y,45,72,Y
W002,9790112233,suresh.kumar@yahoo.com,2023-06-15,N,8,28,N
W003,9884223344,preethi.s@outlook.com,2024-01-20,Y,62,85,Y
W004,9865334455,arumugam.v@gmail.com,2022-11-30,N,3,15,N
W005,9944445566,divya.nair@gmail.com,2023-09-05,N,12,35,N
W006,9876556677,rajesh.babu@hotmail.com,2024-03-18,Y,38,55,Y
W007,9788667788,meenakshi.s@gmail.com,2022-08-22,N,5,18,N
W008,9655778899,karthikeyan.r@gmail.com,2023-12-01,Y,29,42,N
W009,9543889900,anitha.k@yahoo.com,2024-04-10,Y,51,68,Y
W010,9432990011,balamurugan.s@gmail.com,2022-10-15,N,7,22,N
W011,9321001122,lakshmi.p@outlook.com,2024-01-05,Y,44,63,Y
W012,9210112233,senthil.n@gmail.com,2022-05-18,N,4,12,N
W013,9109223344,vijaya.m@gmail.com,2023-11-28,Y,55,74,Y
W014,9998334455,murugesan.p@hotmail.com,2023-03-10,N,9,31,N
W015,9887445566,saranya.d@gmail.com,2024-05-15,Y,48,66,Y
```

---

## 3. Level 1 — Beginner

> 🟢 **BEGINNER LEVEL — Ingest, Map & Unify**

**Objective:** Connect all three data sources to Data Cloud, map to DMOs, run Identity Resolution, and verify a unified Member 360 profile exists for each member.

---

### 3.1 Step-by-Step: Create the CRM Member Data Stream (DSO)

1. In Data Cloud, go to **Data Streams** and click **New**.
2. Select **Salesforce CRM** as the connector type.
3. Choose the **Contact** object.
4. Select fields: `Id`, `FirstName`, `LastName`, `Email`, `Phone`, `MailingCity`, `MediCare360_Plan_Type__c` (or the closest available field in your org).
5. Set refresh schedule to **Hourly**.
6. Save and deploy. Wait for the first run to complete.

> 💡 **Tip:** In a real healthcare project, the CRM object would be a Person Account (Health Cloud). For this exercise we use the Contact object to keep it accessible in any Dev Org. The mapping logic is the same.

---

### 3.2 Step-by-Step: Create Claims DLO (CSV Upload)

1. In Data Cloud, go to **Data Streams** and click **New**.
2. Select **File Upload**. Upload `claims_records.csv`.
3. Review auto-detected types:
   - Change `ClaimDate` → **Date** (format: YYYY-MM-DD)
   - Change `ClaimAmount` → **Number**
4. Name the Data Stream: `Claims_Records_DS` and deploy.

> 💡 **Tip:** Notice `MemberEmail` in this file is the link to the member — but some emails differ slightly from CRM. The reliable match is `MemberPhone`. Point this out to participants as the reason IR needs multiple match rules.

---

### 3.3 Step-by-Step: Create Wellness DLO with Transformation

1. In Data Cloud, go to **Data Streams** and click **New**.
2. Select **File Upload**. Upload `wellness_engagement.csv`.
3. Fix field types:
   - `LastCheckupDate` → **Date**
   - `WellnessScore` → **Number**
   - `WellnessAppLogins` → **Number**
4. Click **Add Formula Field**. Create `EngagementTier` (Text):

```
IF(WellnessScore >= 70, "Active", IF(WellnessScore >= 40, "Moderate", "Disengaged"))
```

5. Name: `Wellness_Engagement_DS` and deploy.

> 💡 **Tip:** The `EngagementTier` formula field is a key teaching moment. Ask participants: why do we create this at ingestion time rather than in the Calculated Insight SQL? Answer: it becomes a reusable field available everywhere in Data Cloud — segments, CI, activations — not just in one SQL query.

---

### 3.4 Step-by-Step: Map Data Streams to DMOs

| Data Stream | Map To DMO | Key Field Mappings | Primary Key |
|---|---|---|---|
| CRM Members DS | Individual (standard) | Email → ContactPointEmail, Phone → ContactPointPhone, FirstName → FirstName, Id → SourceRecordId | MemberId |
| Claims Records DS | Sales Order (standard) | ClaimId → Id, MemberEmail → ContactPointEmail, MemberPhone → ContactPointPhone, ClaimAmount → TotalAmount, ClaimDate → OrderedDate | ClaimId |
| Wellness Engagement DS | Individual (standard) | MemberEmail → ContactPointEmail, MemberPhone → ContactPointPhone, WellnessId → SourceRecordId | WellnessId |

> 💡 **Tip:** Both CRM Members and Wellness Engagement map to the Individual DMO. Claims map to Sales Order. This is the same pattern as UC1 — reinforce that the DMO mapping layer is the standardisation step that makes identity resolution and segmentation possible.

---

### 3.5 Step-by-Step: Configure Identity Resolution

1. Go to **Identity Resolution** and click **New Ruleset**.
2. Name: `MediCare360_Member_IR`.
3. Add **Match Rule 1: Exact Phone Match**
   - Field: `ContactPointPhone` | Match Type: Exact
4. Add **Match Rule 2: Exact Email Match**
   - Field: `ContactPointEmail` | Match Type: Exact | Normalization: Lowercase
5. Set **Reconciliation Rule**: Most Recent Source Wins.
6. Save and Run.
7. Search for `Kavitha Rajan` in **Unified Individual** — verify CRM + Wellness records are merged.

> **Expected Outcome — Identity Resolution**
> - Kavitha Rajan (M001 + W001) — merged via exact phone + email.
> - Preethi Sundaram (M003 + W003) — merged via phone (emails differ: `preethi.s` vs `preethi.sundaram`).
> - Vijayalakshmi Mohan (M013 + W013) — merged via phone (emails differ: `vijaya.m` vs `vijaya.mohan`).
> - Unified Individual count should be **15** — one per member, not 30.
> - Claims records linked to Unified Individual via `ContactPointEmail` join in the Sales Order DMO.

---

## 4. Level 2 — Intermediate

> 🟡 **INTERMEDIATE LEVEL — Calculated Insights, Segmentation, Activation & Data Actions**

**Objective:** Identify members who are both high-claim-risk AND disengaged from wellness — the care gap segment. Activate this list to CRM for care coordinator outreach. Trigger a Data Action to auto-create a case when a new member enters this group.

> *Pre-requisite: Beginner level complete.*

---

### 4.1 Step-by-Step: Create the Calculated Insight (Care Gap Score)

1. In Data Cloud, go to **Calculated Insights** and click **New**.
2. Name: `Member_Care_Gap_Score`.
3. Enter the following SQL:

```sql
SELECT
    ui.Id__c                                    AS uid__c,
    COUNT(so.Id__c)                             AS total_claims__c,
    SUM(so.TotalAmount__c)                      AS total_claim_amount__c,
    MAX(so.OrderedDate__c)                      AS last_claim_date__c,
    DATEDIFF(TODAY(), MAX(so.OrderedDate__c))   AS days_since_last_claim__c,
    CASE
        WHEN SUM(so.TotalAmount__c) > 50000
        THEN 'High'
        WHEN SUM(so.TotalAmount__c) > 20000
        THEN 'Medium'
        ELSE 'Low'
    END                                         AS claim_risk_tier__c
FROM UnifiedIndividual__dlm ui
JOIN SalesOrder__dlm so
    ON ui.Id__c = so.ContactPointEmailId__c
GROUP BY ui.Id__c
```

4. Save and Run. Preview data — verify one row per member with correct risk tiers.

> 💡 **Tip:** This CI gives claim risk. The wellness engagement tier (Active / Moderate / Disengaged) is already on the Unified Individual profile from the `EngagementTier` formula field we created at ingestion. In the segment we will combine both — this is a powerful pattern: pre-compute metrics at ingestion (wellness tier) and at CI level (claim risk), then combine in segments.

---

### 4.2 Step-by-Step: Build the Care Gap Segment

1. In Data Cloud, go to **Segments** and click **New**.
2. Name: `High_Risk_Disengaged_Members`.
3. Segment On: **Individual**.
4. Add Filter 1: Calculated Insight → `Member_Care_Gap_Score` → `claim_risk_tier__c` → Equals → `High`.
5. Add Filter 2 (AND): Wellness Engagement DS → `EngagementTier` → Equals → `Disengaged`.
6. Set Publish Schedule: **Every 12 hours**.
7. Save and Publish. Note the member count.

> **Expected Segment Members (based on sample data)**
> - Meenakshi Sundaram — Total Claims: ₹1,83,000 | EngagementTier: Disengaged → ✅ Qualifies
> - Senthil Nathan — Total Claims: ₹1,19,000 | EngagementTier: Disengaged → ✅ Qualifies
> - Arumugam Velayutham — Total Claims: ₹8,500 | EngagementTier: Disengaged → ❌ Low claim risk
>
> Note: `claim_risk_tier__c` threshold is > 50,000 total. Adjust if needed for demo.

---

### 4.3 Step-by-Step: Activate to Salesforce CRM

1. In CRM, create a Campaign: `MediCare360 Care Coordinator Outreach Q2`.
2. In Data Cloud, go to **Activations** → **New**.
3. Select **Salesforce CRM** target.
4. Choose segment: `High_Risk_Disengaged_Members`.
5. Map to **Campaign Member**. Map: Unified Individual Email → Contact Email, Name → Name.
6. Set Contact Point: **Email**.
7. Save and run. Verify Campaign Members in CRM.

---

### 4.4 Step-by-Step: Configure a Data Action

1. In Salesforce CRM, create a Flow: Trigger = Platform Event (Data Cloud Triggered Flow), Action = Create a **Case** with Subject `Care Coordinator Assignment — High Risk Member`, Status = `New`, Priority = `High`.
2. In Data Cloud, go to **Data Actions** → **New**.
3. Name: `High_Risk_Member_Alert`.
4. Trigger: Segment Membership Change → `High_Risk_Disengaged_Members` → **On Entry**.
5. Action Type: **Salesforce Platform Event**.
6. Map payload: Individual Id, Email, `claim_risk_tier__c`, `EngagementTier`.
7. Save and Activate.

> 💡 **Tip:** Note that we create a **Case** here (not a Task like in UC1). In healthcare, a Case represents a service interaction — it is the right CRM object for care coordinator assignment. This teaches participants that Data Actions are flexible — the Salesforce Flow on the other end can do anything.

---

## 5. Level 3 — Advanced

> 🔴 **ADVANCED LEVEL — Search Index, Retriever, Prompt Builder & Agentforce**

**Objective:** Build an Agentforce Service Agent that answers member questions about their plan benefits, claims process, and wellness program — grounded in MediCare360's actual plan documents.

> *Pre-requisite: Beginner level complete. Agentforce and Einstein Studio enabled.*

---

### 5.1 Prepare Knowledge Content

Use any of these options:

- **Option A:** Copy the plan benefit content below into a `.txt` or Word file and upload to Data Cloud.
- **Option B:** Create Knowledge Articles in Service Cloud using the same content.
- **Option C:** Both.

**Sample knowledge content — copy into your knowledge source:**

```
MediCare360 Member Benefits Guide — 2025

PLAN TYPES
Individual Plan: Covers the primary member only. Annual sum insured: Rs.3 lakh, Rs.5 lakh, or Rs.10 lakh options.
Family Floater Plan: Covers member, spouse, and up to 2 children. Annual sum insured: Rs.5 lakh or Rs.10 lakh.
Corporate Group Plan: Employer-sponsored. Coverage depends on employer policy. Contact HR for details.

CLAIMS PROCESS
Cashless Claims: Available at all network hospitals. Show your MediCare360 card at the hospital reception.
Reimbursement Claims: For non-network hospitals. Submit bills within 30 days of discharge to claims@medicare360.in.
Pre-authorisation: Required for planned surgeries above Rs.50,000. Apply 72 hours in advance via the member portal.
Emergency Admissions: No pre-auth required. Notify MediCare360 within 24 hours of admission.
Documents required: Discharge summary, original bills, prescription copies, lab reports, and claim form.

WAITING PERIODS
Pre-existing diseases: 2-year waiting period for coverage.
Specific illnesses (cardiac, oncology, orthopaedic): 1-year waiting period.
Maternity benefits: 2-year waiting period. Covered under Family Floater plans only.
General illnesses: No waiting period after first 30 days of policy inception.

WELLNESS PROGRAM
Annual health checkup: Free for all members. Schedule at any partner clinic via the member portal.
Wellness app: Download the MediCare360 app. Complete weekly health goals to earn wellness points.
Wellness points: 100 points = Rs.100 discount on renewal premium.
Preventive care: Free consultations for diabetes screening, blood pressure monitoring, and vision tests.
Platinum wellness status: Members with wellness score above 80 get 10% discount on renewal premium.

RENEWALS AND CANCELLATIONS
Renewal: Policy auto-renews 30 days before expiry. Premium payment due 15 days before expiry.
Grace period: 30-day grace period after expiry. Coverage suspended during grace period.
Cancellation: Request cancellation 30 days before renewal. Pro-rata refund applicable.
```

---

### 5.2 Step-by-Step: Create the Search Index

1. In Data Cloud, go to **Search Indexes** and click **New**.
2. Name: `MediCare360_Benefits_Index`.
3. Select your uploaded file or Knowledge Article DMO as the data source.
4. Chunking: chunk size **300 tokens**, overlap **50 tokens**.
5. Search Type: **Hybrid**.
6. Save and Build. Wait for **Active** status.

---

### 5.3 Step-by-Step: Create the Retriever

1. In **Einstein Studio**, go to **Retrievers** → **New Retriever**.
2. Select **Individual Retriever**.
3. Data Space: **default**.
4. Select the DMO linked to your Search Index.
5. Select Search Index: `MediCare360_Benefits_Index`.
6. Field selection: select only **`Chunk__c`**.
7. Name: `MediCare360_Benefits_Retriever`. Save.

---

### 5.4 Step-by-Step: Build the Prompt Template

1. In Setup, open **Prompt Builder** → **New Prompt Template** → type: **Field Generation**.
2. Name: `MediCare360_Member_Query_Answer`.
3. Paste this template:

```
You are a helpful member support assistant for MediCare360, a health insurance provider.
Answer the member's question using ONLY the information in the context below.
If the answer is not in the context, say: 'I don't have that information right now.
Please call our member helpline at 1800-360-MEDI or email support@medicare360.in.'

Do not make up information. Be clear, empathetic, and concise.

CONTEXT:
{!$Input:Retriever.MediCare360_Benefits_Retriever.Chunk__c}

MEMBER QUESTION:
{!$Input:question}

ANSWER:
```

4. Save and **Preview**. Test with: `How do I file a reimbursement claim?`
   Verify the response matches the claims process section of the document.

---

### 5.5 Step-by-Step: Deploy the Agentforce Service Agent

1. In Setup, open **Agentforce Builder** → create new agent from **Service Agent** template.
2. Name: `MediCare360 Member Support Agent`.
3. Add Topic: `Member Benefits and Claims`.
   - Description: *Handles member questions about plan coverage, claims process, waiting periods, wellness program, and renewals.*
4. Add Action: `Answer Member Query`.
   - Action Type: Prompt Template → `MediCare360_Member_Query_Answer`.
5. Activate and open **Agent Preview**.
6. Test with:
   - *How do I claim for a non-network hospital?*
   - *What is the waiting period for cardiac surgery?*
   - *How do I earn a discount on my renewal premium?*

> **Expected Agent Behaviour**
>
> **Q: How do I claim for a non-network hospital?**
> Expected: Agent explains reimbursement claim process — submit bills within 30 days, documents required, email address. Should not confuse cashless and reimbursement processes.
>
> **Q: What is the waiting period for cardiac surgery?**
> Expected: Agent says 1-year waiting period for cardiac conditions (specific illness waiting period).
>
> **Q: How do I earn a discount on renewal premium?**
> Expected: Agent mentions wellness points (100 points = Rs.100 discount) AND Platinum wellness status (score > 80 = 10% discount).
>
> If agent hallucinates — check that `Chunk__c` is being returned by retriever. Test retriever in Prompt Builder Preview first.

---

## 6. Common Mistakes & How to Help

| Mistake | Where It Happens | How to Help |
|---|---|---|
| Identity Resolution does not merge Preethi Sundaram (email mismatch) | Beginner — IR setup | Confirm Match Rule 1 is Exact Phone match — not just email. Phone is the reliable field in this dataset. Check that `ContactPointPhone` is mapped correctly in the Individual DMO for both CRM and Wellness streams. |
| Claims not showing up under the Unified Individual | Beginner — DMO mapping | Claims map to Sales Order DMO — not Individual. The join happens via `ContactPointEmailId__c` in the CI SQL. If emails differ, some claims may not join. Walk participant through the join logic. |
| CI returns no rows | Intermediate — CI | The join `ON ui.Id__c = so.ContactPointEmailId__c` requires that the Sales Order DMO has `ContactPointEmail` mapped from the Claims stream. Verify DMO mapping for Claims includes `MemberEmail → ContactPointEmail`. |
| Segment has 0 members | Intermediate — Segment | Two filters combined with AND — both must be true. Check each filter individually first. The `EngagementTier` field must be on the Individual DMO from the Wellness stream mapping. |
| Agent answers correctly in Preview but not in Agent Builder | Advanced — Agentforce | The action must explicitly reference the Prompt Template. Check the topic description is detailed enough for the agent to route queries correctly. |
| Waiting period question gives wrong answer | Advanced — Agentforce | The chunk containing waiting period information may not be retrieved if the query is too vague. Test with more specific phrasing. Adjust chunk size if needed. |

---

## 7. Discussion Questions for the Team Session

### After Beginner

1. In this use case, we used phone number as the primary match rule (not email). Why? What does this tell you about how to choose match rules for a client project?
2. The Wellness Engagement stream also maps to the Individual DMO — the same as CRM Members. How does Data Cloud know not to create duplicate Unified Individuals from the same stream?
3. What is the difference between `ContactPointEmail` and `ContactPointPhone` as match fields in Identity Resolution? When would you use one over the other?

### After Intermediate

1. The Care Gap segment uses TWO filters combined with AND — claim risk AND wellness disengagement. What happens to the segment size if you change AND to OR? Is that a better or worse segment for the care coordinator use case? Why?
2. We triggered a Case creation (not a Task like in UC1) via the Data Action. What are the implications of using a Case — what additional automation can you build in CRM on top of a Case that you cannot on a Task?
3. The `EngagementTier` was created as a formula field at ingestion. Could you have calculated it in the CI SQL instead? What are the trade-offs?

### After Advanced

1. A member asks: *'Is my pre-existing diabetes condition covered from day one?'* What should the agent say based on the policy doc? Test it. Does the agent get it right?
2. The prompt says *'Be clear, empathetic, and concise.'* Does the tone instruction affect the quality of answers? Remove it and compare. What did you notice?
3. Healthcare data is sensitive. If a member asks the agent about a specific claim they filed — *'What is the status of my claim from January?'* — should the agent answer? What would you need to add to the architecture to handle this safely?

---

## 8. Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA INGESTION LAYER                         │
│  CRM Members (DSO)  │  Claims Records (CSV→DLO)                │
│  Wellness Engagement (CSV→DLO + Transformation: EngagementTier)│
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DATA MODEL LAYER                             │
│  Individual DMO (CRM Members + Wellness)                       │
│  Sales Order DMO (Claims Records)                              │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                   IDENTITY RESOLUTION                           │
│  Exact Phone Match (primary) + Exact Email Match               │
│  → Unified Individual (Member 360 — one record per member)     │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│               INSIGHTS & ACTIVATION LAYER                       │
│  Calculated Insight: Member_Care_Gap_Score (claim risk tier)   │
│  Segment: High_Risk_Disengaged_Members                         │
│  (CI claim risk = High  AND  EngagementTier = Disengaged)      │
│  → Activation → CRM Campaign (Care Coordinator Outreach)       │
│  → Data Action → Salesforce Flow → CRM Case (High Priority)   │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                  AI & AGENTFORCE LAYER                          │
│  Plan Benefits Document / Knowledge Articles                   │
│  → Search Index (Hybrid) → Retriever                           │
│  → Prompt Builder (empathetic, grounded) → Service Agent       │
│  → Answers: claims process, waiting periods, wellness, renewal │
└─────────────────────────────────────────────────────────────────┘
```

---

*Next: UC3 — DriveSmart Motors (Automotive) | UC4 — CloudPilot Inc (B2B SaaS)*
