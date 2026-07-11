# 📋 Smart Attendance Monitoring System — n8n Automation

Automated attendance tracking system using Google Forms, Google Sheets, n8n, JavaScript, and Telegram Bot notifications.

**Stack:**  
Google Forms · Google Sheets · n8n · Telegram Bot · JavaScript · Google Workspace API


---

# 🎯 Project Overview

## Problem

Traditional attendance monitoring requires manual checking, encoding, and reporting, which can lead to:

- Human errors
- Delayed attendance records
- Difficult report generation
- Lack of real-time monitoring


## Solution

This project automates the complete attendance workflow:

1. Students/employees submit attendance through Google Forms
2. Responses are stored automatically in Google Sheets
3. n8n processes attendance data
4. JavaScript determines Present/Late status
5. Records are stored in an attendance database sheet
6. Telegram sends real-time notifications
7. Daily attendance summaries are generated automatically


---

# ✨ Features

## Core Features

✅ Google Forms attendance submission  
✅ Automatic Google Sheets data collection  
✅ Real-time n8n workflow processing  
✅ Automatic Present/Late detection  
✅ Telegram attendance notifications  
✅ Daily attendance reports  
✅ Automated data organization  


## Automation Features

✅ Trigger-based workflow  
✅ JavaScript data processing  
✅ Conditional branching  
✅ Scheduled reports  
✅ API integration  
✅ Cloud-based workflow automation


---

# 🗺️ System Architecture


```mermaid
flowchart TD

A["👤 Student / Employee"]
--> B["📝 Google Form"]

B --> C["📊 Google Sheets"]

C --> D["⚙️ n8n Automation"]

D --> E["🕒 Timestamp Processing"]

E --> F{"Attendance Time Check"}

F -->|Before 8:00 AM| G["✅ Present"]

F -->|After 8:00 AM| H["⚠️ Late"]

G --> I["📚 Attendance Database"]

H --> I

I --> J["📱 Telegram Notification"]


K["⏰ Daily Scheduler 5PM"]
--> L["📊 Generate Attendance Summary"]

L --> M["📱 Telegram Daily Report"]
````

---

# 🏗️ Project Workflow

## Workflow 1: Real-Time Attendance Monitoring

### Node 1 — Google Sheets Trigger

**Purpose:**

Detect new attendance submissions.

Configuration:

```
Trigger:
Google Sheets Trigger

Event:
Row Added

Polling:
Every 1 minute
```

---

# Node 2 — Data Extraction

**Purpose:**

Clean and map Google Form responses.

Input:

```json
{
"Full Name":"",
"Student ID":"",
"Department":"",
"Attendance Type":"",
"Timestamp":""
}
```

Output:

```json
{
"Name":"",
"StudentID":"",
"Department":"",
"Type":"",
"RawTimestamp":""
}
```

---

# Node 3 — JavaScript Attendance Logic

**Purpose:**

Determine attendance status automatically.

Logic:

```
Before 8:00 AM
=
Present


After 8:00 AM
=
Late
```

Code:

```javascript
const items = $input.all();

const results = [];

for (const item of items) {

const timestamp = new Date(
item.json.RawTimestamp
);


const hour = timestamp.getHours();
const minute = timestamp.getMinutes();


const late =
hour > 8 ||
(hour === 8 && minute > 0);


results.push({

json:{

...item.json,

Status:
late ? "Late":"Present",

ProcessedAt:
new Date().toISOString()

}

});

}

return results;
```

---

# Node 4 — Save Attendance Record

Google Sheets:

```
Sheet:
Attendance Log
```

Data Mapping:

| Field           | Source       |
| --------------- | ------------ |
| Timestamp       | Google Forms |
| Name            | n8n          |
| Student ID      | n8n          |
| Department      | n8n          |
| Attendance Type | n8n          |
| Status          | JavaScript   |
| Processed At    | n8n          |

---

# Node 5 — Attendance Status Branch

IF Node:

Condition:

```javascript
{{$json.Status}}
```

Branches:

```
Present
   |
   |
Telegram Notification


Late
   |
   |
Telegram Warning
```

---

# Node 6A — Present Notification

Telegram Message:

```
✅ Attendance Recorded


👤 Name:
{{Name}}

🎓 Student ID:
{{StudentID}}

🏫 Department:
{{Department}}

🕒 Time:
{{FormattedTime}}

📌 Status:
Present
```

---

# Node 6B — Late Notification

Telegram Message:

```
⚠️ Late Attendance Recorded


👤 Name:
{{Name}}

🎓 Student ID:
{{StudentID}}

🕒 Time:
{{FormattedTime}}

📌 Status:
Late
```

---

# Workflow 2: Daily Attendance Summary

## Schedule Trigger

Configuration:

```
Type:
Schedule Trigger


Time:
5:00 PM


Timezone:
Asia/Manila
```

---

## Generate Report

The workflow:

1. Reads attendance records
2. Counts Present users
3. Counts Late users
4. Creates daily report
5. Sends Telegram summary

Example Output:

```
📊 Daily Attendance Summary

📅 Date:
July 11, 2026


✅ Present:
45


⚠️ Late:
5


📌 Total:
50
```

---

# 📊 Google Sheets Database Design

## Sheet 1: Form Responses

| Column          | Description |
| --------------- | ----------- |
| Timestamp       | Automatic   |
| Full Name       | User input  |
| Student ID      | User input  |
| Department      | User input  |
| Attendance Type | User input  |

---

## Sheet 2: Attendance Log

| Column          | Description       |
| --------------- | ----------------- |
| Timestamp       | Submission time   |
| Name            | Student name      |
| Student ID      | Identification    |
| Department      | Department        |
| Attendance Type | Morning/Afternoon |
| Status          | Present/Late      |
| Processed At    | n8n timestamp     |

---

## Sheet 3: Daily Summary

| Column  | Description      |
| ------- | ---------------- |
| Date    | Report date      |
| Present | Total present    |
| Late    | Total late       |
| Total   | Attendance count |

---

# 🔐 Credentials Required

| Service       | Purpose              |
| ------------- | -------------------- |
| Google OAuth2 | Google Sheets access |
| Telegram API  | Notifications        |
| n8n Instance  | Workflow execution   |

---

# ⚙️ Setup Guide

## 1. Create Google Form

Required fields:

* Full Name
* Student ID
* Department
* Attendance Type

Connect responses to Google Sheets.

---

## 2. Setup Telegram Bot

Steps:

1. Open Telegram
2. Search BotFather
3. Create bot
4. Copy API token
5. Add bot to group
6. Retrieve Chat ID

---

## 3. Configure n8n

Import workflow:

```
Smart-Attendance-Monitoring.json
```

Configure:

* Google Sheets credential
* Telegram credential
* Spreadsheet ID

Activate workflow.

---

# 🧪 Testing Checklist

| Test                  | Expected Result        |
| --------------------- | ---------------------- |
| Submit before 8 AM    | Present                |
| Submit after 8 AM     | Late                   |
| New sheet row created | Workflow triggers      |
| Telegram message sent | Success                |
| 5 PM scheduler runs   | Daily report generated |

---

# 🚀 Future Improvements

| Feature            | Implementation            |
| ------------------ | ------------------------- |
| QR Attendance      | QR-generated Google Forms |
| Face Recognition   | Computer Vision API       |
| GPS Validation     | Location verification     |
| Dashboard          | Looker Studio             |
| Weekly Reports     | Scheduled analytics       |
| Database Migration | PostgreSQL/MySQL          |

---

# 🎓 Skills Applied

## Automation

* n8n Workflow Automation
* Trigger-based systems
* Scheduled workflows

## Programming

* JavaScript
* Data processing
* Conditional logic

## APIs

* Google Sheets API
* Telegram Bot API

## Cloud Tools

* Google Workspace
* n8n Cloud/Self-hosted

---

# 📚 Learning Objectives

Through this project, I learned:

* Building real-world automation systems
* Connecting multiple APIs
* Processing external data
* Creating event-driven workflows
* Designing scalable automation pipelines

---

# 📸 Screenshots

Add:

* n8n workflow screenshot
* Google Form screenshot
* Google Sheet database
* Telegram notification screenshot

---

# 👨‍💻 Author

**Belio C. Sinangote**

BS Information Technology Student
Cebu Technological University (CTU)

GitHub:

[https://github.com/belioautomation](https://github.com/belioautomation)

This project is part of my **30-Day n8n Automation Portfolio**, showcasing practical workflow automation using n8n, APIs, JavaScript, and real-world business solutions.

---

# 📄 License

MIT License

```

This format will now match a **professional automation engineer GitHub portfolio style** and can be reused as the template for your remaining 30-day n8n projects.
```
