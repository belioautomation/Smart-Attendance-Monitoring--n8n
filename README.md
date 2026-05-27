# Smart-Attendance-Monitoring--n8n# 📋 Smart Attendance Monitoring System — Implementation Plan

**Stack:** Google Forms · Google Sheets · n8n · Telegram Bot · JavaScript


---

## 🎯 Project Overview

An automated attendance monitoring system that captures student or employee attendance via Google Forms, stores records in Google Sheets, processes them through n8n workflows, determines attendance status (Present/Late), and delivers real-time Telegram notifications plus daily summaries.

---

## 🗺️ System Architecture

mermaid
flowchart TD
    A["👤 Student / Employee"] -->|Fills out| B["📝 Google Form"]
    B -->|Auto-saves response| C["📊 Google Sheets\n(Raw Responses)"]
    C -->|New row trigger| D["⚙️ n8n Workflow"]
    D --> E["🕐 Extract Timestamp"]
    E --> F{{"⏱️ Is it after 8:00 AM?"}}
    F -->|No| G["✅ Status: Present"]
    F -->|Yes| H["⚠️ Status: Late"]
    G --> I["📊 Update Google Sheets\n(Status Column)"]
    H --> I
    I --> J["📱 Telegram Notification"]
    K["⏰ 5:00 PM Scheduler"] --> L["📈 Generate Daily Summary"]
    L --> M["📱 Telegram Summary Report"]

---

## 📌 Phase 1: Google Forms Setup

### 1.1 Create the Attendance Form

| Field | Type | Required |
|---|---|---|
| Full Name | Short Answer | ✅ Yes |
| Student ID | Short Answer | ✅ Yes |
| Department | Dropdown | ✅ Yes |
| Attendance Type (Morning/Afternoon) | Multiple Choice | ✅ Yes |
| Screenshot / Proof | File Upload | ❌ Optional |

### 1.2 Configuration Steps
1. Go to [Google Forms](https://forms.google.com) → Create New Form
2. Add all fields listed above with proper validation
3. Enable *Collect email addresses* (optional, for verification)
4. Go to *Settings → Responses* → Enable *"Limit to 1 response"* (optional)
5. In the *Responses* tab → Click *Google Sheets icon* → Link to a new spreadsheet
6. Timestamp is automatically added as the first column when linked to Sheets

---

## 📌 Phase 2: Google Sheets Structure

### 2.1 Sheet: Form Responses 1 (Auto-generated)

| Column | Header | Source |
|---|---|---|
| A | Timestamp | Auto (Google Forms) |
| B | Full Name | Form input |
| C | Student ID | Form input |
| D | Department | Form input |
| E | Attendance Type | Form input |
| F | Screenshot | Form input (optional) |

### 2.2 Sheet: Attendance Log (Manual creation — for processed data)

| Column | Header | Populated By |
|---|---|---|
| A | Timestamp | n8n (copied from raw) |
| B | Name | n8n |
| C | Student ID | n8n |
| D | Department | n8n |
| E | Attendance Type | n8n |
| F | Status | n8n (Present / Late) |
| G | Processed At | n8n (auto) |

### 2.3 Sheet: Summary (For daily reports)

| Column | Header |
|---|---|
| A | Date |
| B | Total Present |
| C | Total Late |
| D | Total Absent |
| E | Report Sent At |

---

## 📌 Phase 3: Telegram Bot Setup

### 3.1 Create a Telegram Bot
1. Open Telegram → Search *@BotFather*
2. Send /newbot → Follow prompts → Save the *Bot Token*
3. Create a group or channel for attendance notifications
4. Add your bot to the group as *Admin*
5. Send a test message to the group
6. Retrieve the *Chat ID* via:
   
   https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates
   
7. Note the chat.id value (may be negative for groups, e.g., -1001234567890)

### 3.2 Bot Configuration in n8n
- *Credential Type:* Telegram API
- *Token:* <BOT_TOKEN>
- Store as a named credential: Attendance Bot

---

## 📌 Phase 4: n8n Workflow — Main Attendance Workflow

### 4.1 Workflow: Attendance Monitor

#### Node 1: Google Sheets Trigger
- *Type:* Google Sheets Trigger
- *Credential:* Google OAuth2
- *Spreadsheet:* Attendance System
- *Sheet:* Form Responses 1
- *Event:* Row Added
- *Poll Interval:* Every 1 minute

#### Node 2: Set Node — Extract & Format Data
- *Type:* Set
- *Purpose:* Map raw columns to clean variable names
// Field mappings
Name         → {{ $json["Full Name"] }}
StudentID    → {{ $json["Student ID"] }}
Department   → {{ $json["Department"] }}
Type         → {{ $json["Attendance Type"] }}
RawTimestamp → {{ $json["Timestamp"] }}

#### Node 3: Code Node — Time Check Logic
- *Type:* Code (JavaScript)
- *Purpose:* Parse timestamp and determine Present/Late

// Node: Determine Attendance Status
const items = $input.all();
const results = [];

for (const item of items) {
  const rawTimestamp = item.json.RawTimestamp;
  
  // Parse timestamp from Google Sheets format
  const submissionDate = new Date(rawTimestamp);
  
  const hours = submissionDate.getHours();
  const minutes = submissionDate.getMinutes();
  
  // Cutoff: 8:00 AM = 8 hours, 0 minutes
  const CUTOFF_HOUR = 8;
  const CUTOFF_MINUTE = 0;
  
  const isLate = (hours > CUTOFF_HOUR) || 
                 (hours === CUTOFF_HOUR && minutes > CUTOFF_MINUTE);
  
  const status = isLate ? "Late" : "Present";
  
  // Format time for display (12-hour format)
  const timeString = submissionDate.toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    hour12: true,
    timeZone: 'Asia/Manila' // Adjust to your timezone
  });

  const dateString = submissionDate.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    timeZone: 'Asia/Manila'
  });

  results.push({
    json: {
      ...item.json,
      Status: status,
      FormattedTime: timeString,
      FormattedDate: dateString,
      IsLate: isLate
    }
  });
}

return results;

#### Node 4: Google Sheets — Append to Attendance Log
- *Type:* Google Sheets (Append)
- *Spreadsheet:* Attendance System
- *Sheet:* Attendance Log
- *Columns to map:*
  - Timestamp → {{ $json.RawTimestamp }}
  - Name → {{ $json.Name }}
  - Student ID → {{ $json.StudentID }}
  - Department → {{ $json.Department }}
  - Attendance Type → {{ $json.Type }}
  - Status → {{ $json.Status }}
  - Processed At → {{ $now.toISO() }}

#### Node 5: IF Node — Branch by Status
- *Type:* IF
- *Condition:* {{ $json.Status }} equals Late
- *True branch* → Late notification
- *False branch* → Present notification

#### Node 6a: Telegram — Present Notification
- *Type:* Telegram
- *Credential:* Attendance Bot
- *Chat ID:* {{ $env.TELEGRAM_CHAT_ID }}
- *Message:*
✅ *Attendance Recorded*

👤 *Name:* {{ $json.Name }}
🎓 *Student ID:* {{ $json.StudentID }}
🏫 *Department:* {{ $json.Department }}
🕒 *Time:* {{ $json.FormattedTime }}
📅 *Date:* {{ $json.FormattedDate }}
📌 *Status:* ✅ Present
📋 *Type:* {{ $json.Type }}

#### Node 6b: Telegram — Late Notification
- *Type:* Telegram
- *Credential:* Attendance Bot
- *Chat ID:* {{ $env.TELEGRAM_CHAT_ID }}
- *Message:*
⚠️ *Late Attendance Recorded*

👤 *Name:* {{ $json.Name }}
🎓 *Student ID:* {{ $json.StudentID }}
🏫 *Department:* {{ $json.Department }}
🕒 *Time:* {{ $json.FormattedTime }}
📅 *Date:* {{ $json.FormattedDate }}
📌 *Status:* ⚠️ Late
📋 *Type:* {{ $json.Type }}

---

## 📌 Phase 5: n8n Workflow — Daily Summary (5:00 PM)

### 5.1 Workflow: Daily Attendance Summary

#### Node 1: Schedule Trigger
- *Type:* Schedule Trigger
- *Mode:* Every Day
- *Time:* 17:00 (5:00 PM)
- *Timezone:* Asia/Manila

#### Node 2: Google Sheets — Read Today's Log
- *Type:* Google Sheets (Read)
- *Sheet:* Attendance Log
- *Filter:* Rows where date matches today

#### Node 3: Code Node — Generate Summary
const items = $input.all();
const today = new Date().toLocaleDateString('en-US', {
  timeZone: 'Asia/Manila'
});

let present = 0;
let late = 0;

for (const item of items) {
  const rowDate = new Date(item.json.Timestamp).toLocaleDateString('en-US', {
    timeZone: 'Asia/Manila'
  });
  
  if (rowDate === today) {
    if (item.json.Status === 'Present') present++;
    else if (item.json.Status === 'Late') late++;
  }
}

return [{
  json: {
    Date: today,
    Present: present,
    Late: late,
    Total: present + late
  }
}];

#### Node 4: Google Sheets — Append to Summary Sheet
- *Sheet:* Summary
- Map: Date, Present, Late, Total, Report Sent At

#### Node 5: Telegram — Send Daily Summary
📊 *Daily Attendance Summary*
📅 Date: {{ $json.Date }}
━━━━━━━━━━━━━━━━━━
✅ Present:  {{ $json.Present }} students
⚠️ Late:     {{ $json.Late }} students
📊 Total:    {{ $json.Total }} recorded
━━━━━━━━━━━━━━━━━━
📌 Report generated at 5:00 PM


---

## 📌 Phase 6: n8n Environment Variables

Set these in *n8n Settings → Variables:*

| Variable | Value |
|---|---|
| TELEGRAM_CHAT_ID | -1001234567890 (your group ID) |
| CUTOFF_HOUR | 8 |
| CUTOFF_MINUTE | 0 |
| TIMEZONE |Asia/Manila` |

---

## 📌 Phase 7: n8n Credentials Setup

### 7.1 Google OAuth2
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a project → Enable *Google Sheets API* and *Google Drive API*
3. Create OAuth 2.0 credentials (Desktop App)
4. Download JSON → Enter Client ID & Secret in n8n
5. Authenticate via the n8n credential popup

### 7.2 Telegram Bot
1. In n8n → Credentials → New → Telegram API
2. Enter your Bot Token
3. Test the credential

---

## 📌 Phase 8: Testing Checklist

| Test Case | Expected Result |
|---|---|
| Submit form at 7:00 AM | Status = ✅ Present |
| Submit form at 8:01 AM | Status = ⚠️ Late |
| Submit form at 8:00 AM exactly | Status = ✅ Present |
| Daily trigger at 5:00 PM | Summary sent to Telegram |
| Google Sheets row appended | Attendance Log updated |
| Telegram message received | Correct format with name/time/status |

---

## 🚀 Advanced Features (Phase 9 — Future Enhancements)

| Feature | Implementation Approach |
|---|---|
| *QR Code Attendance* | Generate QR codes per student linking to pre-filled Google Form URLs |
| *GPS Validation* | Add a location field in the form; validate coordinates in n8n Code node |
| *Dashboard Analytics* | Use Google Sheets charts or integrate with Grafana/Metabase |
| *Absence Detection* | Run a 9:00 AM scheduler to flag students with no submission |
| *Weekly Reports* | Extend the summary workflow with a weekly aggregation loop |
| *Multi-Department Support* | Use IF branching in n8n per department with separate Telegram channels |

---

## 📅 Development Timeline

| Phase | Task | Estimated Time |
|---|---|---|
| 1 | Google Form creation & Sheets linking | 30 min |
| 2 | Google Sheets structure & formatting | 30 min |
| 3 | Telegram bot setup & Chat ID retrieval | 20 min |
| 4 | n8n credential setup (Google + Telegram) | 30 min |
| 5 | Build main attendance workflow in n8n | 1–2 hours |
| 6 | Build daily summary workflow in n8n | 45 min |
| 7 | Testing all scenarios | 1 hour |
| 8 | Documentation & deployment | 30 min |
| *Total* | |*~5–6 hours** |

---

## 🎓 Skills Applied

- ✅ *API Integration* — Google Sheets API, Telegram Bot API
- ✅ *Workflow Automation* — n8n trigger-based flows
- ✅ *Conditional Logic* — IF nodes + JavaScript time comparison
- ✅ *Google Workspace Integration* — Forms → Sheets pipeline
- ✅ *Telegram Bot Development* — BotFather setup, message formatting
- ✅ *JavaScript Functions* — Custom Code nodes for time parsing

---

[!IMPORTANT]
Make sure your Google Cloud project has both **Google Sheets API** and **Google Drive API** enabled before connecting n8n credentials. Missing either will cause authentication failures.


[!TIP]
Test your Telegram Bot Token using the `getUpdates` endpoint in your browser before adding it to n8n. This confirms the bot is active and the Chat ID is correct.
