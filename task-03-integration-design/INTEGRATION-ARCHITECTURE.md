# Task 03 — Integration Design

## End-to-end architecture

On submit, the landing page JS fires the `consultation_form_submitted` dataLayer push (Task 02)
and, in parallel, calls a small serverless function I'd own (a Vercel/Netlify Node function)
rather than posting straight to HubSpot's native embed or Forms API. That function is the single
orchestration point:

1. **Dedup check** — before creating anything, it calls the **HubSpot Contacts Search API**
   filtering on the `phone` property. HubSpot's default dedup is on email, and this form never
   collects one — Indian healthcare lead gen almost never does — so a native form embed would
   turn every repeat enquiry into a duplicate contact with a fragmented history. The function
   does the phone lookup itself and either updates the existing contact or creates a new one,
   with phone as the effective unique key.
2. **Contact upsert** — via the **HubSpot Contacts API** (create-or-update), setting Name, Phone,
   Clinic Preference, `Source = Google Ads - Consultation Landing Page`, and `Lead Status = New
   Enquiry`.
3. **WhatsApp confirmation** — the same function calls **Karix's WhatsApp Business API** directly
   (not through Make/Zapier), since a middleware hop is one more place for the 2-minute SLA to
   slip.
4. **Google Ads conversion** — fired server-side via the **Google Ads API / Enhanced Conversions**
   on the same request, rather than relying only on the client-side GTM tag, so the conversion
   still records even if the user closes the tab right after submitting.

A direct API call beats Zapier/Make here because this flow has a hard SLA and a non-trivial dedup
rule — both are easier to control, log, and retry inside one function than across a third-party
tool's own queue and rate limits.

## Biggest failure point and fallback

The **Karix WhatsApp call** is the weakest link — it's the one external dependency with its own
uptime and delivery variability. Fallback: if the Karix call fails or times out, the function
queues an SMS confirmation as a backup channel and flags the contact in HubSpot with `Lead Status
= New Enquiry - WhatsApp Failed`, so the clinic team knows to call manually.

## Protecting the 2-minute SLA

Karix rate limits, network timeouts, or the serverless function cold-starting could all blow the
SLA. I'd monitor this with a logging/alerting layer (e.g. a lightweight table or a service like
Better Stack) that records a timestamp at form submit and at WhatsApp delivery confirmation, and
alerts if the gap exceeds 90 seconds — giving a buffer to react before the SLA is actually missed.
