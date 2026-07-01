# Task 1 – GTM Event Schema
## OrthoNow Website – Full Event Tracking Plan

---

## 1. Complete GTM Event Schema Table

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|---|---|---|---|
| `booking_step_complete` | Custom Event (dataLayer push) | `step_number`, `step_name`, `clinic_location`, `specialty` | Funnel Exploration Report |
| `call_now_click` | Click – Just Links | `page_location`, `clinic_name`, `device_type` | Events Report, Remarketing Audience |
| `whatsapp_click` | Click – Just Links | `page_location`, `widget_type`, `device_type` | Events Report |
| `pdf_download_form_submit` | Form Submission | `form_name`, `clinic_name`, `page_location` | Conversions Report |
| `clinic_page_view` | Page View | `clinic_name`, `clinic_city`, `page_path` | Pages & Screens Report |
| `blog_scroll_depth` | Scroll Depth (25/50/75/90%) | `article_title`, `scroll_percent`, `time_on_page` | Engagement Report |
| `consultation_form_submitted` | Custom Event (dataLayer push) | `form_name`, `clinic_preference`, `page_location` | Conversions Report, Google Ads |

---

## 2. Booking Form – Step-by-Step Funnel Tracking

### How it works
GTM cannot automatically detect when a user moves from Step 1 to Step 2 in a multi-step form.
The **front-end developer** must write a `dataLayer.push()` call in the JavaScript that runs
when the user clicks "Next" on each step. GTM listens for the event name and captures the data.

---

### Step 1 – Select Clinic Location + Specialty

**GTM Trigger:** Custom Event — event name equals `booking_step_complete`

**dataLayer Push (written by front-end dev):**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee Replacement"
}
```

---

### Step 2 – Enter Name / Phone / Preferred Date

**GTM Trigger:** Custom Event — event name equals `booking_step_complete`

**dataLayer Push (written by front-end dev):**
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee Replacement",
  "preferred_date": "2026-07-05"
}
```

---

### Step 3 – Confirm Booking

**GTM Trigger:** Custom Event — event name equals `booking_step_complete`

**dataLayer Push (written by front-end dev):**
```json
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee Replacement",
  "preferred_date": "2026-07-05",
  "booking_id": "ORN-20260629-001"
}
```

---

### How to surface funnel drop-off in GA4 Funnel Exploration

1. Go to GA4 → Explore → Funnel Exploration
2. Add 3 steps:
   - Step 1: Event = `booking_step_complete` AND `step_number` = 1
   - Step 2: Event = `booking_step_complete` AND `step_number` = 2
   - Step 3: Event = `booking_step_complete` AND `step_number` = 3
3. GA4 will show exactly how many users dropped between Step 1→2 and Step 2→3

---

## 3. Conversion Action to Import into Google Ads

**Import: `consultation_form_submitted` (Step 3 – Booking Confirmed)**

### Why this one over the others:

- It represents the **highest intent action** — the user completed all 3 steps
- It is the closest signal to an actual booked patient (real revenue for OrthoNow)
- Using Step 1 or Step 2 would send too many low-intent signals to Google Ads Smart Bidding
- Google Ads **Target CPA** and **Max Conversions** bidding works best with a clean,
  high-intent conversion signal — not top-of-funnel clicks
- `call_now_click` was the second option considered, but calls don't confirm a booking —
  a completed booking form does
