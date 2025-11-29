# RR TMS - Complete Workflow Documentation

## Document Version
**Version:** 1.0  
**Last Updated:** November 29, 2025  
**Author:** System Documentation

---

## Table of Contents
1. [System Workflows Overview](#1-system-workflows-overview)
2. [Task Lifecycle Workflow](#2-task-lifecycle-workflow)
3. [Ticket Lifecycle Workflow](#3-ticket-lifecycle-workflow)
4. [Form Creation & Public Links](#4-form-creation--public-links)
5. [Recurring Tasks Workflow](#5-recurring-tasks-workflow)
6. [Project Management Workflow](#6-project-management-workflow)
7. [Audit Logs & Tracking](#7-audit-logs--tracking)
8. [Notification System](#8-notification-system)
9. [Complete Feature Summary](#9-complete-feature-summary)

---

## 1. System Workflows Overview

RR TMS supports multiple interconnected workflows that enable comprehensive task and ticket management across organizations.

### Core Workflows

```
┌────────────────────────────────────────────────────────────┐
│                  RR TMS WORKFLOW ECOSYSTEM                 │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐    ┌──────────────┐    ┌─────────────┐ │
│  │   PROJECT    │───▶│     TASK     │───▶│ CHECKLIST   │ │
│  │  MANAGEMENT  │    │  MANAGEMENT  │    │  TRACKING   │ │
│  └──────────────┘    └──────────────┘    └─────────────┘ │
│         │                    │                    │        │
│         │                    ▼                    │        │
│         │            ┌──────────────┐             │        │
│         │            │ ASSIGNMENTS  │             │        │
│         │            │ (User/Dept/  │             │        │
│         │            │  Store)      │             │        │
│         │            └──────────────┘             │        │
│         │                    │                    │        │
│         ▼                    ▼                    ▼        │
│  ┌──────────────────────────────────────────────────────┐ │
│  │           NOTIFICATION & AUDIT SYSTEM                │ │
│  └──────────────────────────────────────────────────────┘ │
│         │                    │                    │        │
│  ┌──────▼──────┐    ┌───────▼──────┐    ┌───────▼──────┐ │
│  │   TICKET    │    │    FORMS     │    │  RECURRING   │ │
│  │  MANAGEMENT │    │  & PUBLIC    │    │    TASKS     │ │
│  │             │    │    LINKS     │    │  (AUTO-GEN)  │ │
│  └─────────────┘    └──────────────┘    └──────────────┘ │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 2. Task Lifecycle Workflow

### 2.1 Task Creation by ClientAdmin

**Complete Task Creation Flow:**

```
ClientAdmin → Create Project → Create Task → Define Details →
Select Assignment Type → Add Subtasks (Optional) → Add Checklists →
Save → Auto-Assign Users → Send Notifications → Task CREATED
```

#### Task States Diagram:
```
CREATED → ASSIGNED → ACKNOWLEDGED → IN_PROGRESS → COMPLETED → CLOSED
   │          │            │             │             │
   ├──────────┴────────────┴─────────────┴─────────────┴──→ CANCELLED
   └────────────────────────────────────────────────────→ OVERDUE (auto)
```

#### Detailed Task Execution by User:

**Phase 1: Task Assignment & Notification**
- User receives push notification
- Task appears in "My Tasks" dashboard
- Status: ASSIGNED

**Phase 2: Task Acknowledgment**
- User opens task details
- Reviews description, due date, priority
- Clicks "Acknowledge Task" button
- Status changes: ASSIGNED → ACKNOWLEDGED
- Notification sent to task creator

**Phase 3: Task Start**
- User clicks "Start Task" button OR completes first checklist
- Timer starts for SLA tracking
- Status changes: ACKNOWLEDGED → IN_PROGRESS
- Checklists become editable

**Phase 4: Task Execution**
- User completes task-level checklists
- User completes subtasks (if any)
- User completes subtask checklists
- Progress tracked: X% completed
- User can add comments, attachments

**Phase 5: Completion Validation**
- System validates:
  - All task checklists completed ✓
  - All subtasks completed ✓
  - All subtask checklists completed ✓
- "Complete Task" button becomes enabled

**Phase 6: Task Completion**
- User clicks "Complete Task"
- Popup asks for confirmation + optional notes
- Status changes: IN_PROGRESS → COMPLETED
- Task locked (no further edits allowed)
- Notifications sent to creator/manager

**Phase 7: Task Lock & Redirect**
- If user tries to access completed task
- Popup: "Task completed. No changes allowed."
- Redirects to dashboard
- Task appears in "Completed Tasks" list

---

### 2.2 Assignment Types

| Type | Description | Auto-Assignment | Example |
|------|-------------|----------------|---------|
| **Individual** | Specific users selected | Creates 1 assignment per user | "Assign to John, Jane" |
| **Department** | All users in department(s) | Auto-assigns all dept users | "Assign to Sales Dept" |
| **Store** | All users in branch/store | Auto-assigns all store users | "Assign to Mumbai Branch" |
| **Multiple/Mixed** | Combination of above | Auto-assigns from all sources | "Sales Dept + John + Delhi Store" |


---

## 3. Ticket Lifecycle Workflow

### 3.1 Complete Ticket Flow

```
User Raises Ticket → Ticket OPEN → Auto-Assign to Anchor →
Anchor Investigates → IN_PROGRESS →

   [Path A: Direct Resolution]
   → Mark RESOLVED → User Verifies → CLOSED

   [Path B: Convert to Task]
   → Create Task from Ticket → Task Lifecycle Begins → 
   Task Completed → Update Ticket RESOLVED → CLOSED

   [Path C: Escalation]
   → ESCALATED → Manager Assigns to Specialist → Resolution → CLOSED
```

### 3.2 Ticket States

| State | Description | Next Actions | Triggered By |
|-------|-------------|--------------|--------------|
| **OPEN** | Newly created ticket | Assign to Anchor | User creates ticket |
| **ASSIGNED** | Anchor assigned | Start investigation | System/Manager |
| **IN_PROGRESS** | Being investigated | Resolve/Convert/Escalate | Anchor starts work |
| **RESOLVED** | Issue fixed | Close or Reopen | Anchor resolves |
| **CLOSED** | Verified & closed | Can reopen if needed | User/Manager confirms |
| **CONVERTED_TO_TASK** | Task created from ticket | Track task completion | Anchor converts |
| **CANCELLED** | Ticket cancelled | None | Manager/User cancels |

### 3.3 Ticket-to-Task Conversion

**Scenario:** Complex issue requiring scheduled work

**Process:**
1. Anchor investigates ticket
2. Realizes it needs multiple steps/resources
3. Clicks "Convert to Task"
4. Form auto-fills:
   - Title from ticket
   - Description from ticket + resolution notes
   - Priority inherited
5. Anchor selects:
   - Project to link
   - Users/departments to assign
   - Due date
   - Subtasks (if needed)
6. Task created with `task_type = 'ticket_resolution'`
7. Task linked: `task.linked_ticket_id = ticket.id`
8. Ticket status: CONVERTED_TO_TASK
9. Task lifecycle begins
10. When task completed → Ticket status: RESOLVED

**Benefits:**
- Complex issues properly tracked
- Multiple team members can work
- Progress visible to all stakeholders
- Ticket and task histories linked

---

## 4. Form Creation & Public Links

### 4.1 Form Builder Workflow

**Complete Form Creation Process:**

```
ClientAdmin → Forms Tab → Create Form Template → Design Form Fields →
Save Template (INACTIVE) → Review → Activate Form →
System Creates Dynamic DB Table → Generate Public Link →
Share Link (Email/QR/Embed) → Users Submit → Data Stored →
Link to Tasks/Tickets
```

### 4.2 Form Field Types Available

| Field Type | Use Case | Validation | Data Type |
|------------|----------|------------|-----------|
| Text Input | Names, short text | Max length, regex | VARCHAR |
| Text Area | Comments, descriptions | Max length | TEXT |
| Number | Scores, quantities | Min/max, step | INT/DECIMAL |
| Email | Email addresses | Email format | VARCHAR |
| Phone | Phone numbers | Phone format | VARCHAR |
| Date | Dates | Date range | DATE |
| Time | Time values | Time format | TIME |
| DateTime | Date + time | Datetime format | DATETIME |
| Dropdown | Single selection | Options list | VARCHAR/ENUM |
| Radio | Single choice | Options list | VARCHAR/ENUM |
| Checkboxes | Multiple selection | Options list | JSON |
| File Upload | Photos, documents | File type, size | JSON (paths) |
| Signature | Digital signature | Required capture | VARCHAR (path) |
| Location | GPS coordinates | Lat/long validation | JSON |


```



### 4.3 Public Link Features

**Public Link Capabilities:**
- ✅ Anonymous submissions (no login)
- ✅ Email verification (optional)
- ✅ Submission limits (max submissions)
- ✅ Expiry dates (time-limited links)
- ✅ QR code generation
- ✅ Embed in websites (iframe)
- ✅ Mobile-responsive
- ✅ Offline support (PWA)
- ✅ Multi-language support
- ✅ Custom branding

**Security:**
- Token-based authentication (UUID)
- Rate limiting (prevent spam)
- CAPTCHA (optional)
- IP tracking
- Duplicate submission prevention

---

## 5. Recurring Tasks Workflow

### 5.1 Recurring Task Creation

**Setup Process:**

```
ClientAdmin → Recurring Tasks → Create → Define Template →
Set Recurrence Pattern → Configure Assignment Rules →
Set Start/End Dates → Add Checklist Template → Save →
Cron Job Activated → Auto-Generate Tasks Daily
```

### 5.2 Recurrence Patterns

| Pattern | Description | Example | Cron Expression |
|---------|-------------|---------|-----------------|
| **Daily** | Every day | Daily opening checklist | `0 9 * * *` |
| **Weekdays** | Monday-Friday | Weekday operations | `0 9 * * 1-5` |
| **Weekly** | Specific days | Every Monday & Friday | `0 9 * * 1,5` |
| **Monthly** | Specific date | 1st of every month | `0 9 1 * *` |
| **Custom Cron** | Custom pattern | Every 2 hours | `0 */2 * * *` |

### 5.3 Automated Task Generation (Cron Job)


**Execution:**
```yaml
Schedule: Runs every hour (or as configured)

Process:
  1. Fetch all active recurring_tasks WHERE is_active = 1
  2. For each recurring task:
     a. Check if pattern matches current date/time
     b. Check if task already generated today
     c. If not generated:
        - Create new task from template
        - Apply assignment rules (role-based/user-based)
        - Add checklists from template
        - Set due date/time
        - Send notifications
        - Mark as generated for today
  3. Log all generated tasks
  4. Update recurring_tasks.last_generated_at

Example:
  Recurring Task: "Daily Store Opening"
  Pattern: "0 9 * * 1-5" (9 AM, Monday-Friday)
  
  On Dec 1, 2025 at 9:00 AM:
    → Cron job runs
    → Pattern matches (Monday, 9 AM)
    → Creates task: "Daily Store Opening - Dec 1, 2025"
    → Assigns to: BranchManager at each store
    → Due: Dec 1, 2025 11:00 AM
    → Notifications sent
    → recurring_tasks.last_generated_at = "2025-12-01 09:00:00"
  
  On Dec 1, 2025 at 10:00 AM:
    → Cron job runs
    → Task already generated today → Skip
```

**Assignment Rules:**
- **Role-Based**: Assigns to users with specific role at each scope (e.g., BranchManager at each store)
- **User-Based**: Assigns to specific users
- **Department-Based**: Assigns to all users in department
- **Rotation**: Assigns to users in rotation (if configured)

---

## 6. Project Management Workflow

### 6.1 Project Creation & Scope

**Project Setup:**

```
ClientAdmin → Projects Tab → Create Project → Define Scope →
Add Team Members → Set Timeline → Add Tasks → Track Progress
```

**Project Scope Types:**

| Scope | Description | Example |
|-------|-------------|---------|
| **Client-Wide** | All stores, all departments | "Annual Compliance Audit" |
| **Multi-Store** | Specific stores | "Mumbai & Delhi Stores" |
| **Multi-Department** | Specific departments | "Sales & Marketing" |
| **Store-Specific** | Single store only | "Mumbai Store Q4 Tasks" |
| **Department-Specific** | Single department | "HR Onboarding" |

**Project Database Structure:**
```yaml
projects:
  - id: 10
  - name: "Q4 Hygiene Compliance"
  - description: "..."
  - status: active
  - start_date: "2025-10-01"
  - end_date: "2025-12-31"
  - created_by: ClientAdmin

project_clients:
  - (project_id: 10, client_id: 1)

project_stores:
  - (project_id: 10, store_id: 1)
  - (project_id: 10, store_id: 2)
  - (project_id: 10, store_id: 3)

project_departments:
  - (project_id: 10, department_id: 5)

project_users:
  - (project_id: 10, user_id: 15, role: 'member')
  - (project_id: 10, user_id: 20, role: 'manager')
```

### 6.2 Project Tracking & Analytics

**Auto-Calculated Metrics:**
- Total tasks created
- Tasks completed
- Completion percentage
- Average completion time
- Overdue tasks count
- Tasks by priority
- Tasks by status
- User-wise completion rate

**Stored in:** `project_analytics` table (updated via triggers)

---

## 7. Audit Logs & Tracking

### 7.1 Comprehensive Audit Trail

**What Gets Logged:**
- ✅ All CRUD operations (Create, Read, Update, Delete)
- ✅ User logins/logouts
- ✅ Permission changes
- ✅ Role assignments
- ✅ Task status changes
- ✅ Ticket status changes
- ✅ Form submissions
- ✅ File uploads
- ✅ Configuration changes
- ✅ API access
- ✅ Failed login attempts

**Audit Log Structure:**
```yaml
audit_logs table:
  - id: Unique ID
  - user_id: Who performed action
  - action: CREATE | READ | UPDATE | DELETE | LOGIN | LOGOUT | VIEW | LIST
  - entity_type: task | ticket | user | project | client | etc.
  - entity_id: ID of entity
  - old_data: JSON of previous state
  - new_data: JSON of new state
  - ip_address: User's IP
  - user_agent: Browser/device info
  - device_type: mobile | desktop | tablet
  - location: Geo-location (if available)
  - created_at: Timestamp

Example Entry:
{
  "id": 1250,
  "user_id": 15,
  "action": "UPDATE",
  "entity_type": "task",
  "entity_id": 100,
  "old_data": {"status": "in_progress", "priority": "medium"},
  "new_data": {"status": "completed", "priority": "high"},
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0...",
  "device_type": "mobile",
  "location": "Mumbai, India",
  "created_at": "2025-11-29 14:30:00"
}
```

### 7.2 Audit Log Viewing & Filtering

**SuperAdmin Audit Dashboard:**

**Filters:**
- Date range
- User (who performed action)
- Action type (CREATE/UPDATE/DELETE/etc.)
- Entity type (task/ticket/user/etc.)
- Search by entity ID or keyword

**Features:**
- Export to CSV/Excel
- Real-time updates
- Drill-down to view details
- Compare old vs new data
- Filter by IP/location

**Use Cases:**
- Security audits
- Compliance reporting
- Troubleshooting issues
- Performance analysis
- User activity tracking

---

## 8. Notification System

### 8.1 Notification Types

| Type | Delivery Channel | Use Case |
|------|-----------------|----------|
| **Push Notification** | Firebase Cloud Messaging | Real-time mobile/web alerts |
| **Email** | SMTP | Detailed notifications, reports |
| **In-App** | Database + WebSocket | Dashboard notifications |
| **SMS** | Twilio/SMS Gateway | Critical alerts |

### 8.2 Notification Events

**Task-Related:**
- Task assigned to you
- Task due soon (configurable)
- Task overdue
- Task completed by assignee
- Comment added to your task
- Task status changed
- Subtask completed
- Checklist completed

**Ticket-Related:**
- Ticket assigned to you
- Ticket escalated
- Ticket resolved
- Ticket reopened
- Comment added
- Status changed

**Form-Related:**
- New form submission
- Form linked to your task
- Form submission requires action

**System-Related:**
- New user created
- Permission changed
- Role assigned
- Password reset request
- Login from new device

### 8.3 Notification Preferences

**User Settings:**
- Choose channels (push/email/SMS)
- Set quiet hours
- Priority filtering (only high/critical)
- Frequency control (real-time/hourly digest/daily digest)
- Entity-specific (tasks only, tickets only, etc.)

---

## 9. Complete Feature Summary

### 9.1 All Implemented Features

#### **Core Task Management:**
- ✅ Task creation with multiple assignment types
- ✅ Subtasks with nested checklists
- ✅ Task-level checklists
- ✅ Auto-assignment to users/departments/stores
- ✅ Task lifecycle (10 states)
- ✅ SLA tracking & breach detection
- ✅ Completion validation
- ✅ Task locking after completion
- ✅ Comments & mentions
- ✅ File attachments
- ✅ Progress tracking
- ✅ Due date reminders

#### **Ticket Management:**
- ✅ Ticket raising by users
- ✅ Auto-assignment to Anchors
- ✅ Ticket lifecycle (8 states)
- ✅ Investigation workflow
- ✅ Direct resolution
- ✅ Convert ticket to task
- ✅ Escalation to managers
- ✅ User verification & closure
- ✅ Ticket reopening
- ✅ Resolution time tracking
- ✅ Customer feedback/ratings

#### **Form System:**
- ✅ Dynamic form builder
- ✅ 13+ field types
- ✅ Form templates
- ✅ Dynamic database table creation
- ✅ Public link generation
- ✅ Anonymous submissions
- ✅ QR code generation
- ✅ Form submissions linked to tasks/tickets
- ✅ Offline support (PWA)
- ✅ Custom validation rules
- ✅ File uploads in forms
- ✅ Digital signature capture

#### **Recurring Tasks:**
- ✅ Recurring task templates
- ✅ Multiple recurrence patterns (daily/weekly/monthly/custom)
- ✅ Cron-based auto-generation
- ✅ Role-based auto-assignment
- ✅ Checklist templates
- ✅ Start/end date control
- ✅ Pattern testing
- ✅ Generation history

#### **Project Management:**
- ✅ Multi-scope projects
- ✅ Client/store/department scoping
- ✅ Project timelines
- ✅ Team member assignment
- ✅ Project analytics
- ✅ Progress tracking
- ✅ Task distribution reports

#### **User & Permission Management:**
- ✅ Dynamic role creation
- ✅ 8 granular permissions (R/W/E/D/A/X/S/M)
- ✅ 13+ dashboard tabs
- ✅ Tab-level permission control
- ✅ Hierarchical access (SuperAdmin → User)
- ✅ Scope-based data filtering
- ✅ Multi-tenant isolation
- ✅ Role hierarchy levels

#### **Audit & Compliance:**
- ✅ Comprehensive audit logging
- ✅ All CRUD operations tracked
- ✅ Old/new data comparison
- ✅ IP & device tracking
- ✅ Geo-location logging
- ✅ Export audit logs (CSV/Excel)
- ✅ Searchable audit trail
- ✅ Compliance reporting

#### **Notifications:**
- ✅ Push notifications (Firebase)
- ✅ Email notifications
- ✅ In-app notifications
- ✅ SMS notifications (optional)
- ✅ Notification preferences
- ✅ Real-time/digest modes
- ✅ Priority filtering

#### **Analytics & Reporting:**
- ✅ Task completion metrics
- ✅ User performance tracking
- ✅ SLA compliance reports
- ✅ Project progress
- ✅ Ticket resolution times
- ✅ Department-wise statistics
- ✅ Store-wise analytics
- ✅ Custom date range reports
- ✅ Export to Excel/CSV

---

### 9.2 Missing/Future Features

#### **Identified Gaps:**

**1. Comments System:**
- ❌ Comments API not fully implemented
- ❌ @mentions not triggering notifications
- ❌ Comment threading/replies

**2. Advanced Analytics:**
- ❌ Custom dashboard widgets
- ❌ Predictive analytics
- ❌ Performance trends

**3. Mobile App:**
- ❌ Native mobile app (iOS/Android)
- Currently: PWA (Progressive Web App)

**4. Integrations:**
- ❌ Third-party integrations (Slack, Teams, etc.)
- ❌ Calendar integration
- ❌ External API webhooks

**5. Advanced Reporting:**
- ❌ Custom report builder
- ❌ Scheduled reports
- ❌ Report templates

---

## 10. Key Workflows Summary

### Task Workflow (End-to-End):
```
CREATE → ASSIGN → NOTIFY → ACKNOWLEDGE → START → 
EXECUTE → COMPLETE CHECKLISTS → COMPLETE SUBTASKS →
VALIDATE → COMPLETE → LOCK → NOTIFY → CLOSED
```
**Duration:** Hours to Days  
**Participants:** Creator, Assignees, Managers  
**Completion Criteria:** All checklists + subtasks done

---

### Ticket Workflow (End-to-End):
```
RAISE → AUTO-ASSIGN → NOTIFY → INVESTIGATE → 
[Direct Resolution OR Convert to Task OR Escalate] →
RESOLVE → USER VERIFY → CLOSE
```
**Duration:** Minutes to Days  
**Participants:** User (raiser), Anchor (resolver), Managers  
**Completion Criteria:** Issue resolved & verified

---

### Form Workflow (End-to-End):
```
DESIGN → SAVE TEMPLATE → ACTIVATE → CREATE DB TABLE →
GENERATE PUBLIC LINK → SHARE → USERS SUBMIT → 
DATA STORED → LINK TO TASKS/TICKETS → ANALYZE
```
**Duration:** One-time setup, Continuous submissions  
**Participants:** ClientAdmin (creator), External/Internal users (submitters)  
**Use Cases:** Audits, Surveys, Feedback, Inspections

---

### Recurring Task Workflow:
```
CREATE TEMPLATE → SET PATTERN → CONFIGURE ASSIGNMENTS →
ACTIVATE → CRON JOB RUNS → AUTO-GENERATE TASK →
ASSIGN USERS → NOTIFY → [Regular Task Lifecycle] →
REPEAT DAILY/WEEKLY/MONTHLY
```
**Duration:** Continuous (until end date)  
**Participants:** ClientAdmin (setup), Auto-assigned users (execution)  
**Use Cases:** Daily checklists, Weekly reports, Monthly audits

---

## End of Workflow Documentation

**Related Documents:**
- `HIERARCHY_FLOW.md` - System hierarchy and permissions
- `API_DOCUMENTATION.md` - API endpoints reference
- `DATABASE_SCHEMA.md` - Database structure

**For Support:**
- Technical Issues: support@rrtms.com
- Documentation: docs@rrtms.com

---

**Document Status:** ✅ COMPLETE  
**Last Review:** November 29, 2025  
**Next Review:** March 2026
