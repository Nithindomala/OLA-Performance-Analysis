
# 🚕 Ola Ride Performance Analysis — Business Intelligence Platform

> **Tools:** SQL (MySQL) · Power BI (DAX, Calculated Measures) · Excel
>
> **Dataset:** 20,407 booking records | July 2024

### 🔗 Quick Links

- 📁 [Dataset](https://github.com/Nithindomala/OLA-Performance-Analysis/blob/main/Ola_Bookings.csv)
- 📊 [Power BI Dashboard](https://github.com/Nithindomala/OLA-Performance-Analysis/blob/main/ola%20bookings%20project.pbix)
- 🗄️ [SQL Queries](https://www.dropbox.com/scl/fi/rtn7fb3p0wbw79wwu8o1a/sql-queries.sql?rlkey=iylyhyxmv1frruqh52nofahzy&st=upz5awkb&dl=0)

---

## 📌 Business Problem

Ola's operations team had no structured visibility into three questions that directly affected profitability:

1. **Why** were 17.91% of rides being cancelled — and which party (driver or customer) was responsible?
2. **Which** vehicle types and payment channels were generating the most revenue?
3. **Which** customers represented the highest lifetime booking value?

Without these answers, fleet allocation, driver incentive programs, and payment strategy decisions were being made reactively. This project transforms raw booking logs into a structured, stakeholder-ready BI platform to answer each of these questions with data.

---

## 🔍 Data Validation & Quality Assurance

Before any analysis, the raw dataset was systematically validated to ensure data accuracy and integrity — a non-negotiable standard for reporting that feeds real business decisions.

| Issue Identified | Validation Method | Resolution |
|---|---|---|
| Null `Booking_Value` on incomplete rides | Filtered via `WHERE Booking_Status = 'Success'` | Prevented revenue inflation in KPI reporting |
| Inconsistent `Booking_Status` labels | `GROUP BY` audit across all status categories | Confirmed 5 distinct statuses; standardised for dashboard filters |
| Cancellation reasons with mixed attribution | Separate SQL views for driver vs. customer cancellations | Enabled root-cause analysis by responsible party |
| Potential duplicate `Customer_ID` entries | Cross-referenced ID with ride counts | Confirmed uniqueness; no duplicates found |

> This validation process mirrors the data governance and control documentation standards applied in audit and compliance environments (ITGC / SOX adjacent).

---

## ⚙️ Analytical Approach

### SQL — Business Questions → Structured Views

Each business question was translated into a reusable SQL view, making the analysis auditable, repeatable, and stakeholder-shareable.

```sql
-- Business Question: Which customers drive the most booking value?
-- Stakeholder use: Customer loyalty program targeting

CREATE VIEW Top_5_Customers AS
SELECT
    Customer_ID,
    COUNT(Booking_ID)                                AS Total_Rides,
    SUM(Booking_Value)                               AS Total_Spend,
    ROUND(SUM(Booking_Value) / COUNT(Booking_ID), 2) AS Avg_Per_Ride
FROM bookings
WHERE Booking_Status = 'Success'
GROUP BY Customer_ID
ORDER BY Total_Spend DESC
LIMIT 5;

-- Business Question: Why are drivers cancelling rides?
-- Stakeholder use: Driver policy and fleet maintenance decisions

CREATE VIEW Cancellation_By_Driver_Reason AS
SELECT
    Canceled_Rides_By_Driver AS Cancellation_Reason,
    COUNT(*)                 AS Total_Cancellations,
    ROUND(COUNT(*) * 100.0 / (
        SELECT COUNT(*) FROM bookings
        WHERE Booking_Status = 'Cancelled by Driver'
    ), 2)                    AS Pct_Of_Driver_Cancellations
FROM bookings
WHERE Booking_Status = 'Cancelled by Driver'
GROUP BY Cancellation_Reason
ORDER BY Total_Cancellations DESC;

-- Business Question: What is the average ride distance per vehicle type?
-- Stakeholder use: Fare pricing and vehicle deployment strategy

CREATE VIEW Ride_Distance_Per_Vehicle AS
SELECT
    Vehicle_Type,
    ROUND(AVG(Ride_Distance), 2) AS Avg_Distance_KM,
    COUNT(*)                     AS Total_Rides
FROM bookings
WHERE Booking_Status = 'Success'
GROUP BY Vehicle_Type
ORDER BY Avg_Distance_KM DESC;
```

### Power BI — DAX Measures for Self-Service BI

The dashboard was designed so operations managers can filter by vehicle type, date range, and payment method without needing analyst support.

```
Booking Success Rate =
    DIVIDE(
        COUNTROWS(FILTER(Bookings, Bookings[Booking_Status] = "Success")),
        COUNTROWS(Bookings),
        0
    )

Cancellation Rate =
    DIVIDE(
        COUNTROWS(FILTER(Bookings, Bookings[Booking_Status] <> "Success")),
        COUNTROWS(Bookings),
        0
    )

Avg Revenue Per Successful Ride =
    DIVIDE(
        CALCULATE(SUM(Bookings[Booking_Value]), Bookings[Booking_Status] = "Success"),
        CALCULATE(COUNTROWS(Bookings), Bookings[Booking_Status] = "Success"),
        0
    )
```

---

## 📊 Key KPIs & Findings

| KPI | Value | Business Signal |
|---|---|---|
| Total Bookings | 20,407 | Baseline volume for trend benchmarking |
| Total Booking Value | ₹7M | Revenue baseline for fleet ROI calculation |
| Booking Success Rate | **62%** | ⚠️ Below industry benchmark — requires investigation |
| Overall Cancellation Rate | **17.91%** | Split: drivers (34.56% personal/car issues) vs customers (29.31% change of plans) |
| Top Revenue Channel | **Cash (₹4M)** | UPI adoption gap signals a digital payment opportunity |
| Top Customer (CID836942) | **₹3,800 total spend** | High-value loyalty program candidate |

---

## 📈 Strategic Recommendations

Three data-backed recommendations for operations leadership:

**1. Reduce driver cancellations through a fleet maintenance protocol**

34.56% of driver cancellations cite "Personal & Car related issues." A mandatory pre-shift vehicle checklist, enforced via the driver app, could reduce these cancellations by an estimated 15–20% and directly improve the 62% success rate.

**2. Launch a UPI adoption campaign in high-cash markets**

Cash accounts for ₹4M of ₹7M revenue (57%) despite UPI being the lower-friction option. A targeted UPI cashback incentive in high-cash regions would reduce payment processing costs and improve booking completion rates.

**3. Introduce a pre-cancellation intervention for customers**

"Change of plans" (29.31%) is the top customer cancellation reason — often triggered by long perceived wait times. A real-time ETA notification with a small wait incentive (e.g., ₹10 discount on next ride) could recapture an estimated 8–10% of these cancellations.

---

## 🗂️ Repository Structure

```
OLA-Performance-Analysis/
├── README.md                  ← Project documentation (this file)
├── Ola_Bookings.csv           ← Source dataset (20,407 records)
├── ola bookings project.pbix  ← Power BI dashboard file
├── Dashboard images.png       ← Dashboard preview screenshot
└── sql-queries.sql            ← All SQL views (via Dropbox link above)
```

---

## 🛠️ Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| Data Cleaning | Excel | Initial formatting, status standardisation |
| Analysis & KPIs | SQL (MySQL) — 10 views | Business question → structured query → reusable view |
| Visualisation | Power BI (DAX) | Interactive self-service BI dashboard |
| Documentation | This README | Stakeholder-readable project narrative |

---

*Project by [Nithin Domala](https://www.linkedin.com/in/nithin-domala) · [Portfolio](https://nithindomala.netlify.app/)*
