<div align="center">

<img src="https://media.giphy.com/media/v1.Y2lkPWVjZjA1ZTQ3NWNqcXFmY3d5ZHk0b2RucjVyNDgwazNsZmpxZHJza2lnMmVuYWU4biZlcD12MV9naWZzX3NlYXJjaCZjdD1n/TLCpf8koMoNEWkToQD/giphy.gif" width="60%" alt="Sales Data Pipeline">

# 📊 Sales Data Pipeline — n8n

**An automated sales data pipeline built with n8n for retrieving, transforming, analyzing, and generating business sales reports.**

</div>

---

# 📌 Project Overview

## Project Name

**Sales Data Pipeline**

### Workflow Name

```text
Section 2 - Sales Data Pipeline
```

### Platform

**n8n**

### Course

**n8n Foundations — Section 2**

### Estimated Time

```text
60 minutes
```

### Tag

```text
n8n101
```

---

# 🎯 Project Objective

The objective of this project is to build an end-to-end sales data pipeline using n8n.

The workflow retrieves sales data from the n8n Academy sales warehouse, transforms individual orders, calculates order totals, filters delivered orders, generates regional summaries, creates a CSV report, and sends the resulting data to validation and reporting endpoints.

The complete process is:

```text
Retrieve
   ↓
Transform
   ↓
Process
   ↓
Filter
   ↓
Analyze
   ↓
Generate Report
   ↓
Finalize
```

The project demonstrates how a single data source can be transformed into different outputs for different business requirements.

---

# 🏗️ Architecture

The workflow uses a single sales-data retrieval step and then branches after the order transformation.

```text
                              ┌─────────────────────┐
                              │    TriggerManual    │
                              │    Manual Trigger   │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │    GetSalesData      │
                              │     HTTP Request     │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │     SplitOrders      │
                              │      Split Out       │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │    SetOrderTotals    │
                              │      Edit Fields     │
                              └──────────┬──────────┘
                                         │
                         ┌───────────────┴────────────────┐
                         │                                │
                         ▼                                ▼
              ┌─────────────────────┐          ┌─────────────────────┐
              │   AggregateOrders   │          │   FilterDelivered   │
              │      Aggregate      │          │         IF          │
              └──────────┬──────────┘          └──────────┬──────────┘
                         │                                │
                         ▼                         ┌──────┴──────┐
              ┌─────────────────────┐              │             │
              │     SendOrders      │            TRUE          FALSE
              │     Operations      │              │             │
              └─────────────────────┘              ▼             ▼
                                         ┌─────────────────┐ ┌─────────────────────┐
                                         │SummarizeByRegion│ │ IgnoreNonDelivered  │
                                         │    Summarize    │ │        NoOp          │
                                         └────────┬────────┘ └─────────────────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │UpdateFieldNames │
                                         │   Rename Keys   │
                                         └────────┬────────┘
                                                  │
                                      ┌───────────┴───────────┐
                                      │                       │
                                      ▼                       ▼
                           ┌─────────────────────┐  ┌──────────────────────┐
                           │  AggregateRegions   │  │  SetReportMetadata   │
                           │      Aggregate      │  │     Edit Fields      │
                           └──────────┬──────────┘  └──────────┬───────────┘
                                      │                        │
                                      ▼                        ▼
                           ┌─────────────────────┐  ┌──────────────────────┐
                           │    SendAnalysis     │  │     ConvertToCSV     │
                           │      Finance        │  │    Convert to File   │
                           └─────────────────────┘  └──────────┬───────────┘
                                                                │
                                                                ▼
                                                     ┌──────────────────────┐
                                                     │      SendReport       │
                                                     │      Management       │
                                                     └──────────────────────┘
```

### Business Flow

The three outputs serve different purposes:

```text
                         SetOrderTotals
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
            Operations                   Analysis
                 │                           │
                 ▼                           ▼
          All Orders                  Delivered Orders
                 │                           │
                 ▼                           ▼
          SendOrders                 Regional Summary
                                             │
                                  ┌──────────┴──────────┐
                                  │                     │
                                  ▼                     ▼
                              Finance              Management
                                  │                     │
                                  ▼                     ▼
                           SendAnalysis          CSV Report
                                                        │
                                                        ▼
                                                   SendReport
```

---

# 🏷️ Node Naming Convention

Every node in the workflow has a clear and descriptive name.

| Node Type       | Node Name           |
| --------------- | ------------------- |
| Manual Trigger  | `TriggerManual`     |
| HTTP Request    | `GetSalesData`      |
| Split Out       | `SplitOrders`       |
| Edit Fields     | `SetOrderTotals`    |
| Aggregate       | `AggregateOrders`   |
| HTTP Request    | `SendOrders`        |
| IF              | `FilterDelivered`   |
| Summarize       | `SummarizeByRegion` |
| Rename Keys     | `UpdateFieldNames`  |
| Aggregate       | `AggregateRegions`  |
| HTTP Request    | `SendAnalysis`      |
| Edit Fields     | `SetReportMetadata` |
| Convert to File | `ConvertToCSV`      |
| HTTP Request    | `SendReport`        |

Clear node names make the workflow easier to understand, debug, maintain, and share.

---

# 🔐 Authentication

The Academy API requires authentication.

Instead of passing the assessment ID through the URL, this workflow sends it through an HTTP header.

### Authentication

```text
Generic Credential Type
        ↓
Header Auth
        ↓
n8n Academy API Key
```

### Assessment Header

```text
Name:
X-Assessment-ID

Value:
[Your Assessment ID]
```

The same authentication configuration is used for every Academy HTTP request.

This includes:

```text
GetSalesData
SendOrders
SendAnalysis
SendReport
```

Using headers avoids putting the assessment identifier directly into the URL.

---

# 📥 Step 1 — Get the Sales Data

## Learning Objectives

This step demonstrates:

* Header authentication
* HTTP Request configuration
* JSON data inspection
* Nested JSON structures
* Working with arrays

---

## 1.1 Create the Workflow

Create a new n8n workflow named:

```text
Section 2 - Sales Data Pipeline
```

Add the tag:

```text
n8n101
```

Add a **Manual Trigger** node.

Rename it:

```text
TriggerManual
```

---

## 1.2 Configure GetSalesData

Add an HTTP Request node.

Rename it:

```text
GetSalesData
```

### Configuration

| Property        | Value                                                          |
| --------------- | -------------------------------------------------------------- |
| Method          | `GET`                                                          |
| URL             | `https://learn.app.n8n.cloud/webhook/course/n8n101/sales-data` |
| Authentication  | Generic Credential Type                                        |
| Credential Type | Header Auth                                                    |
| Credential      | n8n Academy API Key                                            |

Enable:

```text
Send Headers
```

Add:

```text
X-Assessment-ID: [Your Assessment ID]
```

The first part of the workflow is:

```text
TriggerManual
      ↓
GetSalesData
```

---

## 1.3 Inspect the Response

The API response contains:

```text
orders
generated_at
total_records
```

The `orders` field contains the individual sales orders.

Each order contains fields such as:

```text
order_id
customer_name
region
status
quantity
unit_price
```

The response also contains metadata:

```text
generated_at
total_records
```

### Checkpoint

The response should contain:

```text
total_records: 50
```

and an:

```text
orders
```

array containing 50 orders.

---

# 🔄 Step 2 — Transform the Data

## Learning Objectives

This step demonstrates:

* Array processing
* Split Out
* n8n expressions
* Calculations
* Edit Fields
* Aggregate
* Preparing data for downstream processing

---

# 2.1 Split the Orders Array

The API returns all orders inside a single array.

Add a **Split Out** node.

Rename it:

```text
SplitOrders
```

### Configuration

```text
Field To Split Out:
orders
```

```text
Include:
No Other Fields
```

Execute the node.

The result should contain:

```text
50 items
```

Each item now represents one order.

```text
orders[50]
      ↓
Order 1
Order 2
Order 3
...
Order 50
```

---

# 2.2 Calculate Order Totals

Add an **Edit Fields** node after `SplitOrders`.

Rename it:

```text
SetOrderTotals
```

Set:

```text
Include Other Input Fields:
Off
```

Add the following fields:

| Name            | Type   | Value                                     |
| --------------- | ------ | ----------------------------------------- |
| `order_id`      | String | `{{ $json.order_id }}`                    |
| `customer_name` | String | `{{ $json.customer_name }}`               |
| `region`        | String | `{{ $json.region }}`                      |
| `status`        | String | `{{ $json.status }}`                      |
| `order_total`   | Number | `{{ $json.quantity * $json.unit_price }}` |

---

## Order Total Formula

The calculated field uses:

```text
order_total = quantity × unit_price
```

n8n expression:

```text
{{ $json.quantity * $json.unit_price }}
```

For example:

```text
Quantity   = 4
Unit Price = 25

Order Total = 4 × 25
            = 100
```

The workflow is now:

```text
GetSalesData
      ↓
SplitOrders
      ↓
SetOrderTotals
```

---

# 2.3 Operations Branch

The first branch sends all transformed orders for processing.

From:

```text
SetOrderTotals
```

connect to:

```text
AggregateOrders
```

### AggregateOrders

Configure:

```text
Aggregate:
All Item Data (Into a Single List)
```

Output field:

```text
orders
```

This converts the 50 individual items back into a single payload:

```json
{
  "orders": [
    "... transformed orders ..."
  ]
}
```

---

# 📤 SendOrders

Add an HTTP Request node after `AggregateOrders`.

Rename it:

```text
SendOrders
```

### Configuration

```text
Method:
POST
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n101/process-orders
```

Use the Academy Header Auth credential.

Add:

```text
X-Assessment-ID:
[Your Assessment ID]
```

### Body

Select:

```text
JSON
```

Using Fields Below:

```text
Name:
orders

Value:
{{ $json.orders }}
```

---

# ✅ Process Orders Checkpoint

The processing endpoint should return:

```json
{
  "status": "processed",
  "orders_received": 50,
  "transformation_valid": true,
  "confirmation_code": "N8N-FOUNDATIONS-S2-PROCESS_ORDERS-2026-VSY7N4WY"
}
```

The important validation values are:

```text
status = processed
orders_received = 50
transformation_valid = true
```

The confirmation code is generated by the Academy endpoint.

---

# 🌿 Step 3 — Filter, Aggregate, and Summarize

The second branch is used for analysis and reporting.

From:

```text
SetOrderTotals
```

create another connection to:

```text
FilterDelivered
```

This gives the workflow two primary paths:

```text
SetOrderTotals
      │
      ├──────────────► AggregateOrders ─────► SendOrders
      │
      └──────────────► FilterDelivered
```

---

# 3.1 Filter Delivered Orders

Add an **IF** node.

Rename it:

```text
FilterDelivered
```

Configure:

### Value 1

```text
{{ $json.status }}
```

### Operation

```text
equals
```

### Value 2

```text
delivered
```

The node produces two outputs:

```text
TRUE
 ↓
Delivered Orders
```

and:

```text
FALSE
 ↓
Non-delivered Orders
```

Only the `TRUE` output continues into the reporting pipeline.

---

# 3.2 Summarize by Region

Connect the `TRUE` output of `FilterDelivered` to a **Summarize** node.

Rename it:

```text
SummarizeByRegion
```

Configure three aggregations.

### Total Order Value

```text
Aggregation:
Sum

Field:
order_total
```

### Order Count

```text
Aggregation:
Count

Field:
order_total
```

### Average Order Value

```text
Aggregation:
Average

Field:
order_total
```

Under:

```text
Fields to Split By
```

add:

```text
region
```

The node should produce:

```text
4 items
```

with one summary for each region.

---

# 3.3 Rename Summary Fields

The Summarize node generates aggregation-based field names.

Add a **Rename Keys** node.

Rename it:

```text
UpdateFieldNames
```

Configure:

| Current Key Name      | New Key Name          |
| --------------------- | --------------------- |
| `sum_order_total`     | `order_totals`        |
| `count_order_total`   | `order_count`         |
| `average_order_total` | `average_order_total` |

The regional data is now easier to consume.

---

# 3.4 Finance Branch

From:

```text
UpdateFieldNames
```

connect to:

```text
AggregateRegions
```

Configure:

```text
Aggregate:
All Item Data (Into a Single List)
```

Output field:

```text
regions
```

The output becomes:

```json
{
  "regions": [
    "... Region 1 ...",
    "... Region 2 ...",
    "... Region 3 ...",
    "... Region 4 ..."
  ]
}
```

---

# 📤 SendAnalysis

Add an HTTP Request node after `AggregateRegions`.

Rename it:

```text
SendAnalysis
```

### Configuration

```text
Method:
POST
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n101/validate-analysis
```

Use the same Header Auth credential.

Add:

```text
X-Assessment-ID:
[Your Assessment ID]
```

### JSON Body

```text
Name:
regions

Value:
{{ $json.regions }}
```

---

# ✅ Validate Analysis Checkpoint

The endpoint should return a response containing:

```json
{
  "status": "valid",
  "analysis_valid": true,
  "regions_validated": 4,
  "confirmation_code": "N8N-FOUNDATIONS-S2-VALIDATE_ANALYSIS-2026-4IDGWMSZ"
}
```

The important validation values are:

```text
status = valid
analysis_valid = true
regions_validated = 4
```

---

# 📄 Step 4 — Generate and Finalize the Report

The management report uses the same regional summary data.

A second branch is created from:

```text
UpdateFieldNames
```

The structure is:

```text
UpdateFieldNames
       │
       ├──────────────► AggregateRegions ─────► SendAnalysis
       │
       └──────────────► SetReportMetadata
```

---

# 4.1 Prepare Report Metadata

Add an **Edit Fields** node after `UpdateFieldNames`.

Rename it:

```text
SetReportMetadata
```

Add:

### Report Timestamp

```text
Name:
report_generated
```

```text
Type:
String
```

```text
Value:
{{ $now.format('yyyy-MM-dd HH:mm:ss') }}
```

Add:

### Assessment ID

```text
Name:
assessment_id
```

```text
Value:
[Your Assessment ID]
```

Set:

```text
Include Other Input Fields:
On
```

This preserves the regional summary fields while adding report metadata.

---

# 4.2 Convert to CSV

Add a **Convert to File** node.

Rename it:

```text
ConvertToCSV
```

Configure:

```text
Operation:
Convert to CSV
```

```text
Put Output File in Field:
report
```

Execute the node.

The output should contain binary data.

Open the output and switch to:

```text
Binary
```

to verify the generated CSV.

---

# 📑 Report Fields

The generated report contains the regional summary information and metadata.

Expected fields include:

```text
region
order_totals
order_count
average_order_total
report_generated
assessment_id
```

---

# 📤 4.3 Finalize the Report

Duplicate the `SendAnalysis` HTTP Request node.

Connect it after:

```text
ConvertToCSV
```

Rename it:

```text
SendReport
```

### Configuration

```text
Method:
POST
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n101/send-report
```

Use the same Header Auth credential and:

```text
X-Assessment-ID:
[Your Assessment ID]
```

---

# 📦 Binary File Configuration

Set:

```text
Body Content Type:
n8n Binary File
```

Add:

```text
Input Data Field Name:
report
```

The field must match the binary field produced by:

```text
ConvertToCSV
```

---

# 🏆 Final Assessment Checkpoint

A successful report submission should return a response similar to:

```json
{
  "status": "success",
  "section": 2,
  "confirmation_code": "N8N-FOUNDATIONS-S2-SEND-REPORT-2026-OS2R1TX",
  "message": "Congratulations! You've completed Section 2. Your sales data pipeline is production-ready.",
  "skills_verified": [
    "Header Authentication",
    "Branching",
    "Data Transformation",
    "Expressions",
    "Filtering",
    "Aggregation",
    "File Generation"
  ]
}
```

---

# 🔀 Final Workflow

The final n8n workflow is:

```text
TriggerManual
      │
      ▼
GetSalesData
      │
      ▼
SplitOrders
      │
      ▼
SetOrderTotals
      │
      ├───────────────────────────────┐
      │                               │
      ▼                               ▼
AggregateOrders                FilterDelivered
      │                               │
      ▼                        ┌──────┴──────┐
SendOrders                     │             │
                          TRUE │           FALSE
                               │             │
                               ▼             ▼
                       SummarizeByRegion  IgnoreNonDelivered
                               │
                               ▼
                       UpdateFieldNames
                               │
                      ┌────────┴────────┐
                      │                 │
                      ▼                 ▼
              AggregateRegions    SetReportMetadata
                      │                 │
                      ▼                 ▼
                SendAnalysis       ConvertToCSV
                                        │
                                        ▼
                                   SendReport
```

---

# 🚫 Optional — Ignore Non-Delivered Orders

The `FALSE` output of `FilterDelivered` contains orders whose status is not:

```text
delivered
```

For example:

```text
pending
cancelled
```

An optional **NoOp** node can be connected to this output.

Rename it:

```text
IgnoreNonDelivered
```

Add a Sticky Note explaining:

```text
Non-delivered orders are excluded from the
regional financial report because the report
is based on delivered orders only.
```

This makes the filtering decision explicit.

---

# 📝 Optional — Workflow Documentation

Add a Sticky Note using:

```text
Shift + S
```

Suggested content:

```markdown
# Academy Section 2 Sales Data Pipeline

Created by: [Your Name]

Date: [Today's Date]

This workflow retrieves sales data, transforms individual
orders, calculates order totals, processes all transformed
orders, filters delivered orders, generates regional
summaries, creates a CSV report, and submits the results
to the Academy validation endpoints.

## Branches

- Operations → All transformed orders
- Finance → Regional summaries
- Management → CSV report

## Authentication

Uses Header Auth and X-Assessment-ID.

Part of:
n8n Foundations Course — Section 2
```

---

# 🔔 Optional — Notifications

The workflow can be extended with notifications after successful execution.

For example:

```text
SendOrders
      ↓
Slack / Discord
      ↓
Pipeline Completed
```

A notification could include:

```text
Sales pipeline complete.

Orders processed: 50
Delivered orders: [count]
Regions analyzed: 4
Report generated: Yes
```

---

# 📧 Optional — Email the Report

The generated CSV can be sent to management through an email node.

```text
ConvertToCSV
      ↓
Send Email
      ↓
Management
```

The CSV can be attached as a binary file.

---

# 🚨 Optional — Anomaly Detection

The regional summaries can also be checked for unusual values.

```text
Regional Summary
      ↓
IF
      ↓
Unusual Result?
      │
      ├── YES → Send Alert
      │
      └── NO  → Continue
```

This could be used to detect unusually low order counts or other business conditions.

---

# 🧪 Testing Checklist

Use the following checklist to verify the completed workflow.

### Data Retrieval

```text
☐ GetSalesData executes successfully
☐ 50 orders are returned
☐ total_records = 50
☐ orders array is present
```

### Transformation

```text
☐ SplitOrders produces 50 items
☐ SetOrderTotals creates order_total
☐ order_total = quantity × unit_price
```

### Processing

```text
☐ AggregateOrders creates orders array
☐ SendOrders succeeds
☐ orders_received = 50
☐ transformation_valid = true
```

### Filtering

```text
☐ FilterDelivered checks status
☐ TRUE branch contains delivered orders
☐ FALSE branch contains non-delivered orders
```

### Analysis

```text
☐ SummarizeByRegion executes
☐ Regional summaries are created
☐ 4 regions are returned
☐ Field names are updated
☐ SendAnalysis succeeds
☐ analysis_valid = true
```

### Report

```text
☐ SetReportMetadata adds timestamp
☐ Assessment ID is included
☐ ConvertToCSV creates binary output
☐ SendReport receives binary field
☐ Final report submission succeeds
```

---

# 🛠️ Troubleshooting

## Authentication Failed

Verify:

```text
Header Auth credential
```

and:

```text
X-Assessment-ID
```

Make sure the correct API key and assessment ID are being used.

---

## Cannot Read Property of Undefined

Inspect the JSON output of:

```text
GetSalesData
```

Verify that the array is named:

```text
orders
```

Then verify `SplitOrders` uses exactly:

```text
orders
```

---

## No Items After FilterDelivered

Check:

```text
TRUE output
```

versus:

```text
FALSE output
```

Also verify that the condition is:

```text
{{ $json.status }}
equals
delivered
```

Status values are case-sensitive.

---

## Incorrect Order Totals

Verify:

```text
{{ $json.quantity * $json.unit_price }}
```

and confirm that the incoming fields are:

```text
quantity
unit_price
```

---

## Binary File Not Found

Check that:

```text
ConvertToCSV
```

outputs the file into:

```text
report
```

and that:

```text
SendReport
```

uses:

```text
Input Data Field Name:
report
```

---

## Node Reference Not Found

Node names are case-sensitive.

For example:

```text
$('GetSalesData')
```

is different from:

```text
$('Get Sales Data')
```

Use the exact node name when referencing another node.

---

# 📊 Evaluation Criteria

| Criteria            | Requirement                                  |
| ------------------- | -------------------------------------------- |
| Data Retrieved      | 50 orders fetched from `sales-data` endpoint |
| Authentication      | Header Auth and `X-Assessment-ID` used       |
| Data Transformation | Orders split into individual items           |
| Calculations        | `order_total` calculated correctly           |
| Processing          | 50 transformed orders accepted               |
| Filtering           | Delivered orders selected                    |
| Aggregation         | Regional summaries generated                 |
| Analysis            | 4 regions validated                          |
| Report Generation   | Valid CSV file created                       |
| Binary Handling     | CSV transmitted as binary                    |
| Finalization        | Report successfully submitted                |
| Confirmation        | Confirmation code received                   |

---

# 🎯 Success Criteria

The pipeline is successfully completed when:

```text
✓ 50 sales orders retrieved
✓ Orders split into individual items
✓ Order totals calculated
✓ All transformed orders aggregated
✓ Processing validation succeeds
✓ 50 orders received by processing endpoint
✓ Delivered orders filtered
✓ Regional summaries generated
✓ 4 regions validated
✓ Analysis validation succeeds
✓ Report metadata added
✓ CSV report generated
✓ Binary report submitted
✓ Final report accepted
✓ Confirmation code received
```

---

# 📚 Skills Demonstrated

This project demonstrates practical experience with:

* n8n workflow automation
* HTTP Request nodes
* Header authentication
* Credential management
* JSON data structures
* Nested arrays
* Split Out
* Edit Fields
* n8n expressions
* Mathematical calculations
* Workflow branching
* IF conditions
* Data filtering
* Aggregate
* Summarize
* Rename Keys
* Regional analysis
* CSV generation
* Binary data
* HTTP file uploads
* API validation
* Workflow documentation
* End-to-end data pipelines

---

# 💡 Key Concepts Learned

## Header Authentication

Sensitive identifiers should be passed through headers rather than URL query parameters.

```text
HTTP Request
      │
      ├── Header Auth
      │
      └── X-Assessment-ID
```

## Data Transformation

Raw API data can be converted into individual records and enriched with calculated fields.

```text
Raw Order
    ↓
Split
    ↓
Transform
    ↓
Calculated Order Total
```

## Branching

A single data source can support multiple downstream business processes.

```text
                    Transformed Data
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
         Operations     Finance     Management
```

## Aggregation

Individual records can be combined into lists or summarized into business statistics.

```text
50 Orders
   ↓
Aggregate
   ↓
orders[]
```

or:

```text
Delivered Orders
      ↓
Summarize
      ↓
Regional Statistics
```

## Binary Data

Structured JSON can be converted into a physical report file and transmitted through an HTTP request.

```text
JSON
 ↓
Convert to CSV
 ↓
Binary File
 ↓
HTTP Upload
```

---

# 🔮 Future Improvements

A production version could replace the Academy endpoints with real business systems.

### Database Integration

```text
n8n
 ↓
MySQL / PostgreSQL
```

### Scheduled Pipeline

Replace the manual trigger with:

```text
Schedule Trigger
      ↓
Daily Sales Pipeline
```

### Dashboard

```text
Sales API
    ↓
n8n
    ↓
Database
    ↓
BI Dashboard
```

### Error Handling

```text
API Request
     ↓
  Success?
   /    \
 YES     NO
 │        │
 ▼        ▼
Continue Retry
          │
       Failure
          │
          ▼
      Alert Team
```

### Notifications

```text
Pipeline Complete
       ↓
Slack / Discord / Email
       ↓
Business Team
```

---

# 🏁 Final Result

The completed project creates an end-to-end automated sales reporting pipeline:

```text
┌──────────────────────┐
│    Sales Data API    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    GetSalesData      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│     SplitOrders      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    SetOrderTotals    │
└──────────┬───────────┘
           │
      ┌────┴────┐
      │         │
      ▼         ▼
 Operations   Analysis
      │         │
      ▼         ▼
SendOrders  Delivered
              Orders
                │
                ▼
          Regional Summary
                │
          ┌─────┴─────┐
          │           │
          ▼           ▼
      SendAnalysis  CSV Report
                       │
                       ▼
                   SendReport
```

The pipeline follows the complete data-processing lifecycle:

```text
FETCH
  ↓
SPLIT
  ↓
TRANSFORM
  ↓
BRANCH
  ↓
FILTER
  ↓
AGGREGATE
  ↓
ANALYZE
  ↓
GENERATE
  ↓
DELIVER
```

---

# 📁 Repository Structure

```text
sales-data-pipeline/
│
├── README.md
│   └── section-2-sales-data-pipeline.json
│   ├── workflow.png
```

---

# 📜 Project Notes

This project is intended for learning, experimentation, and demonstration of n8n data-pipeline concepts.

The n8n Academy endpoints and associated course resources belong to their respective owners.

**Do not commit real API credentials or assessment identifiers to a public repository.**

---

<div align="center">

<img src="https://media.giphy.com/media/v1.Y2lkPWVjZjA1ZTQ3cjFxeDUzbmNiYTY5ejhoMmowMDZrbm1scWs1YWdrYmtjOHZwYmJyNiZlcD12MV9naWZzX3NlYXJjaCZjdD1n/SvckSy7fFviqrq8ClF/giphy.gif" width="70%" alt="Sales Data Pipeline Footer">

**Sales Data Pipeline · Built with n8n**

</div>
