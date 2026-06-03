# UC1 — Facilitator Guide | NovaMart Retail
## Data Cloud & Agentforce Training

> ⚠️ **THIS DOCUMENT IS FOR FACILITATOR USE ONLY. DO NOT SHARE WITH PARTICIPANTS.**

---

| Field | Details |
|---|---|
| **Domain** | Retail / E-Commerce |
| **Company** | NovaMart (fictional) |
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

NovaMart is a fast-growing D2C (Direct-to-Consumer) retail brand that sells electronics, lifestyle products, footwear, and apparel. They operate through their website, mobile app, and physical stores across 12 cities in India.

NovaMart has grown quickly over the last three years, but their customer data is scattered across three systems:

- **CRM (Salesforce Sales Cloud)** — where the sales team tracks leads and contacts.
- **Loyalty Program Database** — maintained separately, exported monthly as a CSV file.
- **Purchase History System** — an ERP export that lists every transaction by customer email or phone.

> *The core problem: The same customer, Arjun Sharma, exists as three different records across these three systems — with slightly different email addresses and no shared unique ID. The marketing team cannot tell how many unique customers they actually have. The sales team cannot see a customer's purchase or loyalty history when they call them. Leadership cannot report on true customer lifetime value.*

NovaMart wants to fix this using Salesforce Data Cloud — creating one unified, golden record per customer that combines data from all three sources. Once unified, they want to find their most valuable lapsed customers and build an AI-powered agent to assist the service team.

---

### 1.2 What the Participant Will Build

| Level | What They Build | Data Cloud Features | Outcome |
|---|---|---|---|
| 🟢 **Beginner** | Ingest 3 data sources, map to DMOs, resolve identity | DSO, DLO, DLO+Transform, DMO, Identity Resolution | One unified customer profile per person |
| 🟡 **Intermediate** | Build a Calculated Insight for RFM scoring, create segment, activate to CRM, set up Data Action | Calculated Insights, Segments, Activation, Data Actions | Dynamic CRM campaign auto-populated with high-value lapsed customers |
| 🔴 **Advanced** | Build a Search Index, create Retriever, configure Prompt Builder, deploy Agentforce Service Agent | Search Index, Retriever, Prompt Builder, Agentforce Agent | AI agent answers customer product and policy questions grounded in NovaMart knowledge base |

---

### 1.3 Systems & Access Required

> **Pre-Requisites — Share this with participants before the session**
> 1. A Salesforce Developer Org or SDO with Data Cloud provisioned and enabled.
> 2. Agentforce enabled in the org (Service Agent, Einstein Studio, Prompt Builder).
> 3. Data Cloud Admin or Data Cloud Operator permission set assigned to the user.
> 4. Sample CSV files provided in this document (copy-paste and save as .csv).
> 5. For Advanced level: at least one PDF or Word document to upload as knowledge content. A sample product FAQ content is included in Section 5 of this guide.

---

## 2. Sample Data

> *Three CSV files are used in this use case. They are intentionally designed with mismatched email addresses for some records to simulate real-world identity resolution challenges. Ask participants to observe these discrepancies — it is the key learning point for Identity Resolution.*

---

### 2.1 File 1 — CRM Contacts (`crm_contacts.csv`)

**Source:** Salesforce CRM Sales Cloud — connected via DSO (Data Stream from CRM).

> *In the actual exercise, participants will use the Salesforce CRM DSO connector — this CSV is provided so they understand the data shape.*

**Sample rows (note the highlighted mismatches):**

| ContactId | FirstName | Email | Phone | City | LeadSource | Note |
|---|---|---|---|---|---|---|
| C001 | Arjun | arjun.sharma@gmail.com | 9876543210 | Mumbai | Web | |
| C002 | Priya | priya.nair@yahoo.com | 9845001122 | Bangalore | Referral | |
| C003 | Ravi | ravi.kumar@outlook.com | 9901234567 | Chennai | Web | ⚠️ Email differs in Loyalty |
| C004 | Sneha | sneha.patel@gmail.com | 9712345678 | Ahmedabad | Event | |
| C005 | Karthik | karthik.rao@gmail.com | 9988776655 | Hyderabad | Web | |

**Full CSV — save as `crm_contacts.csv`:**

```csv
ContactId,FirstName,LastName,Email,Phone,City,AccountName,LeadSource
C001,Arjun,Sharma,arjun.sharma@gmail.com,9876543210,Mumbai,NovaMart,Web
C002,Priya,Nair,priya.nair@yahoo.com,9845001122,Bangalore,NovaMart,Referral
C003,Ravi,Kumar,ravi.kumar@outlook.com,9901234567,Chennai,NovaMart,Web
C004,Sneha,Patel,sneha.patel@gmail.com,9712345678,Ahmedabad,NovaMart,Event
C005,Karthik,Rao,karthik.rao@gmail.com,9988776655,Hyderabad,NovaMart,Web
C006,Deepa,Menon,deepa.menon@hotmail.com,9123456789,Kochi,NovaMart,Referral
C007,Vikram,Singh,vikram.singh@gmail.com,9234567890,Delhi,NovaMart,Web
C008,Ananya,Das,ananya.das@gmail.com,9345678901,Kolkata,NovaMart,Event
C009,Rahul,Verma,rahul.verma@yahoo.com,9456789012,Pune,NovaMart,Web
C010,Meena,Iyer,meena.iyer@gmail.com,9567890123,Chennai,NovaMart,Referral
C011,Suresh,Joshi,suresh.joshi@outlook.com,9678901234,Jaipur,NovaMart,Web
C012,Lakshmi,Reddy,lakshmi.reddy@gmail.com,9789012345,Hyderabad,NovaMart,Web
C013,Arun,Nambiar,arun.nambiar@gmail.com,9890123456,Kochi,NovaMart,Referral
C014,Pooja,Shah,pooja.shah@yahoo.com,9901234568,Mumbai,NovaMart,Event
C015,Dinesh,Pillai,dinesh.pillai@gmail.com,9012345679,Trivandrum,NovaMart,Web
```

---

### 2.2 File 2 — Loyalty Members (`loyalty_members.csv`)

**Source:** Loyalty Program — ingested as a DLO (Data Lake Object) via CSV upload.

> *Important: Some email addresses here are slightly different from the CRM file (e.g., `p.nair@yahoo.com` vs `priya.nair@yahoo.com`, `ravikumar@outlook.com` vs `ravi.kumar@outlook.com`). This is intentional. Participants must configure Identity Resolution to match these.*

| LoyaltyId | FirstName | EmailAddress | Mobile | TierLevel | Points | Note |
|---|---|---|---|---|---|---|
| L001 | Arjun | arjun.sharma@gmail.com | 9876543210 | Gold | 4500 | ✅ Exact match with CRM |
| L002 | Priya | p.nair@yahoo.com | 9845001122 | Silver | 1200 | ⚠️ Email mismatch — phone matches |
| L003 | Ravi | ravikumar@outlook.com | 9901234567 | Bronze | 300 | ⚠️ Email mismatch — phone matches |
| L007 | Vikram | vikram.singh@gmail.com | 9234567890 | Platinum | 9200 | ✅ Exact match |
| L012 | Lakshmi | lakshmi.reddy@gmail.com | 9789012345 | Platinum | 8700 | ✅ Exact match |

**Full CSV — save as `loyalty_members.csv`:**

```csv
LoyaltyId,FirstName,LastName,EmailAddress,MobileNumber,TierLevel,LoyaltyPoints,JoinDate,City
L001,Arjun,Sharma,arjun.sharma@gmail.com,9876543210,Gold,4500,2021-03-15,Mumbai
L002,Priya,Nair,p.nair@yahoo.com,9845001122,Silver,1200,2022-07-20,Bangalore
L003,Ravi,Kumar,ravikumar@outlook.com,9901234567,Bronze,300,2023-01-10,Chennai
L004,Sneha,Patel,sneha.patel@gmail.com,9712345678,Gold,6800,2020-11-05,Ahmedabad
L005,Karthik,Rao,karthik.rao@gmail.com,9988776655,Silver,2100,2022-04-18,Hyderabad
L006,Deepa,Menon,deepa.menon@hotmail.com,9123456789,Bronze,450,2023-06-22,Kochi
L007,Vikram,Singh,vikram.singh@gmail.com,9234567890,Platinum,9200,2019-08-30,Delhi
L008,Ananya,Das,ananya.das@gmail.com,9345678901,Silver,1850,2022-09-14,Kolkata
L009,Rahul,Verma,rahul.verma@yahoo.com,9456789012,Bronze,220,2023-03-01,Pune
L010,Meena,Iyer,meena.iyer@gmail.com,9567890123,Gold,5100,2021-01-25,Chennai
L011,Suresh,Joshi,suresh.joshi@outlook.com,9678901234,Silver,1600,2022-12-11,Jaipur
L012,Lakshmi,Reddy,lakshmi.reddy@gmail.com,9789012345,Platinum,8700,2020-05-17,Hyderabad
L013,Arun,Nambiar,arun.nambiar@gmail.com,9890123456,Gold,4200,2021-07-09,Kochi
L014,Pooja,Shah,pooja.shah@yahoo.com,9901234568,Bronze,150,2023-08-30,Mumbai
L015,Dinesh,Pillai,dinesh.pillai@gmail.com,9012345679,Silver,2300,2022-02-14,Trivandrum
```

---

### 2.3 File 3 — Purchase History (`purchase_history.csv`)

**Source:** ERP transaction export — ingested as a DLO with Transformation.

> *Key transformation required: `OrderDate` is in YYYY-MM-DD format — map to Date type. `OrderAmount` must be cast to Number. A derived field `SpendCategory` must be created: if OrderAmount > 20000 → 'High', if > 5000 → 'Medium', else → 'Low'.*

**Full CSV — save as `purchase_history.csv`:**

```csv
TransactionId,CustomerEmail,CustomerPhone,ProductName,Category,OrderDate,OrderAmount,OrderStatus,ReturnFlag
T001,arjun.sharma@gmail.com,9876543210,Samsung Galaxy S24,Electronics,2024-01-10,74999,Delivered,N
T002,p.nair@yahoo.com,9845001122,Nike Running Shoes,Footwear,2024-02-05,5999,Delivered,N
T003,ravikumar@outlook.com,9901234567,Instant Pot 6L,Kitchen,2024-03-18,7499,Delivered,Y
T004,sneha.patel@gmail.com,9712345678,Apple AirPods Pro,Electronics,2023-08-22,22999,Delivered,N
T005,karthik.rao@gmail.com,9988776655,Levi Jeans 511,Apparel,2024-01-30,3499,Delivered,N
T006,deepa.menon@hotmail.com,9123456789,Philips Air Fryer,Kitchen,2023-11-14,8499,Delivered,N
T007,vikram.singh@gmail.com,9234567890,Sony WH-1000XM5,Electronics,2024-04-02,29999,Delivered,N
T008,ananya.das@gmail.com,9345678901,Adidas Ultraboost,Footwear,2023-12-25,12999,Delivered,N
T009,rahul.verma@yahoo.com,9456789012,Puma T-Shirt Pack,Apparel,2024-02-14,1899,Delivered,N
T010,meena.iyer@gmail.com,9567890123,OnePlus Watch 2,Electronics,2023-10-08,17999,Delivered,N
T011,suresh.joshi@outlook.com,9678901234,Prestige Cooker,Kitchen,2024-03-05,2299,Delivered,N
T012,lakshmi.reddy@gmail.com,9789012345,iPhone 15 Pro,Electronics,2023-09-28,134999,Delivered,N
T013,arun.nambiar@gmail.com,9890123456,Woodland Boots,Footwear,2024-01-20,4499,Delivered,N
T014,pooja.shah@yahoo.com,9901234568,Zara Kurti Set,Apparel,2024-04-10,2999,Cancelled,N
T015,dinesh.pillai@gmail.com,9012345679,boAt Rockerz 450,Electronics,2023-11-11,1999,Delivered,N
T016,arjun.sharma@gmail.com,9876543210,OnePlus 12,Electronics,2023-11-01,59999,Delivered,N
T017,vikram.singh@gmail.com,9234567890,Dyson V15 Vacuum,Appliances,2024-02-20,52999,Delivered,N
T018,meena.iyer@gmail.com,9567890123,Saree Silk Premium,Apparel,2024-03-15,8999,Delivered,N
T019,lakshmi.reddy@gmail.com,9789012345,iPad Pro M4,Electronics,2024-04-01,109999,Delivered,N
T020,sneha.patel@gmail.com,9712345678,KitchenAid Mixer,Kitchen,2023-07-07,45999,Delivered,N
```

---

## 3. Level 1 — Beginner

> 🟢 **BEGINNER LEVEL — Ingest, Map & Unify**

**Objective:** Connect all three data sources to Data Cloud, map them to the correct Data Model Objects, run Identity Resolution, and explore the Unified Profile.

---

### 3.1 Step-by-Step: Create the CRM Data Stream (DSO)

A Data Stream from Salesforce CRM is called a DSO (Data Source Object). It connects Data Cloud directly to a CRM object without needing a file upload.

1. In Data Cloud, go to **Data Streams** and click **New**.
2. Select **Salesforce CRM** as the connector type.
3. Choose the **Contact** object.
4. Select the following fields: `Id`, `FirstName`, `LastName`, `Email`, `Phone`, `MailingCity`, `LeadSource`.
5. Set the refresh schedule to **Hourly**.
6. Save and deploy the Data Stream. Wait for the first run to complete.

> 💡 **Tip:** After deployment, click **View Data** to confirm rows are flowing in. If no rows appear after 10 minutes, check that the Contact object has at least one record in your CRM org.

---

### 3.2 Step-by-Step: Create Loyalty DLO (CSV Upload)

A DLO (Data Lake Object) is created when you upload a CSV file. This is the simplest ingestion method.

1. In Data Cloud, go to **Data Streams** and click **New**.
2. Select **File Upload** as the source type.
3. Upload the `loyalty_members.csv` file.
4. Review the auto-detected field types. Data Cloud will detect most fields as Text.
5. Change the `LoyaltyPoints` field type to **Number**.
6. Change the `JoinDate` field type to **Date** (format: YYYY-MM-DD).
7. Name this Data Stream: `Loyalty_Members_DS` and deploy.

> 💡 **Tip:** The email field in this file is named `EmailAddress` — not `Email` like in CRM. This is intentional. Do not rename it yet. This difference will matter during DMO mapping.

---

### 3.3 Step-by-Step: Create Purchase History DLO with Transformation

This is the most important ingestion step. You will not only upload the data but also transform it.

1. In Data Cloud, go to **Data Streams** and click **New**.
2. Select **File Upload**. Upload `purchase_history.csv`.
3. Change `OrderDate` to type **Date**, `OrderAmount` to type **Number**.
4. Click **Add Formula Field** to create a derived field.
5. Name the new field: `SpendCategory` (type: Text).
6. Enter this formula:

```
IF(OrderAmount > 20000, "High", IF(OrderAmount > 5000, "Medium", "Low"))
```

7. Deploy the Data Stream. Name it: `Purchase_History_DS`.

> 💡 **Tip:** Formula fields are calculated at ingestion time — Data Cloud adds them as it processes each row. Ask participants: why is this useful compared to doing it in SQL later?

---

### 3.4 Step-by-Step: Map Data Streams to DMOs

A DMO (Data Model Object) is the standardized schema in Data Cloud. All incoming data must be mapped to DMOs so Data Cloud can understand relationships between records.

| Data Stream | Map To DMO | Key Field Mappings | Primary Key |
|---|---|---|---|
| CRM Contacts DS | Individual (standard) | Email → ContactPointEmail, FirstName → FirstName, Id → SourceRecordId | Id (Contact ID) |
| Loyalty Members DS | Individual (standard) | EmailAddress → ContactPointEmail, FirstName → FirstName, LoyaltyId → SourceRecordId | LoyaltyId |
| Purchase History DS | Sales Order (standard) | TransactionId → Id, CustomerEmail → ContactPointEmail, OrderAmount → TotalAmount, OrderDate → OrderedDate | TransactionId |

> 💡 **Tip:** Both CRM Contacts and Loyalty Members map to the **same Individual DMO**. This is intentional. Data Cloud will use Identity Resolution to merge them into a Unified Individual later. Do not worry about duplicates at this stage.

---

### 3.5 Step-by-Step: Configure Identity Resolution

Identity Resolution (IR) finds records from different sources that belong to the same real-world person and merges them into a Unified Individual.

1. In Data Cloud, go to **Identity Resolution** and click **New Ruleset**.
2. Name the ruleset: `NovaMart_Customer_IR`.
3. Add **Match Rule 1: Exact Email Match**
   - Field: `ContactPointEmail` | Match Type: Exact | Normalization: Lowercase
4. Add **Match Rule 2: Fuzzy Name + Exact Phone**
   - Field 1: `FirstName` | Match Type: Fuzzy | Field 2: `Phone` | Match Type: Exact
5. Set **Reconciliation Rule**: Most Recent Source Wins for all conflicting fields.
6. Save and Run the ruleset.
7. Go to **Unified Individual** and search for 'Arjun Sharma'. Verify that his CRM and Loyalty records are merged.

> **Expected Outcome — Identity Resolution**
> - Arjun Sharma (C001 + L001) — merged via exact email match.
> - Priya Nair (C002 + L002) — merged via phone match (email was different: `priya.nair` vs `p.nair`).
> - Ravi Kumar (C003 + L003) — merged via phone match (email was different: `ravi.kumar` vs `ravikumar`).
> - Records without a loyalty counterpart remain as individual profiles from CRM only.
> - The Unified Individual count should be **15** (not 30), confirming deduplication worked.

---

## 4. Level 2 — Intermediate

> 🟡 **INTERMEDIATE LEVEL — Insights, Segments, Activation & Data Actions**

**Objective:** Build a Calculated Insight to score customers by recency and spend, create a dynamic segment of high-value lapsed customers, activate it to Salesforce CRM as a campaign, and trigger an automated follow-up using Data Actions.

> *Pre-requisite: Beginner level must be complete. Unified Individuals and Sales Orders must be available.*

---

### 4.1 Step-by-Step: Create the Calculated Insight (RFM Score)

A Calculated Insight (CI) is a SQL-based metric computed across your unified Data Cloud data. Think of it as a persistent, reusable query result that can be used in segments, prompt templates, and Agentforce.

1. In Data Cloud, go to **Calculated Insights** and click **New**.
2. Name it: `Customer_RFM_Score`.
3. Enter the following SQL:

```sql
SELECT
    ui.Id__c                              AS uid__c,
    COUNT(so.Id__c)                       AS purchase_count__c,
    SUM(so.TotalAmount__c)                AS total_spend__c,
    MAX(so.OrderedDate__c)                AS last_purchase_date__c,
    DATEDIFF(TODAY(), MAX(so.OrderedDate__c)) AS days_since_purchase__c,
    CASE
        WHEN DATEDIFF(TODAY(), MAX(so.OrderedDate__c)) > 90
             AND SUM(so.TotalAmount__c) > 10000
        THEN 'High_Value_Lapsed'
        WHEN DATEDIFF(TODAY(), MAX(so.OrderedDate__c)) <= 30
        THEN 'Active'
        ELSE 'Standard'
    END                                   AS customer_status__c
FROM UnifiedIndividual__dlm ui
JOIN SalesOrder__dlm so
    ON ui.Id__c = so.ContactPointEmailId__c
GROUP BY ui.Id__c
```

4. Save and Run the CI. Verify the output by clicking **Preview Data**.

> 💡 **Tip:** CI output fields always use the `__c` suffix. Standard DMO fields use the `__dlm` suffix. This is a common point of confusion — remind participants of this difference.

---

### 4.2 Step-by-Step: Build the Segment

1. In Data Cloud, go to **Segments** and click **New**.
2. Name: `High_Value_Lapsed_Customers`.
3. Select **Individual** as the Segment On object.
4. Add Filter: Calculated Insight → `Customer_RFM_Score` → `customer_status__c` → Equals → `High_Value_Lapsed`.
5. Set Publish Schedule: **Every 12 hours**.
6. Save and Publish the segment. Note the member count.

> **Expected Segment Members (based on sample data)**
> - Arjun Sharma — Total Spend: ₹1,34,998 | Last Purchase: Jan 2024 → Likely qualifies
> - Vikram Singh — Total Spend: ₹82,998 | Last Purchase: Feb 2024 → Likely qualifies
> - Lakshmi Reddy — Total Spend: ₹2,44,999 | Last Purchase: Apr 2024 → Check date diff
>
> Note: `TODAY()` is dynamic. Adjust dates in your CSV to ensure at least 3–5 members qualify.

---

### 4.3 Step-by-Step: Activate to Salesforce CRM

1. In Data Cloud, go to **Activations** and click **New**.
2. Select **Salesforce CRM** as the activation target.
3. Choose the segment: `High_Value_Lapsed_Customers`.
4. Map to CRM Object: **Campaign Member** (create a Campaign in CRM first named `NovaMart Re-Engagement Q2`).
5. Map fields: Unified Individual Email → Lead/Contact Email, Name → Name.
6. Set Contact Point: **Email**.
7. Save and run the activation. Check the Campaign in CRM to verify members are added.

---

### 4.4 Step-by-Step: Configure a Data Action

A Data Action triggers an automated response in Salesforce when a record enters or exits a segment.

1. In Salesforce CRM, create a Flow: Trigger = Platform Event (Data Cloud Triggered Flow), Action = Create Task with Subject `Follow up — High Value Lapsed Customer`.
2. In Data Cloud, go to **Data Actions** and click **New**.
3. Name: `Lapsed_Customer_Alert`.
4. Trigger: Segment Membership Change → `High_Value_Lapsed_Customers` → **On Entry**.
5. Action Type: **Salesforce Platform Event**.
6. Map payload fields: Individual Id, Email, `customer_status__c`.
7. Save and Activate. Test by manually adding a contact to the segment.

> 💡 **Tip:** Data Actions fire in near-real-time when segment membership changes. They are different from Activations — Activations sync segment lists in batch. Data Actions react to individual membership events immediately.

---

## 5. Level 3 — Advanced

> 🔴 **ADVANCED LEVEL — Search Index, Retriever, Prompt Builder & Agentforce**

**Objective:** Build an AI-powered Agentforce Service Agent that answers NovaMart customer questions (return policy, product info, warranty) by reading structured knowledge grounded in Data Cloud.

> *Pre-requisite: Beginner level complete. Agentforce and Einstein Studio must be enabled in the org.*

---

### 5.1 Prepare Knowledge Content

Participants can use any of these options:

- **Option A:** Upload a PDF or Word document (NovaMart Return Policy — content below).
- **Option B:** Use Salesforce Knowledge Articles (create 3–5 articles in Service Cloud).
- **Option C:** Both — upload a file AND use Knowledge Articles.

**Sample knowledge content — copy into a `.txt` or Word file and upload:**

```
NovaMart Customer Policy Guide — Version 2025

RETURN POLICY
Electronics: 7-day return window from delivery date. Item must be unused and in original packaging.
Footwear and Apparel: 15-day return window. Items must have original tags attached.
Kitchen Appliances: 10-day return window. Must include all accessories and user manual.
Non-returnable: Perishables, digital downloads, customized items.

WARRANTY POLICY
All electronics purchased from NovaMart carry a 1-year manufacturer warranty.
Extended warranty (2 additional years) available at 8% of product price at time of purchase.
Warranty claims: Visit any NovaMart service center or raise a ticket at support.novamart.com.

DELIVERY POLICY
Standard delivery: 3-5 business days. Express delivery: 1-2 business days (+₹99).
Free delivery on orders above ₹999. Cash on delivery available for orders below ₹10,000.

LOYALTY PROGRAM
Bronze: 0-999 points. Silver: 1000-4999. Gold: 5000-8999. Platinum: 9000+.
Points earned: 1 point per ₹10 spent. Points expire after 12 months of inactivity.
Platinum members get free express delivery on all orders.
```

---

### 5.2 Step-by-Step: Create the Search Index

A Search Index processes your uploaded content — it chunks the text and vectorizes each chunk so that semantic search can find the most relevant piece for any given question.

1. In Data Cloud, go to **Search Indexes** and click **New**.
2. Name: `NovaMart_Policy_Index`.
3. Select the data source: the file uploaded in Step 5.1 (or Knowledge Article DMO).
4. Chunking Strategy: chunk size **300 tokens**, overlap **50 tokens**.
5. Search Type: **Hybrid** (combines vector search + keyword search).
6. Save and Build the index. Wait for status to show **Active**.

> 💡 **Tip:** Hybrid search is almost always better than pure vector for knowledge base Q&A. Pure vector can miss exact keyword matches. Always recommend Hybrid to clients unless they have a specific reason for pure vector.

---

### 5.3 Step-by-Step: Create the Retriever

1. In **Einstein Studio**, go to **Retrievers** and click **New Retriever**.
2. Select **Individual Retriever** (not Team Retriever).
3. Select the Data Space: **default**.
4. Select the DMO linked to your Search Index.
5. Select the Search Index: `NovaMart_Policy_Index`.
6. In field selection, select **only the `Chunk__c` field** (this is where the actual text lives).
7. Name: `NovaMart_Policy_Retriever`. Save.

> 💡 **Tip:** Why select only `Chunk__c`? The default retriever returns many metadata fields that add noise to the LLM prompt. Fewer tokens in the prompt = better, faster, cheaper LLM response.

---

### 5.4 Step-by-Step: Build the Prompt Template

A Prompt Template in Prompt Builder defines how the retrieved content is combined with the user's question before sending it to the LLM.

1. In Setup, search for **Prompt Builder** and open it.
2. Click **New Prompt Template**. Select type: **Field Generation**.
3. Name: `NovaMart_Policy_Answer`.
4. Paste the following template:

```
You are a helpful customer service assistant for NovaMart, a retail brand.
Answer the customer's question using ONLY the information in the context below.
If the answer is not in the context, say: 'I don't have that information right now.
Please contact our support team at support.novamart.com.'

Do not make up information. Be concise and friendly.

CONTEXT:
{!$Input:Retriever.NovaMart_Policy_Retriever.Chunk__c}

CUSTOMER QUESTION:
{!$Input:question}

ANSWER:
```

5. Save the template. Test using the **Preview** button — enter `What is the return policy for electronics?` and verify the response is grounded in the context.

---

### 5.5 Step-by-Step: Deploy the Agentforce Service Agent

1. In Setup, search for **Agentforce** and open **Agentforce Builder**.
2. Create a new agent from the **Service Agent** template.
3. Name: `NovaMart Support Agent`.
4. Add a Topic: `Customer Policy Questions`.
   - Description: *Handles questions about NovaMart return policy, warranty, delivery, and loyalty program.*
5. Add an Action under this topic: `Answer Policy Questions`.
   - Action Type: Prompt Template → select `NovaMart_Policy_Answer`.
6. Activate the agent and open the **Agent Preview** panel.
7. Test with these questions:
   - *Can I return my phone after 10 days?*
   - *What are the Platinum loyalty benefits?*
   - *Do you offer cash on delivery?*

> **Expected Agent Behaviour**
>
> **Q: Can I return my phone after 10 days?**
> Expected: Agent responds based on the 7-day electronics return policy. It should NOT say 10 days is fine — it should say the window has passed.
>
> **Q: What are the Platinum loyalty benefits?**
> Expected: Agent mentions free express delivery for Platinum members.
>
> If the agent hallucinates, the retriever is not grounding the prompt correctly. Check that the `Chunk__c` field is being returned by the retriever.

---

## 6. Common Mistakes & How to Help

| Mistake | Where It Happens | How to Help |
|---|---|---|
| Identity Resolution runs but finds no matches | Beginner — IR setup | Check that both data streams are mapped to the Individual DMO and have a `ContactPointEmail` field. IR matches on DMO fields, not raw source fields. |
| CI SQL fails with "field not found" error | Intermediate — CI | DMO fields in CI SQL must use the `__dlm` suffix. Custom CI output fields use `__c`. Ask participant to check DMO field names in Data Explorer. |
| Activation shows 0 contacts pushed to CRM | Intermediate — Activation | Ensure the CRM Campaign exists first. Check that the segment has at least one member. Verify Contact Point type is set to Email. |
| Agent gives generic answers not from the policy doc | Advanced — Agentforce | The retriever may not be linked to the agent action correctly. Open Prompt Builder and use Preview to test the retriever response before testing in the agent. |
| Search Index stays in 'Building' state for too long | Advanced — Search Index | In SDO orgs, indexing can take 10–20 minutes. If it fails, check that the file was uploaded correctly and has readable text content. |

---

## 7. Discussion Questions for the Team Session

### After Beginner

1. Why did we need two separate match rules in Identity Resolution? What would have happened if we only used exact email match?
2. Priya Nair had a different email in Loyalty vs CRM. How did Data Cloud still link them? What does this tell you about how Identity Resolution works?
3. What is the difference between a DLO and a DSO in real project terms? When would you choose one over the other?

### After Intermediate

1. The Calculated Insight uses `TODAY()` — what happens to the segment membership next month without any data change? Will the same customers still qualify?
2. What is the difference between a Data Action and an Activation? Can you think of a real client scenario where you would use both together?

### After Advanced

1. The prompt template says *'use ONLY the information in the context'*. Why is this instruction important? What happens if you remove it?
2. A client asks: should we use Hybrid search or Vector-only search for their product catalog? What factors would you consider?
3. How is the Retriever different from a standard Salesforce query? What problem does it solve that SOQL cannot?

---

## 8. Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA INGESTION LAYER                         │
│  CRM Contacts (DSO)  │  Loyalty Members (CSV→DLO)              │
│  Purchase History (CSV→DLO + Transformation)                   │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DATA MODEL LAYER                             │
│  Individual DMO (CRM + Loyalty)  │  Sales Order DMO (Purchase) │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                   IDENTITY RESOLUTION                           │
│  Exact Email Match + Fuzzy Name + Exact Phone                  │
│  → Unified Individual (Golden Profile per customer)            │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│               INSIGHTS & ACTIVATION LAYER                       │
│  Calculated Insight (RFM Score)                                │
│  → Segment → Activation to CRM Campaign                        │
│  → Data Action → Salesforce Flow → CRM Task                    │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                  AI & AGENTFORCE LAYER                          │
│  Policy Doc / Knowledge Articles                               │
│  → Search Index (Hybrid) → Retriever                           │
│  → Prompt Builder → Agentforce Service Agent                   │
└─────────────────────────────────────────────────────────────────┘
```

---

*Next: UC2 — MediCare360 (Healthcare) | UC3 — DriveSmart Motors (Automotive) | UC4 — CloudPilot Inc (B2B SaaS)*
