
---

# 📘 **Election Management API — Documentation**

This API powers the election workflow in your school portal: creation, publishing, candidate approval, rejection, previewing, and reporting.

---

# ✨ **1. Create Election**

### **POST**

`/evoting/create_election/`

Creates a new election with optional eligibility rules and positions.

---

### ✅ **Request Body (JSON)**

```json
{
  "title": "SUG Election",
  "session": 4,
  "start_date": "2025-01-10",
  "end_date": "2025-01-11",
  "start_time": "09:00",
  "end_time": "16:00",
  "application_deadline": "2025-01-05",
  "allowed_levels": ["100", "200", "300"],
  "allowed_faculties": ["SCIENCE", "ENGINEERING"],
  "positions": ["PRESIDENT", "VICE PRESIDENT", "SECRETARY"]
}
```

---

### 📌 **Important Notes**

* `title` and `session` are **required**.
* Duplicate titles for a session are **not allowed**.
* `allowed_levels`, `allowed_faculties`, and `positions` are **optional arrays**.
* Position names are automatically uppercased.

---

### ✅ **Sample Success Response (201)**

```json
{
  "message": "Election created successfully",
  "election": {
    "id": 12,
    "title": "SUG Election",
    "session": 4,
    "allowed_levels": ["100", "200", "300"],
    "allowed_faculties": ["SCIENCE", "ENGINEERING"],
    "positions": ["PRESIDENT", "VICE PRESIDENT", "SECRETARY"],
    "published": false
  }
}
```

---

### ❌ **Sample Error Responses**

**Missing Required Fields (400):**

```json
{
  "error": "Title and session are required fields."
}
```

**Invalid Session (404):**

```json
{
  "error": "Invalid session ID."
}
```

**Duplicate Election (400):**

```json
{
  "error": "An election with this title already exists for the selected session."
}
```

---

---

# ✨ **2. Publish Election**

### **PATCH**

`/evoting/publish_election/<election_id>/`

Publishes an election to make it visible for voting.

---

### ❌ **No Body Required**

---

### ✅ **Sample Success Response (200)**

```json
{
  "message": "Election published successfully.",
  "election": {
    "id": 12,
    "title": "SUG Election",
    "published": true
  }
}
```

---

### ⚠️ **If Already Published (200)**

```json
{
  "message": "Election is already published."
}
```

---

### ❌ **Not Found (404)**

```json
{
  "error": "Election not found."
}
```

---

---

# ✨ **3. Get Election Candidates**

### **GET**

`/evoting/get_election_candidates/`

Fetches all candidates for the current or specified session.

---

### 🔍 **Query Parameters**

| Param          | Type           | Required | Example          | Description                           |
| -------------- | -------------- | -------- | ---------------- | ------------------------------------- |
| `session_id`   | number         | no       | `4`              | Filter by a session                   |
| `search_query` | string         | no       | `john`           | Search by name or reg number          |
| `position`     | string         | no       | `PRESIDENT`      | Filter by position name               |
| `status`       | boolean/string | no       | `true` / `false` | Filter approved or pending candidates |

---

### Example Request

```
/evoting/get_election_candidates/?session_id=4&status=true&position=PRESIDENT
```

---

### ✅ **Sample Response**

```json
[
  {
    "id": 88,
    "student": {
      "id": 401,
      "full_name": "John Doe",
      "registration_number": "ENG/19/2038"
    },
    "position": "PRESIDENT",
    "approved": true,
    "checked": true
  }
]
```

---

---

# ✨ **4. Reject Candidate**

### **POST**

`/evoting/reject_candidate/`

Reject a student's application for a position.

---

### 📩 **Request Body**

```json
{
  "candidate_id": 55,
  "message": "You did not meet the CGPA requirement."
}
```

---

### ⚠️ Required:

* `candidate_id`
* `message`

---

### ✅ **Sample Success Response**

```json
{
  "id": 55,
  "approved": false,
  "checked": true,
  "message": "You did not meet the CGPA requirement.",
  "student": {
    "id": 201,
    "full_name": "Jane Doe",
    "registration_number": "SCI/20/1001"
  },
  "position": "VICE PRESIDENT"
}
```

---

### ❌ **Error (400)**

```json
{
  "error": "The candidate ID and the message field are required"
}
```

---

---

# ✨ **5. Approve Candidate**

### **POST**

`/evoting/approve_candidate/`

Approves a student's application.

---

### 📩 **Request Body**

```json
{
  "candidate_id": 55
}
```

---

### ⚠️ Required:

* `candidate_id`

---

### ✅ **Sample Response**

```json
{
  "id": 55,
  "approved": true,
  "checked": true,
  "student": {
    "id": 201,
    "full_name": "Jane Doe"
  },
  "position": "VICE PRESIDENT"
}
```

---

### ❌ **Error (400)**

```json
{
  "error": "The candidate ID is required"
}
```

---

---

# ✨ **6. Get Elections**

### **GET**

`/evoting/get_elections/`

Retrieve all elections with optional filtering.

---

### 🔍 **Query Params**

| Param        | Type   | Description       |
| ------------ | ------ | ----------------- |
| `session_id` | number | Filter by session |
| `query`      | string | Search by title   |

---

### Example

```
/evoting/get_elections/?session_id=4&query=SUG
```

---

### ✅ **Sample Response**

```json
[
  {
    "id": 12,
    "title": "SUG Election",
    "session": 4,
    "published": false
  }
]
```

---

---

# ✨ **7. Get Election Preview**

### **GET**

`/evoting/get_election_preview/<id>/`

Returns detailed info including positions, rules & eligibility.

---

### Sample Response

```json
{
  "id": 12,
  "title": "SUG Election",
  "session": {
    "id": 4,
    "year": "2024/2025"
  },
  "positions": ["PRESIDENT", "VICE PRESIDENT"],
  "allowed_levels": ["200", "300"],
  "allowed_faculties": ["SCIENCE"],
  "published": false
}
```

---

---

# ✨ **8. Election Status Counts**

### **GET**

`/evoting/election_status_counts/`

Provides an overview of all elections and candidate stats.

---

### 🔍 Query Params

| Param      | Type   | Example | Description                 |
| ---------- | ------ | ------- | --------------------------- |
| session_id | number | `4`     | Filter elections by session |

---

### Sample Response

```json
{
  "upcoming": 2,
  "ongoing": 1,
  "completed": 3,
  "total_candidates": 445,
  "pending_applications": 82
}
```

---

---

# ✨ **9. Recent Activities**

### **GET**

`/evoting/recent_activities/`

Returns the latest 20 election-related logs.

---

### Sample Response

```json
[
  {
    "message": "John Doe applied for PRESIDENT",
    "type": "NEW_APPLICATION",
    "timestamp": "2025-01-10 14:21"
  },
  {
    "message": "SUG Election Created - 2024/2025 Session",
    "type": "ELECTION_CREATED",
    "timestamp": "2025-01-10 13:00"
  }
]
```

---

---

# ✨ **10. Daily Grouped Activity Summary**

### **GET**

`/evoting/grouped_daily_activity_summary/`

Groups today’s activities into categories.

---

### Sample Response

```json
{
  "date": "2025-01-10",
  "elections_created": 1,
  "new_applications": 23,
  "candidates_approved": 4,
  "candidates_rejected": 2,
  "other_activities": 1,
  "recent_logs": [
    {
      "message": "Jane Doe approved for SECRETARY",
      "action_type": "CANDIDATE_APPROVED",
      "time": "14:45"
    }
  ]
}
```

---
