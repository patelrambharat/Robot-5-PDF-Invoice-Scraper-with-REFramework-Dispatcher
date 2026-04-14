# 🤖 Robot 5 — PDF Invoice Scraper with REFramework Dispatcher

> **Project Type:** UiPath Process (Dispatcher)
> **Framework:** Windows | Visual Basic
> **Version:** 1.0.1
> **Studio Version:** 26.0.190.0
> **Author:** Rambharat Patel
> **Email:** patelrambharat@gmail.com

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Project Structure](#project-structure)
4. [Workflow Details](#workflow-details)
5. [Variables](#variables)
6. [Dependencies](#dependencies)
7. [Queue Trigger Setup (How It Works)](#queue-trigger-setup)
8. [Step-by-Step: Queue Trigger Configuration](#step-by-step-queue-trigger-configuration)
9. [How to Run](#how-to-run)
10. [Error Handling](#error-handling)
11. [Notes & Best Practices](#notes--best-practices)

---

## 📌 Project Overview

This project is the **Dispatcher** component of a **REFramework-based PDF Invoice Scraper** automation. Its primary role is to:

- Scan a designated **Input folder** for PDF invoice files
- Read each PDF file from the folder
- **Add each PDF file as a Queue Item** into the UiPath Orchestrator Queue (`InvoiceQueue`)
- Handle any errors gracefully during the queue loading process
- Log success and failure messages for each file processed

This Dispatcher works in tandem with a **Performer** robot that picks up items from the queue and processes/scrapes data from each PDF invoice.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│              REFramework Architecture                │
│                                                     │
│  ┌─────────────────┐        ┌────────────────────┐  │
│  │   DISPATCHER    │        │     PERFORMER      │  │
│  │  (This Robot)   │──────▶ │  (Separate Robot)  │  │
│  │                 │ Queue  │                    │  │
│  │ Scans PDF files │ Items  │ Processes invoices │  │
│  │ Adds to Queue   │        │ Scrapes data       │  │
│  └─────────────────┘        └────────────────────┘  │
│           │                          │               │
│           ▼                          ▼               │
│   Orchestrator Queue: InvoiceQueue                  │
└─────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
Robot 5 PDF Invoice Scraper with REFramework Dispatcher/
│
├── Main.xaml               ← Main entry point workflow
├── project.json            ← Project configuration & dependencies
├── entry-points.json       ← Registered entry points
├── README.md               ← This file
└── Data/
    └── Input/              ← Folder watched for PDF invoice files
```

---

## 🔄 Workflow Details

### Main.xaml — Activity Breakdown

| Activity | Display Name | Purpose |
|----------|-------------|---------|
| `Sequence` | Process PDF Invoice Files | Root container for the entire workflow |
| `ManualTrigger` | Manual Trigger | Allows manual triggering of the process |
| `ForEachFileX` | Iterate Through Each PDF Invoice File | Loops through all `.pdf` files in the Input folder |
| `Sequence` | Process Current PDF File | Handles processing for each individual file |
| `TryCatch` | Handle Queue Item Addition With Error Handling | Wraps queue loading in error handling |
| `Sequence` | Add Invoice File To Queue | Try block — adds the file to the queue |
| `AddQueueItem` | Add Invoice File As Queue Item | Adds the file to `InvoiceQueue` in Orchestrator |
| `LogMessage` | Log Successfully Added Invoice File | Logs success with the full file path |
| `Sequence` | Handle Queue Addition Error | Catch block — handles failures |
| `LogMessage` | Log Queue Item Addition Failure | Logs the error message with file name |
| `Sequence` | Finalize File Processing | Finally block — runs after every file |

### 📂 Input Folder Path
```
E:\UiPath\Robot 5 PDF Invoice Scraper with REFramework Performer\Data\Input
```

### 🗂️ Queue Details
| Property | Value |
|----------|-------|
| **Queue Name** | `InvoiceQueue` |
| **Orchestrator Folder** | `Operations` |
| **Priority** | Normal |
| **Reference** | File Name (`CurrentFile.Name`) |

---

## 📦 Variables

| Variable Name | Type | Description |
|--------------|------|-------------|
| `CurrentFile` | `FileInfo` | Holds the current PDF file being processed |
| `CurrentIndex` | `Int32` | Index of the current iteration |
| `UiPathEventConnector` | `String` | Connector identifier for UiPath event trigger |
| `UiPathEvent` | `String` | Event name received from trigger |
| `UiPathEventObjectType` | `String` | Type of object that triggered the event |
| `UiPathEventObjectId` | `String` | ID of the object that triggered the event |
| `UiPathAdditionalEventData` | `String` | Additional data passed along with the event |

---

## 📦 Dependencies

| Package | Version |
|---------|---------|
| `UiPath.System.Activities` | 26.2.4 |

---

## ⚡ Queue Trigger Setup

This robot was designed to be triggered **automatically using a Queue Trigger** in UiPath Orchestrator. A Queue Trigger fires the Dispatcher robot automatically based on queue conditions — no manual scheduling needed!

### How the Queue Trigger Works

```
┌──────────────────────────────────────────────────────────────┐
│                     Queue Trigger Flow                       │
│                                                              │
│  PDF Files Drop      Queue Trigger     Dispatcher Robot      │
│  into Input Folder ──▶ Fires in  ───▶  Reads PDF files  ──▶ │
│                        Orchestrator    Adds to Queue         │
│                             │                                │
│                             ▼                                │
│                     Performer Robot                          │
│                     Picks up Queue Items                     │
│                     Processes Invoices                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step: Queue Trigger Configuration

Follow these steps to configure the **Queue Trigger** in UiPath Orchestrator so this Dispatcher robot runs automatically:

---

### ✅ Step 1 — Create the Queue in Orchestrator

1. Log in to **UiPath Orchestrator**
2. Navigate to your folder: **`Operations`**
3. Go to **Monitoring → Queues**
4. Click **➕ Add Queue**
5. Fill in the details:
   - **Name:** `InvoiceQueue`
   - **Max # of retries:** `3` *(recommended)*
   - **Auto Retry:** ✅ Enabled
   - **Specific Data Schema:** Optional
6. Click **Add** to save

---

### ✅ Step 2 — Publish the Dispatcher Robot to Orchestrator

1. In **UiPath Studio**, open this project
2. Go to **Home → Publish**
3. Set the **Publish target** to your Orchestrator instance
4. Confirm the **Package Name:** `Robot 5 PDF Invoice Scraper with REFramework Dispatcher`
5. Click **Publish**

---

### ✅ Step 3 — Create a Process in Orchestrator

1. In Orchestrator, go to **Automations → Processes**
2. Click **➕ Add Process**
3. Select the published package: **Robot 5 PDF Invoice Scraper with REFramework Dispatcher**
4. Assign it to your **Robot / Machine**
5. Click **Create**

---

### ✅ Step 4 — Set Up the Queue Trigger

1. In Orchestrator, go to **Automations → Triggers**
2. Click **➕ Add Trigger** → Select **Queue Trigger**
3. Configure the trigger:

   | Setting | Value |
   |---------|-------|
   | **Name** | `InvoiceQueue Dispatcher Trigger` |
   | **Queue** | `InvoiceQueue` (in `Operations` folder) |
   | **Process** | `Robot 5 PDF Invoice Scraper with REFramework Dispatcher` |
   | **Min items to trigger** | `1` |
   | **Max pending/running jobs** | `1` |
   | **Target Robot** | Your assigned robot |

4. Click **Add** to save the trigger

---

### ✅ Step 5 — Test the Queue Trigger

1. Drop one or more **PDF invoice files** into the Input folder:
   ```
   E:\UiPath\Robot 5 PDF Invoice Scraper with REFramework Performer\Data\Input
   ```
2. Manually run the **Dispatcher** once to populate the queue
3. Verify in Orchestrator → **Queues → InvoiceQueue** that items appear
4. The **Performer** robot will automatically pick up and process the items

---

### ✅ Step 6 — Monitor the Jobs

1. Go to Orchestrator → **Monitoring → Jobs**
2. Verify the Dispatcher job ran successfully
3. Check **Monitoring → Queues → InvoiceQueue** for:
   - ✅ **Successful** items
   - ❌ **Failed** items (will be retried automatically)
4. Review **Logs** for detailed activity-level messages

---

## ▶️ How to Run

### Option A — Run Manually from Studio
1. Open the project in **UiPath Studio**
2. Press **F5** or click **Run**
3. The robot will scan the Input folder and add all PDF files to `InvoiceQueue`

### Option B — Run from Orchestrator
1. Go to **Orchestrator → Automations → Processes**
2. Find `Robot 5 PDF Invoice Scraper with REFramework Dispatcher`
3. Click **▶ Start**

### Option C — Automatic via Queue Trigger *(Recommended)*
- Once the Queue Trigger is configured (see above), the robot will run **automatically**

---

## 🛡️ Error Handling

| Scenario | Handling |
|----------|----------|
| File cannot be added to queue | Caught by `TryCatch` — logs error and continues |
| Queue connection failure | Exception logged with file name + error message |
| Empty input folder | `ForEachFileX` simply completes without processing |
| Sensitive data | `Private:*` and `*password*` fields are excluded from logs |

---

## 📝 Notes & Best Practices

- 🔁 This is the **Dispatcher** only — pair it with a **Performer** robot for full end-to-end processing
- 📂 Ensure the **Input folder path** exists before running the robot
- 🔐 The robot is configured as **Unattended** (`isAttended: false`)
- 🪵 All logs are written using UiPath's built-in `LogMessage` activity at `Info` and `Error` levels
- 📌 Queue item **Reference** is set to `CurrentFile.Name` for easy traceability in Orchestrator
- 🔄 **Auto Retry** on the queue is recommended to handle transient failures in the Performer

---

*Generated for Project: **Robot 5 PDF Invoice Scraper with REFramework Dispatcher** | Version 1.0.1*
