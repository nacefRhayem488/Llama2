# Notifications Management System

This microservice provides a comprehensive notification system for tracking employee attendance, work hours, and activities. It monitors various employee statuses including late arrivals, absences, lunch breaks, skill changes, and early logouts, generating appropriate notifications for managers and team leaders.


##project structure
.env
.gitignore
database.py          # Database configuration and connection
main.py              # Entry point for the FastAPI application
models.py            # Database models
requirements.txt     # Project dependencies
schemas.py           # Pydantic schemas for request/response validation
services.py          # Business logic and service layer
routers/             # API route definitions
## Core Functionalities


### Attendance & Late Arrival Tracking

Service: still_absent() & current_late()
    .Purpose:

        -Detects employees who are absent (did not check in at all).

       -Identifies employees who are late (checked in after their       scheduled start time + grace period).

    .Implementation:

        -Queries Workhours table for scheduled shifts.

        -Compares against StatusChange records (Working/Coaching/Training/Lunch).

        -If no status is found, marks as absent.

        -If status is recorded after the scheduled time + 5-minute grace period, marks as late.


### Lunch Break Monitoring

Service: get_exceeded_lunch_times_realtime() & get_missed_lunch()
    .Purpose:

        -Detects employees who exceeded their allowed lunch duration.

        +Identifies employees who missed their scheduled lunch break.

    .Implementation:

        -Checks Workhours for scheduled lunch times.

        -Compares against StatusChange (Lunch status duration).

        -If lunch duration exceeds configured limits, flags as exceeded.

        -If no lunch status is recorded, flags as missed.

### Early Logout Detection

Service: get_the_logout_time()
    .Purpose:

        -Detects employees who logged out before their scheduled shift end.

    .Implementation:

        -Compares Workhours.fin (scheduled end time) against the last StatusChange.end_time.

        -If logout is 5+ minutes early, flags it.

### Skill/Task Compliance Check

Service: get_skills_change()
    .Purpose:

        -Detects if employees performed tasks different from their scheduled assignments.

    .Implementation:

        -Compares Workhours.skill (planned task) vs. Traitement.skill (actual task).

### Notification Management

Service: store_notification(), get_notifications(), mark_notifications_as_viewed()
    .Purpose:

        -Stores, retrieves, and manages notifications with deduplication.

    .Implementation:

        -Deduplication: Prevents duplicate alerts for the same event within a time window.

        -Role-based filtering:

            Managers/Superusers see all notifications.

            Team Leaders (CL) only see notifications for their assigned activity.

        -Mark as viewed: Updates Notifications.viewed status.

## API Endpoints

Endpoint	               |  Method	   |Description
/get-late-employees	       |  GET	Lists late employees.
/still_absent	           |  GET	Lists absent employees.
/get-exceeded-lunch-times  |  GET	employees who exceeded lunch time.
/logout_early	           |  GET	Lists employees who logged out early.
/skills_change	           |  GET	Lists employees with task mismatches.
/missed_lunch	           |  GET	Lists employees who missed lunch.
/notifications	           |  GET	Retrieves notifications (role-based).
/mark-notifications-viewed |  POST	Marks notifications as read.
/store_notifications	   |  POST	Stores a new notification.


## Technical Implementation Details

### Timezone Handling

-Uses UTC for storage.

-Converts to Africa/Tunis (UTC+1) for display.

-Grace periods (e.g., 5-minute late allowance) are timezone-aware.

### Deduplication Logic
Checks for existing notifications with:

    -Same text

    -Same shift ID (id_shift)

    -Within the last 24 hours (configurable)

### Role-Based Access Control
-Superusers/Managers: See all notifications.

-Team Leaders (CL): Only see notifications for their current_activity.

-Regular Employees: Cannot view notifications.

## The api flow 

-Employee schedules are loaded from Workhours.

-Real-time checks compare schedules against StatusChange.

-Notifications are generated for discrepancies.

-Managers retrieve alerts via /notifications.

-Notifications are marked as viewed when acknowledged.
