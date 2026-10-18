# An Appointment Intake for Animal Clinic - n8n Workflow

Automated appointment booking pipeline for a veterinary clinic, built with **n8n** (self-hosted, Community Edition). A website booking form sends data to an n8n webhook. The workflow cleans and validates the request, then stores valid bookings in Google Sheets - no manual data entry for clinic staff.

---

## Business Problem

The clinic received appointment requests through calls and messages, which staff had to record manually. This caused:
- Missed or duplicated bookings
- Inconsistent data (wrong phone formats, past dates, typos)
- No single place to see all upcoming appointments

## Solution

An automated intake workflow that accepts bookings 24/7, rejects invalid requests, and keeps a clean, structured booking log.

---

## Workflow

![Workflow](screenshots/workflow.png)

```
Webhook (POST) → Edit Fields → If (validate) ─┬─ true  → Google Sheets (append row)
                                              └─ false → (planned) error response
```

| Node | Purpose |
|---|---|
| **Webhook** | Receives booking JSON from the website form (`POST /webhook/appointment`) |
| **Edit Fields** | Normalizes data - trims names, lowercases emails, renames fields, generates `bookingId`, sets `status = Pending` |
| **If** | Validates required name, email format, Sri Lankan mobile format (`07XXXXXXXX`), and future appointment date |
| **Google Sheets** | Appends the valid booking as a new row (raw cell format to preserve leading zeros) |

---

## n8n Concepts Used

- Webhook trigger (test vs production URLs)
- Expressions: `$json`, `$now`, `$today`, Luxon date formatting
- Data transformation with **Edit Fields (Set)**
- Conditional branching with **If** (regex + date comparisons)
- **Google Sheets OAuth2** credentials (self-hosted Google Cloud app)
- Pinned test data for development

---

## Sample Request

```bash
curl -X POST http://localhost:5678/webhook/appointment \
  -H "Content-Type: application/json" \
  -d '{
    "ownerName": "Nimal Perera",
    "phone": "0771234567",
    "email": "nimal@test.com",
    "petName": "Rocky",
    "petType": "Dog",
    "date": "2026-10-20",
    "time": "10:30",
    "reason": "Vaccination"
  }'
```

## Result

![Google Sheet result](screenshots/sheet-result.png)

| bookingId | ownerName | phone | email | petName | petType | appointmentDate | appointmentTime | reason | status |
|---|---|---|---|---|---|---|---|---|---|
| VO-20261016161603 | Nimal Perera | 0771234567 | nimal@test.com | Rocky | Dog | 2026-10-20 | 10:30 | Vaccination | Pending |

---

## Setup

### 1. Run n8n
```bash
docker compose up -d
```
Open `http://localhost:5678` and create an owner account.

### 2. Google Sheet
Create a sheet with these headers in row 1:
```
bookingId | ownerName | phone | email | petName | petType | appointmentDate | appointmentTime | reason | status
```

### 3. Google credentials
1. In Google Cloud Console, create a project and enable **Google Sheets API** and **Google Drive API**
2. Configure the OAuth consent screen (External) and add yourself as a test user
3. Create an OAuth client (Web application) with redirect URI:
   `http://localhost:5678/rest/oauth2-credential/callback`
4. Add the Client ID and Secret to a **Google Sheets OAuth2** credential in n8n

### 4. Import the workflow
In n8n: **Create workflow → ⋯ → Import from File** → select `workflow.json`.
Then open the Google Sheets node and choose your sheet (replaces `YOUR_SHEET_ID`).

### 5. Activate
Click **Publish** to enable the production webhook URL.

---

## Roadmap

- [ ] Telegram alert to clinic staff on new bookings
- [ ] Confirmation email to pet owner
- [ ] Error response with validation messages for invalid requests
- [ ] Duplicate booking detection

---

## Tech Stack

n8n · Docker · Google Sheets API · OAuth2 · Postman

## Author

**Akila** - Full-stack Software Engineer
