# 📊 Logistics Operations Dashboard System
**Power BI · SQL · DAX · AI Automation**

A multi-report Power BI dashboard system built end-to-end for a logistics 
company operating at Huu Nghi border gate (Vietnam–China), covering vehicle 
tracking, yard time analysis, import-export customs data, and business 
performance reporting.

> Built and maintained as the sole Data Analyst — from requirements gathering 
> to final delivery.

---

## 📌 Business Context

The company handles hundreds of cross-border vehicles daily across multiple 
service types (pallet, crane, loading, transshipment). Leadership needed 
real-time visibility into operations but data was scattered across multiple 
systems with no unified reporting layer.

**My role:** Design the full reporting architecture, build all dashboards, 
maintain data pipelines, and translate operational data into actionable 
insights for management.

---

## 📂 Dashboard Overview

### 1. 🚛 Vehicle Yard Traffic Report (`BÁO CÁO LƯỢT XE LƯU BÃI`)
Tracks all vehicle movements in and out of the logistics yard.

**Key metrics:**
- Total vehicle trips: **324,000+** (2023)
- Daily average: **888 trips/day**
- Service breakdown: Pallet 46.34% · Loading 34.44% · Crane 16.23%
- Top cargo type: Electronic components (87K trips)
- Vehicle type analysis: Cont45 dominates at 23.05%

**Dimensions:** By service type · cargo type · vehicle type · 
day of week · time series

---

### 2. ⏱ Yard Dwell Time Report (`BÁO CÁO THỜI GIAN XE LƯU BÃI`)
Deep-dive into time efficiency at each stage of the vehicle journey.

**Key metrics:**
- Average total dwell time: **12.54 hours**
- Time breakdown: Service 5.62% · Waiting for service 46.46% · 
  Waiting to exit 47.92%
- Largest bottleneck identified: waiting stages account for ~94% of dwell time

**Insight generated:** The data revealed that actual service time (0.70h avg) 
is minimal — the bottleneck is in queue management before and after service, 
pointing to a process optimization opportunity.

---

### 3. 🛃 Border Gate Import/Export Report (`BÁO CÁO CỬA KHẨU SỐ`)
Analyzes all import/export registration slips at the border gate.

**Key metrics:**
- Total registrations: **201,000** (2019–2020)
- Daily average: **385 registrations/day**
- Import: 157K (77.8%) · Export: 44K (21.6%)
- Peak month: May (25K registrations)
- Top vehicle class: 18-ton+ trucks & containers (166K trips)
- Heatmap analysis: Traffic density by hour and day of week

**Sub-reports:** Vehicle transport · Cargo goods · 
Company-level XNK drill-down · Fee-paying company tracking · 
Customs declaration comparison table

---

### 4. 💼 Business Performance Report (`BÁO CÁO HIỆU QUẢ KINH DOANH`)
Strategic P&L dashboard for management decision-making.

**Key metrics (sample month):**
- Revenue: **9.4 billion VND**
- Cost: **8.4 billion VND**
- Gross profit margin: **10.41%**
- Avg profit per vehicle: **477,558 VND**
- Revenue structure: Customs 84.69% · Transport 8.76% · 
  Transshipment 6.55%

**Features:**
- Top 5 customers by revenue with profit margin per customer
- 12-month rolling trend (Revenue · COGS · Margin)
- Advance/reimbursement tracking by customer
- Profit/loss by contract

---

## 🛠 Technical Stack

| Layer | Tools |
|---|---|
| Visualization | Power BI Desktop |
| Data transformation | Power Query (M language) |
| Business logic | DAX measures & calculated columns |
| Data sources | Internal management systems · Excel files |
| Automation | AI tools (Claude, NotebookLM) for insight summarization |

---

## 🧠 DAX Highlights

Key measures built for this project:

```dax
-- Gross Profit Margin %
Gross Margin % = 
DIVIDE(
    [Total Revenue] - [Total Cost],
    [Total Revenue],
    0
)

-- Average dwell time per vehicle
Avg Dwell Time = 
AVERAGEX(
    FactVehicle,
    FactVehicle[ExitTime] - FactVehicle[EntryTime]
)

-- Rolling 12-month revenue trend
Revenue LTM = 
CALCULATE(
    [Total Revenue],
    DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -12, MONTH)
)
```

---

## 📁 Repository Structure
