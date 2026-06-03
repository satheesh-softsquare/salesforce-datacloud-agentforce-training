# UC2 — Participant Guide | MediCare360 Healthcare
## Data Cloud & Agentforce Training

| Field | Details |
|---|---|
| **Domain** | Healthcare / Insurance |
| **Company** | MediCare360 (fictional health insurance provider) |
| **Levels** | Beginner → Intermediate → Advanced |
| **Org Required** | Dev Org or SDO with Data Cloud enabled |
| **Estimated Time** | 3–5 hours total |

---

> 📌 **How to use this document**
>
> - **Part A — Challenge Card:** Read this first. Attempt the use case on your own.
> - **Part B — Detailed Handout:** Open this only if you are stuck. It has CSV data, task steps, and hints.
>
> Discuss with your team before opening Part B. The goal is to learn, not just finish.

---

# PART A — CHALLENGE CARD
### *Read this first. Attempt before looking at Part B.*

---

## The Company: MediCare360

MediCare360 is a mid-sized health insurance provider covering 2 lakh members across Tamil Nadu and Karnataka. They offer individual health plans, family floater plans, and corporate group health policies.

---

## The Problem

MediCare360 has member data scattered across three systems that do not talk to each other:

| System | What It Contains | The Problem |
|---|---|---|
| **Salesforce CRM** | Member profiles, contact details, support case history | Has member info but no claims or wellness data |
| **Claims Database (CSV)** | Every insurance claim — amount, diagnosis, hospital, status | Some member emails are slightly different from CRM. No shared member ID. |
| **Wellness Engagement (CSV)** | Annual checkup records, wellness app activity, health scores | Managed by a separate team. Never linked to claims or CRM. |

> *When a member named Kavitha Rajan calls the support centre asking about her claim, the agent opens three different screens across two systems to piece together her profile. Meanwhile, the care management team has no way to find members who are both high-claim-risk AND completely disengaged from wellness — the exact people most likely to have a major health event. These members need proactive outreach before it is too late.*

MediCare360 wants Salesforce Data Cloud to build a unified Member 360 — and then act on it.

---

## Your Challenge — Three Levels

| Level | Name | What You Need to Build | You Know You're Done When... |
|---|---|---|---|
| 🟢 **Beginner** | Ingest, Map & Unify | Connect Salesforce CRM, Claims CSV, and Wellness CSV to Data Cloud. Map to the right DMOs. Run Identity Resolution to create one Member 360 profile per person. | You can search for 'Kavitha Rajan' in Unified Individual and see one record combining her CRM profile, claims history, and wellness data. |
| 🟡 **Intermediate** | Care Gap Insights & Activation | Write a Calculated Insight to score members by total claim amount and risk. Build a segment of high-risk members who are also disengaged from wellness. Push that segment to CRM for care coordinator outreach. Trigger a Data Action that auto-creates a high-priority Case when a new member enters this group. | The CRM campaign is auto-populated. A high-priority Case appears in CRM automatically when a member qualifies. |
| 🔴 **Advanced** | AI Member Support Agent | Upload MediCare360's plan benefit document as a knowledge source. Create a Search Index and Retriever. Build a Prompt Template that answers member questions accurately. Deploy an Agentforce Service Agent. | The agent correctly explains the reimbursement claims process and the cardiac surgery waiting period — using the policy document, not guessing. |

---

## What Data Do You Have?

- **Salesforce CRM** — 15 member Contact records already in your org.
- **Claims Database** — 20 claim records as a CSV file.
- **Wellness Engagement** — 15 member wellness records as a CSV file.

> If you need the CSV files or a hint on where to start, ask your facilitator.

---

## Tools & Scope

| Tools & Access You Need | What Is In / Out |
|---|---|
| Salesforce Developer Org or SDO with Data Cloud enabled | ✅ IN: CRM (DSO), CSV uploads, Data Cloud, Agentforce |
| Data Cloud Admin permission set | ✅ IN: Knowledge Articles or uploaded files (Level 3) |
| Agentforce + Einstein Studio enabled (Level 3) | ❌ OUT: Health Cloud, Marketing Cloud, S3, external EHR |

---

---

# PART B — DETAILED HANDOUT
### *Open this only if you are stuck.*

---

## 1. Your Sample Data

Three CSV files are below. Copy each one, save as `.csv`, and use in your exercises.

> ⚠️ **Pay attention to email addresses across files — some are slightly different for the same member. The reliable match field in this dataset is the phone number. This is intentional.**

---

### File 1 — CRM Members → save as `crm_members.csv`

> Connect this via a Salesforce CRM DSO (Data Stream) — not a file upload.

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

### File 2 — Claims Records → save as `claims_records.csv`

> Notice: `MemberEmail` in some rows differs slightly from CRM. `MemberPhone` is consistent.
> `ClaimDate` must be converted to Date type. `ClaimAmount` must be Number.

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

### File 3 — Wellness Engagement → save as `wellness_engagement.csv`

> `LastCheckupDate` must be Date type. `WellnessScore` and `WellnessAppLogins` must be Number.
> You must also add a formula field `EngagementTier` based on `WellnessScore`.

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

## 2. Level 1 — Beginner

> 🟢 **Goal:** Get all three data sources into Data Cloud, map to DMOs, run Identity Resolution, and verify Kavitha Rajan's records from all three systems are merged into one Member 360 profile.

### Task A — Connect Salesforce CRM as a Data Stream (DSO)
- Create a new Data Stream connected to **Salesforce CRM**.
- Select the **Contact** object. Include: `Id`, `FirstName`, `LastName`, `Email`, `Phone`, `MailingCity`.
- Deploy and confirm data is flowing.

### Task B — Ingest Claims Records as a DLO
- Create a new Data Stream using **File Upload**.
- Upload `claims_records.csv`.
- Fix field types: `ClaimDate` → Date, `ClaimAmount` → Number.

### Task C — Ingest Wellness Engagement as a DLO with Transformation
- Upload `wellness_engagement.csv`.
- Fix field types: `LastCheckupDate` → Date, `WellnessScore` → Number, `WellnessAppLogins` → Number.
- Add a **formula field** called `EngagementTier`:
  - If WellnessScore >= 70 → `'Active'`
  - If WellnessScore >= 40 → `'Moderate'`
  - Otherwise → `'Disengaged'`

### Task D — Map to Data Model Objects (DMOs)
- CRM Members → **Individual DMO**
- Wellness Engagement → **Individual DMO** (same DMO — correct)
- Claims Records → **Sales Order DMO**
- Map phone field to `ContactPointPhone` and email to `ContactPointEmail` for all streams.

### Task E — Configure and Run Identity Resolution
- Create a new Identity Resolution ruleset.
- Add match rules — think carefully about which field is the most reliable match field in this dataset.
- *Hint: Look at the email mismatches in the data. What field is always consistent?*
- Run the ruleset and verify in **Unified Individual**.

> 🧠 **Think About It**
> 1. In UC1, we used email as the primary match rule. In this dataset, why is phone a better primary match field? What does this tell you about choosing match rules for a client?
> 2. Preethi Sundaram has `preethi.s@outlook.com` in CRM but `preethi.sundaram@outlook.com` in Claims. Her phone is the same. Which match rule catches this?
> 3. Claims are mapped to the Sales Order DMO — not the Individual DMO. How will Data Cloud link a claim to a member? What field connects them?

> ✅ **You have completed Level 1 when:**
> - All three Data Streams are active and showing correct record counts.
> - Wellness stream has an `EngagementTier` formula field with values Active, Moderate, or Disengaged.
> - Identity Resolution has run and Kavitha Rajan appears as a **single** Unified Individual.
> - Members with email mismatches (Preethi Sundaram, Vijayalakshmi Mohan) are also correctly merged.
> - Unified Individual count is **15** — one per member.

---

## 3. Level 2 — Intermediate

> 🟡 **Goal:** Identify members who are both high-claim-risk AND disengaged from wellness. Push this list to CRM for care coordinator outreach. Auto-create a high-priority Case when a new member qualifies.

> *Pre-requisite: Level 1 complete.*

### Task A — Create a Calculated Insight (Care Gap Score)
- Create a new Calculated Insight named `Member_Care_Gap_Score`.
- Write SQL that gives one row per member with:
  - Total number of claims
  - Total claim amount
  - Most recent claim date
  - A `claim_risk_tier` field: `'High'` if total claim amount > 50,000, `'Medium'` if > 20,000, otherwise `'Low'`

### Task B — Build the Care Gap Segment
- Create a Segment named `High_Risk_Disengaged_Members`.
- Use **two filters combined with AND**:
  - Filter 1: `claim_risk_tier` from your CI = `'High'`
  - Filter 2: `EngagementTier` from Wellness = `'Disengaged'`
- Set refresh: every 12 hours.

### Task C — Activate to Salesforce CRM
- Create a Campaign in CRM: `MediCare360 Care Coordinator Outreach Q2`.
- Create an Activation that pushes `High_Risk_Disengaged_Members` to this Campaign.
- Verify Campaign Members appear in CRM.

### Task D — Set Up a Data Action
- Create a Data Action that fires when a member **enters** the `High_Risk_Disengaged_Members` segment.
- The action should trigger a Flow that creates a **Case** in CRM:
  - Subject: `Care Coordinator Assignment — High Risk Member`
  - Priority: `High`
  - Status: `New`

> 🧠 **Think About It**
> 1. Your segment uses AND — both conditions must be true. What happens to the member count if you change it to OR? Would that be a better or worse list for care coordinators?
> 2. In UC1 we created a Task via Data Action. Here we create a Case. What is the difference in a healthcare context? Why is a Case more appropriate here?
> 3. You created `EngagementTier` as a formula field at ingestion time. Could you have put the same logic in the CI SQL instead? What are the trade-offs of each approach?

> ✅ **You have completed Level 2 when:**
> - Calculated Insight runs with one row per member and correct risk tiers.
> - Segment `High_Risk_Disengaged_Members` has at least 1 member.
> - CRM Campaign is auto-populated via Activation.
> - A high-priority CRM Case is auto-created when a member enters the segment.

---

## 4. Level 3 — Advanced

> 🔴 **Goal:** Build an AI Agentforce Service Agent that answers member questions about plan coverage, claims process, waiting periods, and wellness — grounded in MediCare360's actual policy document.

> *Pre-requisite: Level 1 complete. Agentforce must be enabled.*

### Task A — Prepare Your Knowledge Source
Choose one or both:
- **Option A:** Copy the text below into a Word doc or `.txt` file and upload to Data Cloud.
- **Option B:** Create Knowledge Articles in Salesforce Service Cloud.

```
MediCare360 Member Benefits Guide — 2025

PLAN TYPES
Individual Plan: Covers the primary member only. Sum insured: Rs.3 lakh, Rs.5 lakh, or Rs.10 lakh.
Family Floater Plan: Covers member, spouse, and up to 2 children. Sum insured: Rs.5 lakh or Rs.10 lakh.
Corporate Group Plan: Employer-sponsored. Coverage depends on employer policy. Contact HR for details.

CLAIMS PROCESS
Cashless Claims: Available at all network hospitals. Show MediCare360 card at hospital reception.
Reimbursement Claims: For non-network hospitals. Submit bills within 30 days of discharge to claims@medicare360.in.
Pre-authorisation: Required for planned surgeries above Rs.50,000. Apply 72 hours in advance via the member portal.
Emergency Admissions: No pre-auth required. Notify MediCare360 within 24 hours of admission.
Documents required: Discharge summary, original bills, prescription copies, lab reports, and claim form.

WAITING PERIODS
Pre-existing diseases: 2-year waiting period.
Specific illnesses (cardiac, oncology, orthopaedic): 1-year waiting period.
Maternity benefits: 2-year waiting period. Family Floater plans only.
General illnesses: No waiting period after first 30 days.

WELLNESS PROGRAM
Annual health checkup: Free for all members. Schedule via the member portal.
Wellness app: Download the MediCare360 app. Complete weekly goals to earn wellness points.
Wellness points: 100 points = Rs.100 discount on renewal premium.
Preventive care: Free consultations for diabetes screening, blood pressure, and vision tests.
Platinum wellness status: Score above 80 = 10% discount on renewal premium.

RENEWALS AND CANCELLATIONS
Renewal: Auto-renews 30 days before expiry. Premium due 15 days before expiry.
Grace period: 30 days after expiry. Coverage suspended during grace period.
Cancellation: Request 30 days before renewal. Pro-rata refund applicable.
```

### Task B — Create a Search Index
- In Data Cloud, create a **Search Index** on your knowledge source.
- Choose **Hybrid** search type.
- Wait for **Active** status.

### Task C — Create a Retriever
- In **Einstein Studio**, create a Retriever pointing to your Search Index.
- Return only the **text content field** — not all metadata.

### Task D — Build a Prompt Template
- In **Prompt Builder**, create a **Field Generation** prompt template.
- Instruct the AI to answer using ONLY the retrieved context.
- Add a tone instruction — this is a healthcare context, so the agent should be empathetic.
- Include a fallback instruction for when the answer is not in the document.
- Test with **Preview** before moving to the agent.

### Task E — Deploy the Agentforce Service Agent
- Create a new **Service Agent** named `MediCare360 Member Support Agent`.
- Add a Topic for member benefits and claims questions.
- Add an Action linked to your Prompt Template.
- Activate and test with:
  - *How do I claim for a non-network hospital?*
  - *What is the waiting period for cardiac surgery?*
  - *How do I earn a discount on my renewal premium?*

> 🧠 **Think About It**
> 1. A member asks: *'Is my pre-existing diabetes condition covered from day one?'* What should the agent say? Test it. Does it get it right?
> 2. The prompt says to be 'empathetic'. Does a tone instruction actually change the quality or style of answers? Try removing it and compare.
> 3. A member asks: *'What is the status of my claim from January?'* Should the agent answer this? What is missing from the current architecture to handle this safely?

> ✅ **You have completed Level 3 when:**
> - Search Index is **Active**.
> - Retriever is configured and pointing to the correct index.
> - Prompt Builder Preview returns a correct, grounded answer for a test question.
> - Agent correctly explains the reimbursement claims process step by step.
> - Agent correctly states the 1-year waiting period for cardiac conditions.
> - When asked something not in the document, the agent says it doesn't know and gives the support contact — it does not guess.

---

## 5. Your Notes

**Level 1 — What I learned / What was difficult:**

```
(write here)
```

**Level 2 — What I learned / What was difficult:**

```
(write here)
```

**Level 3 — What I learned / What was difficult:**

```
(write here)
```

**Questions for the team:**

```
(write here)
```

---

> 💬 *Good luck! If you get stuck, discuss with your team. That is part of the learning.*
