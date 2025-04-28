# 📢 Notifications Management System

This microservice provides a comprehensive notification system for **tracking employee attendance, work hours, and activities**. It monitors various statuses including late arrivals, absences, lunch breaks, skill changes, and early logouts, generating real-time notifications for managers and team leaders.

---

## 🏗️ Project Structure

.env                    # Environment variables
.gitignore              # Ignored files for Git
database.py             # Database configuration and connection
main.py                 # Entry point for the FastAPI application
models.py               # SQLAlchemy models
requirements.txt        # Project dependencies
schemas.py              # Pydantic schemas for validation
services.py             # Business logic and service layer
routers/                # API route definitions


---

## ⚙️ Core Functionalities

### 📅 Attendance & Late Arrival Tracking
**Service:** `still_absent()` & `current_late()`

- **Purpose:**
  - Detect employees who are absent (no check-in).
  - Identify employees who are late (checked in after scheduled time + grace period).
- **Implementation:**
  - Query the `Workhours` table for schedules.
  - Compare against `StatusChange` records (Working/Coaching/Training/Lunch).
  - No status ➔ Mark as absent.
  - Status after grace period ➔ Mark as late.

---

### 🍽️ Lunch Break Monitoring
**Service:** `get_exceeded_lunch_times_realtime()` & `get_missed_lunch()`

- **Purpose:**
  - Detect employees exceeding lunch time.
  - Identify employees who missed their lunch.
- **Implementation:**
  - Compare `Workhours` scheduled lunch vs. `StatusChange` lunch duration.
  - Excess ➔ Flagged as exceeded.
  - No lunch ➔ Flagged as missed.

---

### 🚪 Early Logout Detection
**Service:** `get_the_logout_time()`

- **Purpose:**
  - Detect employees who logout before scheduled end time.
- **Implementation:**
  - Compare `Workhours.fin` (end time) vs. last `StatusChange.end_time`.
  - Early logout (≥ 5 minutes) ➔ Flagged.

---

### 🛠️ Skill/Task Compliance Check
**Service:** `get_skills_change()`

- **Purpose:**
  - Detect if employees performed tasks different from scheduled.
- **Implementation:**
  - Compare `Workhours.skill` (planned) vs. `Traitement.skill` (actual).

---

### 🔔 Notification Management
**Service:** `store_notification()`, `get_notifications()`, `mark_notifications_as_viewed()`

- **Purpose:**
  - Store, retrieve, and manage notifications with deduplication.
- **Implementation:**
  - **Deduplication** to prevent redundant alerts.
  - **Role-based filtering**:
    - Managers/Superusers ➔ All notifications.
    - Team Leaders (CL) ➔ Assigned activity only.
  - Mark notifications as viewed.

---

## 🚀 API Endpoints

| Endpoint                     | Method | Description                                  |
| ----------------------------- | ------ | -------------------------------------------- |
| `/get-late-employees`         | GET    | List late employees.                        |
| `/still_absent`               | GET    | List absent employees.                      |
| `/get-exceeded-lunch-times`   | GET    | List employees who exceeded lunch time.     |
| `/logout_early`               | GET    | List employees who logged out early.         |
| `/skills_change`              | GET    | List employees with task mismatches.         |
| `/missed_lunch`               | GET    | List employees who missed lunch.             |
| `/notifications`              | GET    | Retrieve notifications (role-based access). |
| `/mark-notifications-viewed`  | POST   | Mark notifications as read.                 |
| `/store_notifications`        | POST   | Store a new notification.                   |

---

## 🧩 Technical Implementation Details

### 🌍 Timezone Handling
- Use **UTC** for storage.
- Convert to **Africa/Tunis (UTC+1)** for display.
- Grace periods (e.g., 5-minute late allowance) are **timezone-aware**.

### 🛡️ Deduplication Logic
- Prevents duplicate notifications within a time window (default 24h) if:
  - Same text.
  - Same shift ID (`id_shift`).
  - Within last configurable hours.

### 👥 Role-Based Access Control
- **Superusers/Managers**: View all notifications.
- **Team Leaders (CL)**: View notifications related to their activities.
- **Employees**: No access to notifications.

---

## 🔄 API Flow

1. Employee schedules loaded from `Workhours`.
2. Real-time checks vs. `StatusChange`.
3. Notifications generated for any discrepancies.
4. Managers retrieve alerts via `/notifications`.
5. Notifications marked as viewed when acknowledged.
