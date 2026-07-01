# Task 3 – Integration Design
## Connecting the Landing Page → HubSpot → WhatsApp (Karix) → Google Ads

---

## End-to-End Architecture

When a patient submits the consultation form, the following sequence happens automatically:

**Step 1 — Form submit fires a dataLayer push**
The landing page fires `window.dataLayer.push({ event: 'consultation_form_submitted' })`.
GTM picks this up and triggers two things at once: the Google Ads conversion tag, and a call
to the backend.

**Step 2 — Backend API receives the form data**
A lightweight backend endpoint (Node.js/Express) receives Name + Phone posted from the form.
This sits between the landing page and all downstream systems.

**Why a direct backend call instead of Zapier or Make:** a direct API call gives full control
over error handling, retry logic, and speed — which matters because of the 2-minute WhatsApp
SLA. Zapier/Make add extra latency (polling delays, queue hops) that can easily eat into that
2-minute window. A direct call is faster and more reliable for a time-sensitive healthcare lead.

**Step 3 — Contact created/updated in HubSpot via Contacts API**
The backend calls HubSpot's Contacts API (`POST /crm/v3/objects/contacts`) with: Name, Phone,
Clinic Preference, Source = "Google Ads - Consultation Landing Page", Lead Status = "New Enquiry".

**Step 4 — WhatsApp confirmation via Karix**
Immediately after the HubSpot write succeeds, the backend calls Karix's WhatsApp Business API
with the patient's phone number and a pre-approved confirmation message template.

**Step 5 — Google Ads conversion fires**
This happens client-side in parallel (Step 1), with no dependency on the backend — so even if
HubSpot or Karix is briefly down, the Google Ads conversion still fires correctly.

---

## The Phone Number Deduplication Trap

HubSpot's default deduplication is based on **email**, not phone number. Since this form
collects only Name + Phone (no email), two patients submitting with the **same phone number**
but **different names** would create two separate, duplicate contacts in HubSpot by default.

**Fix:** before creating a new contact, the backend first calls HubSpot's Search API
(`POST /crm/v3/objects/contacts/search`) filtering by the `phone` property. If a match is
found, the existing contact is updated (`PATCH`) instead of creating a new one — and the name
field is only overwritten if it was previously blank, to avoid losing a returning patient's
correct details.

---

## Biggest Failure Point

The **Karix WhatsApp API call** is the single biggest point of failure. If Karix is down,
rate-limited, or the message template gets rejected, the confirmation simply doesn't send —
and the patient is left wondering if their booking went through.

**Fallback:** add a simple message queue (a database table with status =
`pending / sent / failed`). A cron job retries failed messages every 30 seconds, up to 5
attempts, before flagging it for manual follow-up by the clinic staff.

---

## Monitoring the 2-Minute SLA

**What could break it:** Karix API downtime, rate limiting during high-traffic campaign
periods, slow HubSpot write blocking the WhatsApp call, or network latency on the backend.

**How to monitor it:** log a timestamp the moment the form submits, and a second timestamp
when Karix confirms delivery. Calculate the gap on every submission. If the gap exceeds
2 minutes, trigger an alert via a Slack webhook (or a tool like Better Uptime) so the team
can investigate immediately rather than discovering it after a patient complaint.

---

**Word count: ~390 words**
