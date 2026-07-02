# Namoza Developer Assignment — OrthoNow

Submission for the Developer (Position 1 — Client Web + Martech) assignment.

**Candidate:** Satyam Singh
**Client:** OrthoNow (9 clinics — Bengaluru, Hyderabad, Chennai)

## Repo structure

```
namoza-developer-assignment/
├── task-01-gtm-schema/
│   └── GTM-Event-Schema.md        # Full event schema, funnel drop-off tracking, Ads conversion pick
├── task-02-landing-page/
│   ├── index.html                 # Self-contained landing page (HTML + CSS + JS, no frameworks)
│   └── PAGESPEED-NOTES.md         # How to run and record the PageSpeed score before submitting
├── task-03-integration-design/
│   └── INTEGRATION-ARCHITECTURE.md # HubSpot + WhatsApp + Google Ads integration writeup
└── README.md
```

## How to review

1. **Task 1** — open `task-01-gtm-schema/GTM-Event-Schema.md`. Event table first, then the
   booking funnel drop-off design with the actual dataLayer JSON, then the Google Ads
   conversion pick and reasoning.
2. **Task 2** — open `task-02-landing-page/index.html` directly in a browser (no build step,
   no server needed). Fill the form and submit — the confirmation state swaps in without a
   reload, and `window.dataLayer` will show the `consultation_form_submitted` push in the
   console.
3. **Task 3** — open `task-03-integration-design/INTEGRATION-ARCHITECTURE.md`

