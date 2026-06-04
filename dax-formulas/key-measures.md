# DAX Measures — Logistics Dashboard System

> **4 report modules** · Power BI Desktop  
> Measures are grouped by report and category. Redundant, test, and duplicate measures have been removed.

---

## 1. Border Gate Report — Cửa Khẩu Số

### Core Counts

```dax
-- Total import/export registrations
Total Records =
    COUNT(manifests[id])

-- Total unique goods entries
Total Goods =
    DISTINCTCOUNT(Good_details[registrationTransportGoods.id])

-- Total unique transport records
TotalRecords_Transport =
    DISTINCTCOUNT(Transport[registrationTransportDetails.registrationId])

-- Total import registrations
Total Import Records =
    COUNTROWS(FILTER(manifests, manifests[type] = 0))

-- Total export registrations
Total Export Records =
    COUNTROWS(FILTER(manifests, manifests[type] = 1))

-- Total weight (tons)
Total Weight =
    SUM(manifests[totalWeight])

-- Average registrations per day
Average Records Per Day =
    AVERAGEX(
        VALUES(manifests[arrivalDate]),
        COUNTROWS(FILTER(manifests, manifests[arrivalDate] = EARLIER(manifests[arrivalDate])))
    )
```

### Growth Rate Analysis

```dax
-- YoY growth rate for goods (2022 → 2023)
Growth Rate Goods 2023 =
    VAR Sales2023 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            YEAR(manifests[arrivalDate]) = 2023
        )
    VAR Sales2022 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            YEAR(manifests[arrivalDate]) = 2022
        )
    RETURN
        IF(Sales2022 > 0, DIVIDE(Sales2023 - Sales2022, Sales2022), BLANK())

-- YoY growth rate for goods (2023 → 2024)
Growth Rate Goods 2024 =
    VAR Sales2024 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            YEAR(manifests[arrivalDate]) = 2024
        )
    VAR Sales2023 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            YEAR(manifests[arrivalDate]) = 2023
        )
    RETURN
        IF(Sales2023 > 0, DIVIDE(Sales2024 - Sales2023, Sales2023), BLANK())

-- 9-month same-period comparison (2023 vs 2024)
Growth Rate Goods 9M 2024 =
    VAR Sales9M2024 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            YEAR(manifests[arrivalDate]) = 2024,
            MONTH(manifests[arrivalDate]) <= 9
        )
    VAR Sales9M2023 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            YEAR(manifests[arrivalDate]) = 2023,
            MONTH(manifests[arrivalDate]) <= 9
        )
    RETURN
        IF(Sales9M2023 > 0, DIVIDE(Sales9M2024 - Sales9M2023, Sales9M2023), BLANK())

-- Electronic components: 9-month growth (2023 vs 2024)
Growth Rate Electronic Components =
    VAR Sales2024 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            Good_details[NhomHangHoa] = "Linh kiện điện tử",
            YEAR(manifests[arrivalDate]) = 2024,
            MONTH(manifests[arrivalDate]) <= 9
        )
    VAR Sales2023 =
        CALCULATE(
            COUNT(Good_details[registrationTransportGoods.registrationTransportId]),
            Good_details[NhomHangHoa] = "linh kiện điện tử",
            YEAR(manifests[arrivalDate]) = 2023,
            MONTH(manifests[arrivalDate]) <= 9
        )
    RETURN
        IF(Sales2023 > 0, DIVIDE(Sales2024 - Sales2023, Sales2023), BLANK())
```

### Cargo Segmentation — Electronic Components

```dax
-- Total records filtered to electronic components
Total Records Electronic Components =
    CALCULATE(
        COUNT(manifests[id]),
        manifests[LoaiHang] = "Linh kiện điện tử"
    )

-- 9-month records: 2023
Total Records 9M 2023 (Electronics) =
    CALCULATE(
        COUNT(manifests[id]),
        manifests[LoaiHang] = "Linh kiện điện tử",
        manifests[arrivalDate] >= DATE(2023, 1, 1),
        manifests[arrivalDate] <= DATE(2023, 9, 30)
    )

-- 9-month records: 2024
Total Records 9M 2024 (Electronics) =
    CALCULATE(
        COUNT(manifests[id]),
        manifests[LoaiHang] = "Linh kiện điện tử",
        manifests[arrivalDate] >= DATE(2024, 1, 1),
        manifests[arrivalDate] <= DATE(2024, 9, 30)
    )
```

### Dynamic Color Formatting (for bar chart highlights)

```dax
-- Highlight max/min bars in Import/Export chart
Import Export Bar Color =
    VAR _max = MAXX(ALL(Elxport[importExportTypeChanges]), [TotalRecords_EIxport])
    VAR _min = MINX(ALL(Elxport[importExportTypeChanges]), [TotalRecords_EIxport])
    RETURN
        SWITCH(
            TRUE(),
            [TotalRecords_EIxport] = _max, "#1B539A",
            [TotalRecords_EIxport] = _min, "#7698C2",
            "#E7EBEF"
        )

-- Highlight max/min bars in vehicle type chart
Vehicle Type Bar Color =
    VAR _max = MAXX(ALL(Transport[registrationTransportDetails.vehicleType.name]), [TotalRecords_Transport])
    VAR _min = MINX(ALL(Transport[registrationTransportDetails.vehicleType.name]), [TotalRecords_Transport])
    RETURN
        SWITCH(
            TRUE(),
            [TotalRecords_Transport] = _max, "#1B539A",
            [TotalRecords_Transport] = _min, "#7698C2",
            "#E7EBEF"
        )

-- Last data reload timestamp
Last Reload =
    "Last Reload: " & MAX('Reload Time'[Reload])
```

---

## 2. Business Performance Report — Hiệu Quả Kinh Doanh

### Revenue

```dax
-- Confirmed revenue (checked = TRUE)
Revenue =
    CALCULATE(
        SUM(order_revenue_category[revenue]),
        FILTER(order_revenue_category, order_revenue_category[checked] = TRUE())
    )

-- Revenue by category: Customs (Hải quan)
Revenue Customs =
    CALCULATE(
        SUM(order_revenue_category[revenue]),
        FILTER(
            order_revenue_category,
            order_revenue_category[checked] = TRUE() &&
            RELATED(order_expense_classifications[classification_id]) = 10
        )
    )

-- Revenue by category: Transshipment (Sang tải)
Revenue Transshipment =
    CALCULATE(
        SUM(order_revenue_category[revenue]),
        FILTER(
            order_revenue_category,
            order_revenue_category[checked] = TRUE() &&
            RELATED(order_expense_classifications[classification_id]) = 30
        )
    )

-- Revenue previous month (for MoM comparison)
Revenue Previous Month =
    CALCULATE([Revenue], PREVIOUSMONTH('Date'[Date]))

-- Revenue same period last year
Revenue Previous Year =
    CALCULATE([Revenue], SAMEPERIODLASTYEAR('Date'[Date]))
```

### Cost & Profit

```dax
-- Cost of goods (confirmed)
Cost =
    CALCULATE(
        SUM(order_expense_category[price]),
        FILTER(order_expense_category, order_expense_category[checked] = TRUE())
    )

-- Gross profit
Gross Profit =
    [Revenue] - [Cost]

-- Gross profit margin %
Gross Margin % =
    IF([Revenue] <> 0, DIVIDE([Gross Profit], [Revenue]), 0)

-- Net profit (after tax)
Net Profit =
    [Gross Profit] - [Tax]

-- Tax (8% of revenue)
Tax =
    [Revenue] * 0.08

-- Profit per vehicle
Profit per Vehicle =
    DIVIDE([Gross Profit], [Total Vehicles by Category])
```

### MoM Variance (Period-over-Period)

```dax
-- Revenue growth % vs previous month
Revenue MoM % =
    IF(
        [Revenue Previous Month] <> 0,
        DIVIDE([Revenue], [Revenue Previous Month]) - 1,
        BLANK()
    )

-- Gross profit growth % vs previous month
Gross Profit MoM % =
    IF(
        CALCULATE([Gross Profit], PREVIOUSMONTH('Date'[Date])) <> 0,
        DIVIDE([Gross Profit], CALCULATE([Gross Profit], PREVIOUSMONTH('Date'[Date]))) - 1,
        BLANK()
    )

-- Cost growth % vs previous month
Cost MoM % =
    IF(
        CALCULATE([Cost], PREVIOUSMONTH('Date'[Date])) <> 0,
        DIVIDE([Cost], CALCULATE([Cost], PREVIOUSMONTH('Date'[Date]))) - 1,
        BLANK()
    )
```

### Cash Flow Tracking

```dax
-- Total advance (tạm ứng)
Total Advance =
    SUM(refund_slip_items[advanced_amount])

-- Total disbursed (approved)
Total Disbursed =
    CALCULATE(
        SUM(disbursement_for_refunds[total]),
        FILTER(disbursement_for_refunds, disbursement_for_refunds[status_id] = 50)
    )

-- End-of-period cash balance
End Period Balance =
    [Total Disbursed] - [Total Reimbursed Spending]

-- Total receivable from customers
Total Customer Receivable =
    [Revenue] + [Collection on Behalf]
```

### Customer Ranking

```dax
-- Rank customers by revenue (top/bottom dynamic)
Customer Revenue Ranking =
    VAR _top = RANKX(ALL(customers[name]), [Revenue], , DESC, Dense)
    VAR _bottom = RANKX(ALL(customers[name]), [Revenue], , ASC, Dense)
    VAR _rank =
        IF(SELECTEDVALUE(TopBottom[Value]) = "Top", _top, _bottom)
    RETURN
        IF(_rank <= 'Ranking Option'[Ranking Option Value], [Revenue])
```

---

## 3. Yard Time Report — Thời Gian Xe Lưu Bãi

### Core Dwell Time Metrics

```dax
-- Average total dwell time (hours)
Avg Dwell Time (Hours) =
    AVERAGEX(
        'vehicle_activity_job',
        DATEDIFF('vehicle_activity_job'[around_time_in], 'vehicle_activity_job'[time_out], HOUR)
    )

-- Average time waiting for service (hours)
Avg Wait for Service (Hours) =
    AVERAGEX(
        'vehicle_activity_job',
        DATEDIFF('vehicle_activity_job'[time_in], 'vehicle_activity_job'[around_start_at], HOUR)
    )

-- Average actual service time (hours)
Avg Service Time (Hours) =
    AVERAGEX(
        'vehicle_activity_job',
        DATEDIFF('vehicle_activity_job'[around_start_at], 'vehicle_activity_job'[around_finish_at], HOUR)
    )

-- Average time waiting to exit yard (hours)
Avg Wait to Exit (Hours) =
    AVERAGEX(
        'vehicle_activity_job',
        DATEDIFF('vehicle_activity_job'[around_finish_at], 'vehicle_activity_job'[time_out], HOUR)
    )
```

### Time Ratio Breakdown

```dax
-- % of dwell time spent on actual service
Service Time Ratio =
    [Avg Service Time (Hours)] / [Avg Dwell Time (Hours)]

-- % of dwell time spent waiting for service
Wait for Service Ratio =
    [Avg Wait for Service (Hours)] / [Avg Dwell Time (Hours)]

-- % of dwell time spent waiting to exit
Wait to Exit Ratio =
    [Avg Wait to Exit (Hours)] / [Avg Dwell Time (Hours)]
```

### Hàng Ghép (Transshipment Pair) — Time Analysis

```dax
-- Avg time: First vehicle enters → full pair assembled
Avg Time First Vehicle to Pair Complete =
    AVERAGEX(
        FILTER(
            'Combined_Jobs',
            NOT ISBLANK('Combined_Jobs'[time_in_ori1]) &&
            NOT ISBLANK('Combined_Jobs'[time_in_des1]) &&
            'Combined_Jobs'[time_in_des1] > 'Combined_Jobs'[time_in_ori1]
        ),
        'Combined_Jobs'[time_in_des1] - 'Combined_Jobs'[time_in_ori1]
    )

-- Avg time: Pair complete → last vehicle exits
Avg Time Pair Complete to Last Exit =
    AVERAGEX(
        FILTER(
            'Combined_Jobs',
            NOT ISBLANK('Combined_Jobs'[time_in_des1]) &&
            NOT ISBLANK('Combined_Jobs'[time_in_desMAX]) &&
            'Combined_Jobs'[time_in_desMAX] > 'Combined_Jobs'[time_in_des1]
        ),
        'Combined_Jobs'[time_in_desMAX] - 'Combined_Jobs'[time_in_des1]
    )

-- Avg total time: First vehicle in → last vehicle out
Avg Total Operation Time =
    AVERAGEX(
        FILTER(
            'Combined_Jobs',
            NOT ISBLANK('Combined_Jobs'[time_in_ori1]) &&
            NOT ISBLANK('Combined_Jobs'[time_in_desMAX]) &&
            'Combined_Jobs'[time_in_desMAX] > 'Combined_Jobs'[time_in_ori1]
        ),
        'Combined_Jobs'[time_in_desMAX] - 'Combined_Jobs'[time_in_ori1]
    )

-- Avg time: Customs cleared → transshipment starts
Avg Customs to Tranship Start =
    AVERAGEX(
        FILTER(
            'Combined_Jobs',
            NOT ISBLANK('Combined_Jobs'[transhipment_started_at]) &&
            NOT ISBLANK('Combined_Jobs'[customs_cleared_at]) &&
            'Combined_Jobs'[transhipment_started_at] > 'Combined_Jobs'[customs_cleared_at]
        ),
        'Combined_Jobs'[transhipment_started_at] - 'Combined_Jobs'[customs_cleared_at]
    )

-- Avg transshipment duration
Avg Transhipment Duration =
    AVERAGEX(
        FILTER(
            'Combined_Jobs',
            NOT ISBLANK('Combined_Jobs'[transhipment_finished_at]) &&
            NOT ISBLANK('Combined_Jobs'[transhipment_started_at]) &&
            'Combined_Jobs'[transhipment_finished_at] > 'Combined_Jobs'[transhipment_started_at]
        ),
        'Combined_Jobs'[transhipment_finished_at] - 'Combined_Jobs'[transhipment_started_at]
    )
```

### CN/VN Vehicle Pair Metrics

```dax
-- Avg time: CN vehicle enters → first VN vehicle enters
Avg CN In to First VN In =
    AVERAGEX(
        FILTER(
            'Combined_Jobs',
            NOT ISBLANK('Combined_Jobs'[time_in_ori1TQ]) &&
            NOT ISBLANK('Combined_Jobs'[time_in_des1VN]) &&
            'Combined_Jobs'[time_in_des1VN] > 'Combined_Jobs'[time_in_ori1TQ]
        ),
        'Combined_Jobs'[time_in_des1VN] - 'Combined_Jobs'[time_in_ori1TQ]
    )

-- Avg time: CN vehicle finishes service → exits yard
Avg CN End Service to Exit =
    AVERAGEX(
        FILTER(
            'vehicle_activities',
            'vehicle_activities'[type] = "CN" &&
            NOT ISBLANK('vehicle_activities'[transhipment_finished_at]) &&
            NOT ISBLANK('vehicle_activities'[time_out]) &&
            'vehicle_activities'[time_out] > 'vehicle_activities'[transhipment_finished_at]
        ),
        'vehicle_activities'[time_out] - 'vehicle_activities'[transhipment_finished_at]
    )

-- Avg CN service duration (first to last service)
Avg CN Service Duration =
    AVERAGEX(
        FILTER(
            'vehicle_activities',
            'vehicle_activities'[type] = "CN" &&
            NOT ISBLANK('vehicle_activities'[transhipment_started_at]) &&
            NOT ISBLANK('vehicle_activities'[transhipment_finished_at]) &&
            'vehicle_activities'[transhipment_finished_at] > 'vehicle_activities'[transhipment_started_at]
        ),
        'vehicle_activities'[transhipment_finished_at] - 'vehicle_activities'[transhipment_started_at]
    )

-- Avg VN service duration
Avg VN Service Duration =
    AVERAGEX(
        FILTER(
            'vehicle_activities',
            'vehicle_activities'[type] = "VN" &&
            NOT ISBLANK('vehicle_activities'[transhipment_started_at]) &&
            NOT ISBLANK('vehicle_activities'[transhipment_finished_at]) &&
            'vehicle_activities'[transhipment_finished_at] > 'vehicle_activities'[transhipment_started_at]
        ),
        'vehicle_activities'[transhipment_finished_at] - 'vehicle_activities'[transhipment_started_at]
    )
```

---

## 4. Vehicle Yard Traffic Report — Lượt Xe Lưu Bãi

### Core Vehicle Counts

```dax
-- Total unique vehicles
Total Vehicles =
    DISTINCTCOUNT('vehicle_activity_job'[vehicle_activity_id])

-- CN (Chinese) vehicles
Total CN Vehicles =
    CALCULATE(
        DISTINCTCOUNT('vehicle_activity_job'[vehicle_activity_id]),
        'vehicle_activities'[type] = "CN"
    )

-- VN (Vietnamese) vehicles
Total VN Vehicles =
    CALCULATE(
        DISTINCTCOUNT('vehicle_activity_job'[vehicle_activity_id]),
        'vehicle_activities'[type] = "VN"
    )

-- Total service jobs (transshipment pairs)
Total Service Jobs =
    DISTINCTCOUNT('vehicle_activity_job'[job_id])

-- Origin (incoming) vehicles
Vehicles Origin =
    CALCULATE(
        DISTINCTCOUNT('vehicle_activity_job'[vehicle_activity_id]),
        'vehicle_activity_job'[type] = "origin"
    )

-- Destination (receiving) vehicles
Vehicles Destination =
    CALCULATE(
        DISTINCTCOUNT('vehicle_activity_job'[vehicle_activity_id]),
        'vehicle_activity_job'[type] = "destination"
    )
```

### Transshipment (Hàng Ghép) Analysis

```dax
-- Vehicles with grouped cargo (hàng ghép)
Grouped Cargo Vehicles =
    CALCULATE(
        [Total Vehicles],
        'vehicle_activity_job'[Hàng Ghép] = "hàng ghép"
    )

-- % of total vehicles with grouped cargo
Grouped Cargo % =
    DIVIDE([Grouped Cargo Vehicles], [Total Vehicles], 0)

-- CN vehicles with grouped cargo
CN Grouped Cargo Vehicles =
    CALCULATE(
        [Total Vehicles],
        'vehicle_activity_job'[Hàng Ghép] = "hàng ghép",
        'vehicle_activities'[type] = "CN"
    )

-- VN vehicles with grouped cargo
VN Grouped Cargo Vehicles =
    CALCULATE(
        [Total Vehicles],
        'vehicle_activity_job'[Hàng Ghép] = "hàng ghép",
        'vehicle_activities'[type] = "VN"
    )

-- Grouped cargo % for CN vehicles
CN Grouped Cargo % =
    DIVIDE([CN Grouped Cargo Vehicles], [Total CN Vehicles], 0)

-- Grouped cargo % for VN vehicles
VN Grouped Cargo % =
    DIVIDE([VN Grouped Cargo Vehicles], [Total VN Vehicles], 0)
```

### Daily & Monthly Averages

```dax
-- Average vehicles per day
Vehicles per Day =
    DIVIDE([Total Vehicles], DISTINCTCOUNT('Calendar'[Date]), 0)

-- Average CN vehicles per day
CN Vehicles per Day =
    DIVIDE([Total CN Vehicles], DISTINCTCOUNT('Calendar'[Date]), 0)

-- Average VN vehicles per day
VN Vehicles per Day =
    DIVIDE([Total VN Vehicles], DISTINCTCOUNT('Calendar'[Date]), 0)

-- Average vehicles per month
Vehicles per Month =
    DIVIDE([Total Vehicles], DISTINCTCOUNT('Calendar'[Month-Year]), 0)

-- Average dwell time per vehicle across jobs (hours)
Avg Dwell Time per Vehicle =
    VAR SummaryTable =
        SUMMARIZE(
            'vehicle_activity_job',
            'vehicle_activity_job'[vehicle_activity_id],
            "TimeIn",  MIN('vehicle_activity_job'[time_in]),
            "TimeOut", MAX('vehicle_activity_job'[time_out])
        )
    RETURN
        AVERAGEX(
            ADDCOLUMNS(SummaryTable, "Duration", DATEDIFF([TimeIn], [TimeOut], MINUTE) / 60.0),
            [Duration]
        )
```

### Quarterly Distribution

```dax
-- Vehicles per day: Q1 2023
Vehicles per Day Q1 2023 =
    DIVIDE(
        CALCULATE(COUNTROWS('vehicle_activity_job'), 'Date'[Date] >= DATE(2023,1,1), 'Date'[Date] <= DATE(2023,3,31)),
        COUNTROWS(FILTER('Date', 'Date'[Date] >= DATE(2023,1,1) && 'Date'[Date] <= DATE(2023,3,31))),
        0
    )

-- % of annual volume in Q1 2023
Q1 2023 Share % =
    DIVIDE(
        CALCULATE(COUNTROWS('vehicle_activity_job'), 'Date'[Date] >= DATE(2023,1,1), 'Date'[Date] <= DATE(2023,3,31)),
        CALCULATE(COUNTROWS('vehicle_activity_job'), 'Date'[Date] >= DATE(2023,1,1), 'Date'[Date] <= DATE(2023,12,31)),
        0
    )
```

### Service Type Breakdown

```dax
-- By service type
Pallet Vehicles =
    CALCULATE(COUNTROWS('vehicle_activity_job'), 'service_jobs'[Service_type] = "Pallet")

Loading Vehicles =
    CALCULATE(COUNTROWS('vehicle_activity_job'), 'service_jobs'[Service_type] = "Bốc xếp")

Crane Vehicles =
    CALCULATE(COUNTROWS('vehicle_activity_job'), 'service_jobs'[Service_type] = "Cẩu")

Roof Cover Vehicles =
    CALCULATE(COUNTROWS('vehicle_activity_job'), 'service_jobs'[Service_type] = "Mái che")

-- Service share ratios
Pallet % = [Pallet Vehicles] / [Total Vehicles]
Loading % = [Loading Vehicles] / [Total Vehicles]
Crane % = [Crane Vehicles] / [Total Vehicles]
Roof Cover % = [Roof Cover Vehicles] / [Total Vehicles]
```

### Dynamic Color Formatting

```dax
-- Highlight max/min months in vehicle volume chart
Vehicle Volume Bar Color =
    VAR _max = MAXX(ALL('Calendar'[Month], 'Calendar'[Month Num], 'Calendar'[Month-Year]), [Total Vehicles])
    VAR _min = MINX(ALL('Calendar'[Month], 'Calendar'[Month Num], 'Calendar'[Month-Year]), [Total Vehicles])
    RETURN
        SWITCH(
            TRUE(),
            [Total Vehicles] = _max, "#036162",
            [Total Vehicles] = _min, "#ABE4AD",
            "#E7EBEF"
        )

-- Font color for highlighted bars
Vehicle Volume Font Color =
    VAR _max = MAXX(ALL('Calendar'[Month], 'Calendar'[Month Num], 'Calendar'[Month-Year]), [Total Vehicles])
    VAR _min = MINX(ALL('Calendar'[Month], 'Calendar'[Month Num], 'Calendar'[Month-Year]), [Total Vehicles])
    RETURN
        SWITCH(
            TRUE(),
            [Total Vehicles] = _max, "White",
            [Total Vehicles] = _min, "White",
            "#E7EBEF"
        )
```

---

## Notes

- Measures prefixed `Avg_` use `AVERAGEX` with `FILTER` to exclude blank or invalid time entries (e.g. `time_out > time_in` validation)
- Color formatting measures follow a consistent pattern: **max = dark blue `#1B539A`**, **min = light blue `#7698C2`**, **others = neutral `#E7EBEF`** (except vehicle report which uses teal)
- YTD/PYTD patterns use `DATESYTD` + `DATEADD(..., -1, YEAR)` for clean year-over-year comparison
- All time differences are computed in hours via `DATEDIFF(..., HOUR)` or `/ 60.0` from minutes for decimal precision
