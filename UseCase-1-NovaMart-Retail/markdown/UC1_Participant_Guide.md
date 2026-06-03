# UC1 — Participant Handout | NovaMart Retail
## Data Cloud & Agentforce Training

| Field | Details |
|---|---|
| **Domain** | Retail / E-Commerce |
| **Company** | NovaMart (fictional) |
| **Levels** | Beginner → Intermediate → Advanced |
| **Org Required** | Dev Org or SDO with Data Cloud enabled |
| **Estimated Time** | 3–5 hours total |

---

> 📌 **How to use this document**
>
> This handout has two parts:
> - **Part A (Pages 1–2) — Challenge Card:** Read this first. Attempt the use case on your own.
> - **Part B — Detailed Handout:** Only open this if you are stuck. It has CSV data, task steps, and hints.
>
> If you get stuck, discuss with your team first before opening Part B. The goal is to learn, not just finish.

---

# PART A — CHALLENGE CARD
### *Read this first. Attempt before looking at Part B.*

---

## The Company: NovaMart

NovaMart is a D2C retail brand selling electronics, footwear, apparel, and kitchen products across 12 cities in India. They operate through a website, mobile app, and physical stores. The company has grown fast — but their customer data has not kept up. It lives in three separate systems, and no one can see the full picture of a single customer.

---

## The Problem

NovaMart has the same customer — let's say **Arjun Sharma** — as three different records across three systems with no shared unique ID:

| System | What It Contains | The Problem |
|---|---|---|
| **Salesforce CRM** | Leads and Contacts — managed by the sales team | Has name and email but no purchase history or loyalty info |
| **Loyalty Program (CSV)** | Loyalty members — exported monthly as a CSV | Some emails are slightly different from CRM for the same person |
| **Purchase History (CSV)** | All transactions from the ERP system | Dates in a different format. No standard customer ID — only email or phone |

> *The marketing team cannot find high-value customers who have gone quiet. The sales team cannot see purchase history when they call. Leadership cannot report on true customer lifetime value. NovaMart needs a single, golden record per customer — and then the ability to act on it.*

---

## Your Challenge — Three Levels

Complete them in order. Each level builds on the previous one.

| Level | Name | What You Need to Build | You Know You're Done When... |
|---|---|---|---|
| 🟢 **Beginner** | Ingest, Map & Unify | Connect Salesforce CRM, Loyalty CSV, and Purchase History CSV to Data Cloud. Map them to the right Data Model Objects. Run Identity Resolution to merge duplicate records. | You can search for 'Arjun Sharma' in Unified Individual and see one single record with data from all three sources. |
| 🟡 **Intermediate** | Insights & Activation | Write a Calculated Insight that scores customers by spend and inactivity. Build a segment of high-value lapsed customers. Push that segment to a Salesforce CRM campaign. Set up a Data Action to auto-create a task when someone enters the segment. | A CRM Campaign named 'NovaMart Re-Engagement Q2' has members added automatically. A Task appears in CRM when a customer enters the segment. |
| 🔴 **Advanced** | AI Agent with RAG | Upload NovaMart's return and warranty policy as a knowledge source. Create a Search Index and Retriever. Build a Prompt Template that answers questions using only the policy document. Deploy an Agentforce Service Agent. | The agent correctly answers 'Can I return my phone after 10 days?' using the policy doc — and says it doesn't know when asked something not in the document. |

---

## What Data Do You Have?

Three data sources — figure out how to bring each one into Data Cloud:

- **Salesforce CRM** — 15 Contacts already in your Salesforce org.
- **Loyalty Program** — 15 records as a CSV file.
- **Purchase History** — 20 transactions as a CSV file.

> If you need the CSV files or want a hint on where to start, ask your facilitator.

---

## Tools & Scope

| Tools & Access You Need | What Is In / Out |
|---|---|
| Salesforce Developer Org or SDO with Data Cloud enabled | ✅ IN: CRM, CSV upload, Data Cloud, Agentforce |
| Data Cloud Admin permission set | ✅ IN: Knowledge Articles or uploaded files (Level 3) |
| Agentforce + Einstein Studio enabled (Level 3 only) | ❌ OUT: Marketing Cloud, S3, external APIs |

---

---

# PART B — DETAILED HANDOUT
### *Open this only if you are stuck.*

---

## 1. Your Sample Data

Three CSV files are provided below. Copy each one, save as a `.csv` file, and use them in your exercises.

> ⚠️ **Look carefully at the email addresses across the three files. Some are slightly different for the same person. This is intentional.**

---

### File 1 — CRM Contacts → save as `crm_contacts.csv`

> This represents Contacts from Salesforce CRM. You will connect this using a CRM Data Stream (DSO) — not by uploading the file.

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

### File 2 — Loyalty Members → save as `loyalty_members.csv`

> Notice: The email field is named `EmailAddress` here — not `Email` like in the CRM file. Also, some emails are different from the CRM for the same person (rows L002, L003).

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

### File 3 — Purchase History → save as `purchase_history.csv`

> Notice: `OrderDate` is in YYYY-MM-DD format. `OrderAmount` is a plain number. Some customers have multiple transactions.

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

## 2. Level 1 — Beginner

> 🟢 **Goal:** Get all three data sources into Data Cloud, map them to DMOs, run Identity Resolution, and verify Arjun Sharma's three records are merged into one unified profile.

### Task A — Connect Salesforce CRM as a Data Stream (DSO)
- In Data Cloud, create a new Data Stream connected to **Salesforce CRM**.
- Select the **Contact** object. Include at minimum: `Id`, `FirstName`, `LastName`, `Email`, `Phone`.
- Deploy and confirm data is flowing in.

### Task B — Ingest Loyalty Members as a DLO (CSV Upload)
- Create a new Data Stream using **File Upload**.
- Upload `loyalty_members.csv`.
- Before deploying, fix the field types: `LoyaltyPoints` → Number, `JoinDate` → Date.

### Task C — Ingest Purchase History as a DLO with Transformation
- Create a new Data Stream using **File Upload** for `purchase_history.csv`.
- Fix field types: `OrderDate` → Date, `OrderAmount` → Number.
- Add a **formula field** called `SpendCategory`:
  - If OrderAmount > 20,000 → `'High'`
  - If OrderAmount > 5,000 → `'Medium'`
  - Otherwise → `'Low'`

### Task D — Map to Data Model Objects (DMOs)
- Map CRM Contacts → **Individual DMO**
- Map Loyalty Members → **Individual DMO** (same DMO — this is correct)
- Map Purchase History → **Sales Order DMO**
- Make sure the email field from each source is mapped to `ContactPointEmail`.

### Task E — Configure and Run Identity Resolution
- Create a new Identity Resolution ruleset.
- Add two match rules — decide what fields to use for matching.
- *Hint: Think about when two records have the same email vs when emails are different but phone matches.*
- Run the ruleset and verify results in **Unified Individual**.

> 🧠 **Think About It**
> 1. You have 15 rows in Loyalty and 15 in CRM. After Identity Resolution, how many Unified Individuals do you expect? Why?
> 2. Priya Nair has `priya.nair@yahoo.com` in CRM but `p.nair@yahoo.com` in Loyalty. Her phone is the same. What match rule catches this?
> 3. What is the difference between a DLO and a DSO? Why did we use a DSO for CRM instead of exporting it as a CSV?

> ✅ **You have completed Level 1 when:**
> - CRM Contacts data stream is active and showing 15 records.
> - Loyalty Members and Purchase History are uploaded and deployed.
> - Purchase History has a `SpendCategory` field with values High, Medium, or Low.
> - Identity Resolution has run and Arjun Sharma appears as a **single** Unified Individual with both CRM and Loyalty data visible.
> - Priya Nair and Ravi Kumar (email mismatch records) are also merged correctly.

---

## 3. Level 2 — Intermediate

> 🟡 **Goal:** Score every customer based on spending behaviour, find high-value customers who haven't purchased in 90+ days, push that list to a CRM campaign, and trigger an automated alert.

> *Pre-requisite: Level 1 must be complete.*

### Task A — Create a Calculated Insight
- In Data Cloud, create a new **Calculated Insight** named `Customer_RFM_Score`.
- Write SQL that produces one row per Unified Individual with:
  - Total number of purchases
  - Total amount spent
  - Date of most recent purchase
  - Number of days since last purchase
  - A `customer_status` field: `'High_Value_Lapsed'` if total spend > 10,000 AND days since last purchase > 90, `'Active'` if purchased in last 30 days, otherwise `'Standard'`

### Task B — Build a Segment
- Create a new Segment named `High_Value_Lapsed_Customers`.
- Filter on your Calculated Insight — only include customers with `customer_status = 'High_Value_Lapsed'`.
- Set the segment to refresh every **12 hours**.

### Task C — Activate the Segment to Salesforce CRM
- First, create a Campaign in Salesforce CRM named `NovaMart Re-Engagement Q2`.
- Create an Activation that pushes the segment to this CRM Campaign.
- Verify that Campaign Members appear in the CRM Campaign.

### Task D — Set Up a Data Action
- Create a Data Action that fires when a customer **enters** the `High_Value_Lapsed_Customers` segment.
- The action should trigger a Salesforce Flow that creates a Task: `Subject = 'Follow up — High Value Lapsed Customer'`.
- Verify a Task appears in CRM after a segment membership event.

> 🧠 **Think About It**
> 1. Your segment has 5 members today. Two weeks later — no new purchases, no data changes — how many members do you expect? What if some make a purchase this week?
> 2. What is the difference between an Activation and a Data Action? When would you use both together on a real project?
> 3. The CI uses `DATEDIFF(TODAY(), last_purchase_date)`. What is a risk with using `TODAY()` in a CI that runs on a schedule?

> ✅ **You have completed Level 2 when:**
> - Calculated Insight `Customer_RFM_Score` runs successfully with one row per customer.
> - Segment `High_Value_Lapsed_Customers` has at least 1 member.
> - CRM Campaign `NovaMart Re-Engagement Q2` shows Campaign Members added by the activation.
> - A CRM Task is created automatically when the Data Action fires.

---

## 4. Level 3 — Advanced

> 🔴 **Goal:** Build an AI-powered Agentforce Service Agent that answers NovaMart customer questions about return policy, warranty, and loyalty — by reading NovaMart's actual policy documents, not by guessing.

> *Pre-requisite: Level 1 complete. Agentforce must be enabled in your org.*

### Task A — Prepare Your Knowledge Source
Choose one or both:
- **Option A:** Copy the text below into a Word doc or `.txt` file and upload it to Data Cloud.
- **Option B:** Create 3–5 Knowledge Articles in Salesforce Service Cloud using the same content.

```
NovaMart Customer Policy Guide

RETURN POLICY
Electronics: 7-day return window from delivery date. Item must be unused and in original packaging.
Footwear and Apparel: 15-day return window. Items must have original tags attached.
Kitchen Appliances: 10-day return window. Must include all accessories and user manual.
Non-returnable: Perishables, digital downloads, customized items.

WARRANTY POLICY
All electronics from NovaMart carry a 1-year manufacturer warranty.
Extended warranty (2 additional years) available at 8% of product price at time of purchase.
Warranty claims: Visit any NovaMart service center or raise a ticket at support.novamart.com.

DELIVERY POLICY
Standard delivery: 3-5 business days. Express delivery: 1-2 business days (+Rs.99).
Free delivery on orders above Rs.999. Cash on delivery available for orders below Rs.10,000.

LOYALTY PROGRAM
Bronze: 0-999 points. Silver: 1000-4999. Gold: 5000-8999. Platinum: 9000+.
Points: 1 point per Rs.10 spent. Points expire after 12 months of inactivity.
Platinum members get free express delivery on all orders.
```

### Task B — Create a Search Index
- In Data Cloud, create a **Search Index** connected to your knowledge source.
- Choose **Hybrid** search type (not vector-only).
- Wait for the index to reach **Active** status before proceeding.

### Task C — Create a Retriever
- In Einstein Studio, create a new **Retriever** connected to your Search Index.
- Configure it to return only the **text content field** (not all metadata fields).

### Task D — Build a Prompt Template
- In **Prompt Builder** (Setup), create a new **Field Generation** prompt template.
- The template should instruct the AI to answer using **ONLY the retrieved context** — not from general knowledge.
- Include a clear instruction for what to say if the answer is not in the document.
- Test the template using **Preview** before moving to the agent.

### Task E — Deploy the Agentforce Service Agent
- Create a new Agentforce Service Agent named `NovaMart Support Agent`.
- Add a Topic for customer policy questions.
- Add an Action that uses your Prompt Template.
- Activate and test with these three questions:
  - *Can I return my phone after 10 days?*
  - *What are the Platinum loyalty benefits?*
  - *Do I get free delivery on a Rs.500 order?*

> 🧠 **Think About It**
> 1. The prompt template says *'Answer using ONLY the information in the context below.'* Why is this instruction important? What happens if you remove it?
> 2. You chose Hybrid search instead of vector-only. Can you explain the difference in your own words? Which would you recommend to a client for a product catalog, and why?
> 3. A customer asks: *'Can I get a refund on a laptop I bought 12 days ago?'* The policy says 7 days. What should the agent say? Test it and see.

> ✅ **You have completed Level 3 when:**
> - Search Index is **Active**.
> - Retriever is configured and pointing to the correct Search Index.
> - Prompt Template Preview returns a grounded answer based on the policy doc.
> - Agentforce Service Agent correctly answers *'Can I return my phone after 10 days?'* by referencing the 7-day electronics return policy.
> - When asked something not in the document, the agent says it doesn't know — it does not guess.

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
