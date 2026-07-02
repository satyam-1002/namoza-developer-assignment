# Task 01 — GTM Event Schema (OrthoNow)

OrthoNow currently has GA4 with pageviews only and no GTM container. Everything below assumes
a fresh GTM container is installed on the WordPress site (head + body snippet) and that GA4
Configuration + Event tags read from a shared **GA4 - Config** tag using a Constant Variable for
the Measurement ID.

## 1. Full event schema

| Event Name | Trigger Type | Key Parameters | Feeds Into (GA4) |
|---|---|---|---|
| `booking_step_1_complete` | Custom Event (dataLayer push, fired by front-end on step 1 submit) | `step_number`, `step_name`, `clinic_location`, `specialty` | Funnel Exploration — Booking Funnel (step 1); Audience: *Started Booking* |
| `booking_step_2_complete` | Custom Event (dataLayer push) | `step_number`, `step_name`, `clinic_location`, `preferred_date` | Funnel Exploration — Booking Funnel (step 2); Audience: *Engaged — Contact Details Entered* |
| `booking_confirmed` | Custom Event (dataLayer push) | `step_number`, `step_name`, `clinic_location`, `specialty`, `booking_id` | Funnel Exploration (step 3, goal step); Conversions report; **imported into Google Ads** (see §3); Audience: *Converted — Booked* |
| `call_now_click` | Click Trigger — Just Links / Click Element, matches CSS class `.call-now-btn` (fires on homepage, clinic pages, and landing page) | `click_text`, `page_location`, `clinic_location` | Engagement report (Events); Audience: *High Intent — Called* |
| `whatsapp_chat_opened` | Click Trigger — Click Element, matches the floating widget's `wa.me` link | `click_url`, `page_location`, `widget_type` | Engagement report; Audience: *WhatsApp Intent* |
| `patient_guide_download` | Two triggers feeding one event name: (a) Form Submission trigger on the gated name+phone form, which on success pushes this event via dataLayer, (b) as a fallback, GTM's built-in **File Download** auto-event trigger scoped to `.pdf` matching the guide's filename | `file_name`, `file_extension`, `lead_captured` (boolean — true only when the gate form succeeded), `page_location` | Audience: *Content Downloaders — Guide*; Lead gen report |
| `clinic_page_view` | Page View trigger, scoped with a Page Path RegEx matching the 9 clinic URL pattern (e.g. `/clinics/[a-z-]+/`) | `clinic_name`, `city`, `page_path` | Custom Exploration — clinic performance by location; Audience: *Viewed [City] Clinic* |
| `blog_scroll_depth` | GTM built-in **Scroll Depth** trigger, vertical thresholds 25 / 50 / 75 / 90% | `percent_scrolled`, `article_title`, `article_category` | Engagement / Content report; Audience: *Engaged Readers* |

All nine clinic pages share the same `clinic_page_view` tag — the clinic-specific values come
from a Data Layer Variable (`clinic_name`, `city`) that the CMS template pushes on page load, not
from nine separate GTM tags. Same principle for the blog: one Scroll Depth trigger, article
metadata comes from page-level dataLayer variables set per post.

## 2. Booking funnel drop-off (3-step form)

**This is the part GTM cannot do on its own.** GTM's native triggers (Click, Form Submission,
Page View) only fire on things that happen to the DOM — a multi-step form that swaps content
in and out of the same page, without a real page load or a native form `submit` event between
steps, is invisible to GTM by default. The front-end has to tell GTM when a step completes by
pushing a custom event to `window.dataLayer` at the moment each step is validated and the user
moves forward. GTM then just listens for those custom events with three separate Custom Event
triggers.

**What I'd brief the front-end dev to build**, e.g. for step 2 (contact details):

> When the user fills name + phone + preferred date and clicks "Next," before moving to step 3,
> push to the dataLayer with the event name, step number, step name, and whatever selections
> carried over from step 1. Do this on every successful step advance, not on page load and not
> retroactively when the whole form is submitted — otherwise GA4 will only ever see the final
> step and the drop-off numbers will be meaningless.

**Step 1 — location + specialty selected:**

```json
{
  "event": "booking_step_1_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "OrthoNow Indiranagar",
  "specialty": "Knee & Sports Injury"
}
```

**Step 2 — contact details entered:**

```json
{
  "event": "booking_step_2_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "OrthoNow Indiranagar",
  "preferred_date": "2026-07-08"
}
```

**Step 3 — booking confirmed:**

```json
{
  "event": "booking_confirmed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "OrthoNow Indiranagar",
  "specialty": "Knee & Sports Injury",
  "booking_id": "ORN-2026-004821"
}
```

**GTM setup:** three Custom Event triggers (`booking_step_1_complete`, `booking_step_2_complete`,
`booking_confirmed`), each firing a corresponding GA4 Event tag that reads `step_number`,
`step_name`, `clinic_location`, etc. from Data Layer Variables and passes them through as GA4
event parameters. `clinic_location` and `specialty` need to be registered as custom
dimensions in GA4 (event-scoped) or they won't show up as breakdown options in reporting.

**Surfacing drop-off in GA4:** build a **Funnel Exploration** with three steps, in order —
`booking_step_1_complete` → `booking_step_2_complete` → `booking_confirmed`. Use "closed funnel"
so it only counts users who followed the exact sequence, which is what actually matters for
diagnosing where the form leaks. GA4 will show the % completing each step and the % drop-off
between consecutive steps directly on the funnel visualization. Add `clinic_location` as a
breakdown dimension so you can see if drop-off is concentrated at one clinic (which usually means
a slot-availability or UX issue specific to that location, not a form problem).

## 3. Which conversion to import into Google Ads

**Import `booking_confirmed`, not `call_now_click` or `patient_guide_download`.**

`booking_confirmed` is the only event in this schema that represents a completed, bottom-of-funnel
action — an actual scheduled patient. Setting Ads Smart Bidding to optimise toward this means the
algorithm is chasing real appointments.

The other two are worse proxies:
- `call_now_click` only tells you a call was *attempted*, not that a patient was booked — without
  call tracking (recording call duration/outcome) it's noise, and importing it as a conversion
  would have Google Ads optimising toward people who tap a button, not people who actually book.
- `patient_guide_download` is a top-of-funnel research action. Importing it would pull spend
  toward casual browsers rather than people ready to book, which is the opposite of what OrthoNow
  needs given the campaign is already underperforming on conversion rate.

If OrthoNow later adds call tracking (e.g. a call-tracking number swap via GTM), `call_now_click`
could be upgraded to a secondary conversion for reporting — but as a primary Ads-optimisation
signal, `booking_confirmed` is the correct single choice today.
