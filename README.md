# 📋 Smart Attendance Monitoring System — n8n Automation

![n8n](https://img.shields.io/badge/n8n-Automation-orange)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-green)
![Telegram](https://img.shields.io/badge/Notifications-Telegram-blue)
![JavaScript](https://img.shields.io/badge/Code-JavaScript-yellow)
![License](https://img.shields.io/badge/license-MIT-green)

An automated attendance monitoring system built using **n8n**, **Google Forms**, **Google Sheets**, **JavaScript**, and **Telegram Bot API**.

This workflow automatically collects attendance submissions, processes attendance records, determines Present/Late status using JavaScript logic, stores attendance data, sends real-time Telegram notifications, and generates automated daily attendance reports.

**Stack:**  
n8n · Google Forms · Google Sheets · Telegram Bot API · JavaScript · Google Workspace API · Automation Workflow


---

# 🎯 Project Overview


## Problem

Traditional attendance monitoring requires manual checking, encoding, and reporting.

Common challenges include:

- Manual attendance recording
- Human errors during data entry
- Delayed attendance reports
- Difficulty tracking attendance patterns
- Lack of real-time monitoring


For schools and organizations, managing daily attendance efficiently can become time-consuming.


---

## Solution

This project creates an automated attendance management system by:


1. Collecting attendance through Google Forms
2. Recording responses in Google Sheets
3. Detecting new attendance entries using n8n
4. Processing attendance data automatically
5. Determining Present/Late status
6. Saving structured attendance records
7. Sending Telegram notifications
8. Generating daily attendance summaries


The workflow acts as a digital attendance assistant that eliminates repetitive manual tasks and provides real-time monitoring.


---

# ✨ Features


## Attendance Processing

✅ Google Forms attendance submission  
✅ Automatic Google Sheets data collection  
✅ Real-time attendance processing  
✅ Present/Late detection  
✅ Structured attendance database  


## Automation

✅ Event-driven workflow  
✅ JavaScript data processing  
✅ Conditional workflow logic  
✅ Scheduled attendance reports  
✅ Automated notifications  


## Reporting

✅ Daily attendance summaries  
✅ Present/Late statistics  
✅ Telegram report delivery  
✅ Attendance history tracking  


## Integration

✅ Google Workspace integration  
✅ Telegram Bot notifications  
✅ n8n workflow automation  


---

# 🗺️ System Architecture


```mermaid
flowchart TD

A["👤 Student / Employee"]

--> B["📝 Google Forms"]

--> C["📊 Google Sheets"]

--> D["⚙️ n8n Automation"]

--> E["🕒 Timestamp Processing"]

--> F{"Attendance Time Check"}

F -->|Before 8:00 AM| G["✅ Present"]

F -->|After 8:00 AM| H["⚠️ Late"]

G --> I["📚 Attendance Database"]

H --> I

I --> J["📱 Telegram Notification"]


K["⏰ Schedule Trigger 5PM"]

--> L["📊 Generate Daily Summary"]

--> M["📱 Telegram Report"]
````

---

# 🏗️ Workflow Implementation

# Workflow 1: Real-Time Attendance Monitoring

## Node 1 — Google Sheets Trigger

### Purpose

Detect new attendance submissions from Google Forms responses.

Configuration:

```text
Trigger:

Google Sheets Trigger


Event:

Row Added


Polling:

Every Minute
```

Captured Information:

| Field           | Description         |
| --------------- | ------------------- |
| Timestamp       | Attendance time     |
| Full Name       | User identity       |
| Student ID      | Identification      |
| Department      | Organization/class  |
| Attendance Type | Attendance category |

---

# Node 2 — Data Extraction

### Purpose

Clean and standardize incoming attendance data before processing.

Input Example:

```json
{
"Full Name":
"John Smith",

"Student ID":
"20260001",

"Department":
"BSIT",

"Timestamp":
"2026-07-11 07:45:00"
}
```

Output:

```json
{
"Name":
"John Smith",

"StudentID":
"20260001",

"Department":
"BSIT",

"RawTimestamp":
"2026-07-11 07:45:00"
}
```

---

# Node 3 — JavaScript Attendance Logic

### Purpose

Automatically determine attendance status based on submission time.

Logic:

```text
Before 8:00 AM

=

Present


After 8:00 AM

=

Late
```

Processing:

```javascript
const timestamp = new Date(
item.json.RawTimestamp
);

const hour = timestamp.getHours();
const minute = timestamp.getMinutes();

const late =
hour > 8 ||
(hour === 8 && minute > 0);

return {

json: {

...item.json,

Status:
late ? "Late" : "Present",

ProcessedAt:
new Date().toISOString()

}

};
```

Output:

```json
{
"Status":
"Present",

"ProcessedAt":
"2026-07-11T00:45:00Z"
}
```

---

# Node 4 — Save Attendance Record

### Purpose

Store processed attendance information into the attendance database.

Google Sheets:

```text
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

# Node 5 — IF Node (Attendance Status Branch)

### Purpose

Separate attendance notifications based on status.

Condition:

```javascript
{{$json.Status}}
```

Branches:

```text
Present

↓

Attendance Confirmation


Late

↓

Late Attendance Warning
```

---

# Node 6A — Present Notification

Telegram Message:

```text
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

```text
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

## Node 1 — Schedule Trigger

### Purpose

Automatically generate attendance reports at the end of the day.

Configuration:

```text
Trigger:

Schedule Trigger


Time:

5:00 PM


Timezone:

Asia/Manila
```

---

## Node 2 — Generate Attendance Report

The workflow:

1. Reads attendance records
2. Counts Present users
3. Counts Late users
4. Creates daily summary
5. Sends Telegram report

Example Output:

```text
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

| Column          | Description               |
| --------------- | ------------------------- |
| Timestamp       | Automatic submission time |
| Full Name       | User input                |
| Student ID      | User input                |
| Department      | User input                |
| Attendance Type | User input                |

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

| Service          | Purpose              |
| ---------------- | -------------------- |
| Google OAuth2    | Google Sheets access |
| Telegram Bot API | Notifications        |
| n8n Instance     | Workflow execution   |

---

# ⚙️ Setup Guide

## 1. Create Google Form

Required Fields:

```text
Full Name

Student ID

Department

Attendance Type
```

Connect responses to Google Sheets.

---

## 2. Setup Telegram Bot

Steps:

1. Open Telegram
2. Search BotFather
3. Create a bot
4. Copy API token
5. Add Telegram credentials in n8n
6. Configure Chat ID

---

## 3. Configure n8n Workflow

Import:

```text
Smart-Attendance-Monitoring.json
```

Configure:

* Google Sheets credential
* Telegram credential
* Spreadsheet ID

Activate workflow.

---

# 🧪 Testing Checklist

| Test Case              | Expected Result       |
| ---------------------- | --------------------- |
| Submit before 8 AM     | Present status        |
| Submit after 8 AM      | Late status           |
| New sheet row added    | Workflow starts       |
| JavaScript executes    | Status generated      |
| Telegram sends message | Notification received |
| 5 PM scheduler runs    | Daily report created  |

---

# 📁 Repository Structure

```text
Smart-Attendance-Monitoring-System/

│
├── README.md
│
├── workflow.json
│
├── screenshots/
│   │
│   ├── workflow.png
│   ├── google-form.png
│   ├── google-sheets.png
│   ├── javascript-node.png
│   ├── attendance-status.png
│   ├── telegram-notification.png
│   └── workflow-execution.png
│
└── LICENSE
```

---

# 📸 Screenshots

Recommended screenshots:

* Complete workflow
* Google Form attendance submission
* Google Sheets responses
* JavaScript attendance processing
* Present/Late classification
* Telegram notification
* Daily summary report
* Workflow execution

---

# 🚀 Future Improvements

| Feature               | Implementation                |
| --------------------- | ----------------------------- |
| QR Attendance         | QR-generated attendance forms |
| Face Recognition      | Computer Vision API           |
| GPS Validation        | Location verification         |
| Dashboard             | Looker Studio analytics       |
| Weekly Reports        | Scheduled summaries           |
| Database Migration    | PostgreSQL/MySQL              |
| Biometric Integration | Hardware attendance devices   |

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
* JSON transformation

## APIs

* Google Sheets API
* Telegram Bot API

## Cloud Tools

* Google Workspace
* n8n Self-hosted/Cloud

## Business Automation

* Attendance management
* Workflow optimization
* Automated reporting

---

# 📚 Learning Objectives

This project demonstrates:

* Building real-world automation systems
* Connecting multiple APIs
* Processing external data automatically
* Creating event-driven workflows
* Designing scalable automation pipelines

---

# 🙌 Acknowledgements

* n8n
* Google Workspace
* Google Sheets API
* Telegram Bot API

---

# 👨‍💻 Author

**Belio C. Sinangote**

BS Information Technology Student
Cebu Technological University (CTU)

GitHub:

[https://github.com/belioautomation](https://github.com/belioautomation)

This project is part of my **30-Day n8n Automation Portfolio**, showcasing practical workflow automation using **n8n, APIs, JavaScript, and real-world business solutions**.

---

# 📄 License

MIT License

```
```
