# Enterprise Fleet Maintenance & PM Compliance Analytics

An end-to-end fleet analytics and data pipeline solution that transforms raw repair order logs into operational metrics. This project automates data generation via **Google Apps Script**, syncs structured records to **Google Sheets**, and publishes an interactive operational dashboard on **Tableau Public**.

![Tableau Public Dashboard Banner](https://img.shields.io/badge/Tableau_Public-Live_Dashboard-E97627?style=for-the-badge&logo=Tableau&logoColor=white)
![Google Apps Script](https://img.shields.io/badge/Google_Apps_Script-Automated_Pipeline-4285F4?style=for-the-badge&logo=JavaScript&logoColor=white)

---

## Executive Summary

Unscheduled fleet downtime and reactive repair expenses present significant operational drag for nationwide logistics operations. This project establishes an automated data pipeline and executive summary framework to:
* **Monitor PM (Preventative Maintenance) Compliance** against an $85\%$ target.
* **Quantify Total Downtime Financial Impact** by combining direct repair costs with unscheduled downtime revenue losses ($\$75/\text{hr}$ industry benchmark).
* **Track Unscheduled Spend Ratios** to keep reactive breakdown expenses below $30\%$ of total maintenance spend.
* **Analyze Vendor Lead Times & Regional Hub Performance** across 10 US logistics centers.

---

## 🛠️ Tech Stack & Architecture
[ Google Apps Script ] ➔ [ Google Sheets Data Warehouse ] ➔ [ Tableau Public Web Authoring ]
* **Automation & ETL:** Google Apps Script (JavaScript ES6)
* **Data Storage / Warehouse:** Google Sheets
* **BI & Data Visualization:** Tableau Public

---

## 📊 Core Business Logic & Metrics

| Metric Name | Formula / Logic | Target Benchmark |
| :--- | :--- | :--- |
| **PM Compliance %** | `SUM(Compliant or Due Soon WOs) / COUNT(Total WOs)` | $\ge 85.0\%$ |
| **Total Financial Impact** | `SUM(Repair Cost) + (SUM(Unscheduled Downtime Hrs) * $75)` | Minimize |
| **Unscheduled Spend Ratio** | `SUM(Non-PM Costs) / SUM(Total Repair Costs)` | $< 30.0\%$ |
| **Unit Operational Status** | `IF Downtime > 0 THEN "Grounded" ELSEIF PM Overdue THEN "Attention" ELSE "Active"` | Operational Readiness |

---

## 🚀 Setup & Installation Guide

Follow these steps to replicate the data pipeline and deploy the dashboard.

### Prerequisites
* A Google Account (Google Drive & Sheets)
* A [Tableau Public Account](https://public.tableau.com/)

---

### Step 1: Set Up the Google Sheet & Apps Script

1. Open [Google Sheets](https://sheets.google.com) and create a **Blank Spreadsheet**.
2. Rename the spreadsheet: `Fleet_Maintenance_Data_Warehouse`.
3. In the top menu, navigate to **Extensions** $\rightarrow$ **Apps Script**.
4. Clear any default code in `Code.gs` and paste the following script:

```javascript
/**
 * Generates 500 realistic fleet maintenance records for Tableau analytics.
 * Creates/clears a sheet tab named 'Fleet_Maintenance_Data'.
 */
function generateFleetData() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheetName = "Fleet_Maintenance_Data";
  var sheet = ss.getSheetByName(sheetName);

  if (!sheet) {
    sheet = ss.insertSheet(sheetName);
  } else {
    sheet.clearContents();
  }

  // Header Row
  var headers = [
    "Unit_ID",
    "Vehicle_Type",
    "Hub_Location",
    "State",
    "Region",
    "Odometer_Miles",
    "PM_Status",
    "WO_Number",
    "Repair_Category",
    "Vendor_Name",
    "Labor_Hours",
    "Total_Repair_Cost",
    "Unscheduled_Downtime_Hrs",
    "Service_Date"
  ];

  sheet.appendRow(headers);

  // Data pools
  var vehicleTypes = [
    { type: "Class 8 Tractor", minOdo: 150000, maxOdo: 650000, avgCost: 1200 },
    { type: "Medium Duty Straight Truck", minOdo: 80000, maxOdo: 350000, avgCost: 800 },
    { type: "Cargo Van", minOdo: 35000, maxOdo: 180000, avgCost: 450 },
    { type: "Service Trailer", minOdo: 20000, maxOdo: 250000, avgCost: 350 }
  ];

  var hubs = [
    { city: "Allentown", state: "PA", region: "Northeast" },
    { city: "Boston", state: "MA", region: "Northeast" },
    { city: "Atlanta", state: "GA", region: "Southeast" },
    { city: "Charlotte", state: "NC", region: "Southeast" },
    { city: "Chicago", state: "IL", region: "Midwest" },
    { city: "Columbus", state: "OH", region: "Midwest" },
    { city: "Dallas", state: "TX", region: "South" },
    { city: "Houston", state: "TX", region: "South" },
    { city: "Denver", state: "CO", region: "West" },
    { city: "Phoenix", state: "AZ", region: "West" }
  ];

  var pmStatuses = ["Compliant", "Compliant", "Compliant", "Due Soon", "Overdue"];
  
  var repairCategories = [
    "Preventative Maintenance",
    "Preventative Maintenance",
    "Brake System",
    "Engine / Powertrain",
    "Electrical / Sensors",
    "Tires & Alignment",
    "HVAC",
    "Suspension & Steering"
  ];

  var vendorPrefixes = [
    "National Fleet Service", 
    "Fleet Care", 
    "Apex Heavy Truck Repair", 
    "Pinnacle Auto & Fleet", 
    "Precision Mobile Mechanic"
  ];

  var rows = [];
  var startDate = new Date(2025, 8, 1);
  var endDate = new Date(2026, 8, 23);

  for (var i = 1; i <= 500; i++) {
    var vType = vehicleTypes[Math.floor(Math.random() * vehicleTypes.length)];
    var hub = hubs[Math.floor(Math.random() * hubs.length)];
    
    var unitPrefix = vType.type.substring(0, 3).toUpperCase();
    var unitNumber = Math.floor(1000 + Math.random() * 9000);
    var unitId = unitPrefix + "-" + unitNumber;

    var odometer = Math.floor(vType.minOdo + Math.random() * (vType.maxOdo - vType.minOdo));
    var pmStatus = pmStatuses[Math.floor(Math.random() * pmStatuses.length)];
    var woNumber = "RO-" + Math.floor(100000 + Math.random() * 900000);
    var category = repairCategories[Math.floor(Math.random() * repairCategories.length)];
    var vendor = vendorPrefixes[Math.floor(Math.random() * vendorPrefixes.length)] + " (" + hub.city + ")";

    var isUnscheduled = (category !== "Preventative Maintenance");
    var laborHours = isUnscheduled ? (Math.random() * 12 + 1).toFixed(1) : (Math.random() * 3 + 1).toFixed(1);
    var baseCost = vType.avgCost * (category === "Engine / Powertrain" ? 2.5 : category === "Brake System" ? 1.4 : 1.0);
    var totalCost = Math.round(baseCost * (0.6 + Math.random() * 0.8));
    var downtimeHrs = isUnscheduled ? Math.floor(Math.random() * 48 + 4) : 0;

    var randomTime = startDate.getTime() + Math.random() * (endDate.getTime() - startDate.getTime());
    var serviceDate = new Date(randomTime);
    var formattedDate = Utilities.formatDate(serviceDate, ss.getSpreadsheetTimeZone(), "yyyy-MM-dd");

    rows.push([
      unitId,
      vType.type,
      hub.city,
      hub.state,
      hub.region,
      odometer,
      pmStatus,
      woNumber,
      category,
      vendor,
      parseFloat(laborHours),
      totalCost,
      downtimeHrs,
      formattedDate
    ]);
  }

  sheet.getRange(2, 1, rows.length, headers.length).setValues(rows);
  
  var headerRange = sheet.getRange(1, 1, 1, headers.length);
  headerRange.setFontWeight("bold");
  headerRange.setBackground("#1F4E78");
  headerRange.setFontColor("#FFFFFF");
  
  sheet.autoResizeColumns(1, headers.length);

  Logger.log("SUCCESS: Generated 500 fleet maintenance records.");
}
