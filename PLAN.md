# Triply — Agreed Specification and Delivery Checklist

This file records the agreed v1 product. It is a delivery checklist, not a finalized technical architecture. Unresolved decisions remain explicitly open.

## Progress rules

- `[ ]` means pending or not verified. `[x]` means implemented and verified.
- Check items only after completing the work and relevant validation; agreement on a feature is not implementation.
- Update this file as work progresses and record validation in the completion log.
- No existing implementation has been audited for this document, so all delivery items start unchecked.

## Product and scope

Triply is an English-language AI trip planner for independent leisure travelers planning city vacations worldwide. It targets iOS and Android. There is no fixed deadline; prioritize a reliable end-to-end experience.

- [ ] Deliver an Expo / React Native app for iOS and Android.
- [ ] Support one destination city per trip, worldwide.
- [ ] Support 1–7 itinerary days, counting arrival and departure days.
- [ ] Keep trips private to their owner.
- [ ] Offer free generation without billing, payments, credits, or per-user generation allowances.
- [ ] Allow repeated generations while shared quotas and overall spending capacity permit.

Out of scope: flight/hotel bookings, multi-city trips, individual activity editing, sharing, offline access, subscriptions, generation packs, and profile-photo uploads. Earlier two-free-generation and paid-pack proposals are superseded.

## Chosen stack

| Technology | Responsibility |
| --- | --- |
| Expo / React Native | Mobile app |
| Clerk | Google and Apple authentication |
| Neon Postgres | Persistent application data |
| Drizzle ORM | Database access |
| Inngest | Background jobs |
| Gemini | AI itinerary generation, free tier only |
| Google Maps / Places | Verified place details and external map links |
| ImageKit | Intended optimization for destination, place, and profile images, subject to source permissions |
| Sentry | Error tracking and monitoring |

Keep this stack. Backend framework, hosting, exact versions, and Gemini model remain undecided. Verify current documentation and compatibility before implementation.

## Authentication and account

- [ ] Implement Google and Apple sign-in through Clerk; no email authentication.
- [ ] Send signed-in users directly to Home; no separate preference onboarding.
- [ ] Use Google/Apple profile photos when available, with initials as fallback.
- [ ] Provide notification preferences, privacy policy access, sign out, and account deletion in Settings.
- [ ] Delete saved trips when their account is deleted.
- [ ] Enforce ownership on the server for every trip read and mutation.

## UI and navigation

- [ ] Use the name Triply.
- [ ] Build a clean, itinerary-focused UI with supporting photos.
- [ ] Use teal as the primary color with neutral backgrounds.
- [ ] Support light and dark themes, following the device by default.
- [ ] Provide Home and Settings tabs with a prominent Create Trip button on Home.
- [ ] Open trip creation and detail views as separate screens.
- [ ] Show saved trips and generation statuses on Home.
- [ ] Support opening, renaming, deleting, and regenerating trips.
- [ ] Prevent deletion while a trip is generating.
- [ ] Review detailed screen designs before finalizing the UI.

Manual theme controls and a day-jump selector were proposed but not explicitly confirmed.

## Step-by-step trip creation

- [ ] Collect a single city, distinguishing cities with the same name.
- [ ] Collect arrival and departure dates; arrival must be today or later.
- [ ] Validate 1–7 itinerary days, counting both arrival and departure.
- [ ] Collect approximate arrival/departure times with a Not sure option.
- [ ] Collect adult and child counts: 1–8 travelers total, at least one adult.
- [ ] Collect each child's age at travel.
- [ ] Ask how many hotel rooms the group needs.
- [ ] Collect a whole-group total budget and user-selected currency.
- [ ] Explain that the budget covers accommodation, food, activities, and local transport, excluding flights.
- [ ] Offer interests: food, culture/history, nature, shopping, adventure, nightlife.
- [ ] Offer relaxed, balanced, and packed pace; default to balanced.
- [ ] Offer dietary presets: no preference, vegetarian, vegan, halal, gluten-free.
- [ ] Offer mobility presets: no preference, minimize walking, step-free access needed.
- [ ] Apply dietary and mobility choices to the whole travel group.
- [ ] Explain that venue dietary/accessibility suitability still requires confirmation.
- [ ] Save unfinished form drafts on the device.
- [ ] Allow revised inputs before regenerating a trip.
- [ ] Count hotel nights between arrival and departure; same-day trips have no hotel stay.

## Trip details and generated content

Use one scrollable screen: overview → day-by-day itinerary → hotel alternatives → budget breakdown → travel tips.

- [ ] Show overview and trip context.
- [ ] Organize each day into morning, afternoon, and evening sections rather than exact schedules.
- [ ] Include verified attractions and restaurants with descriptions and estimated costs.
- [ ] Prefer walking and public transport; allow taxis for mobility needs and include their estimated costs.
- [ ] Show place cards with name, photo when available, description, address, estimated cost, and Open in Google Maps.
- [ ] Use photo placeholders when necessary.
- [ ] Suggest up to three real hotels with location, photos when available, and approximate nightly prices.
- [ ] Label hotel prices as estimates, without implying live availability.
- [ ] Base accommodation cost on one suggested hotel, not all alternatives combined.
- [ ] If hotels are unavailable, show a clear message and approximate accommodation allowance.
- [ ] Break down whole-group accommodation, food, activity, and local transport estimates.
- [ ] For insufficient budgets, generate the closest practical itinerary and clearly show the estimated shortfall.
- [ ] Include practical travel tips.
- [ ] Keep past trips readable; require updated dates before regenerating past trips.

## Background jobs, retries, and failures

- [ ] Run generation asynchronously through Inngest.
- [ ] Enforce one active generation per user on the server.
- [ ] Disable Generate after submission.
- [ ] Reopen the existing active job on repeat submission or reconnection instead of creating duplicates.
- [ ] Allow users to leave and return to view generation status.
- [ ] Verify itinerary places through the selected Google place service.
- [ ] Retry place-verification failures, then fail if verification remains unsuccessful.
- [ ] Do not publish unverified activities as a successful itinerary.
- [ ] Permit a valid itinerary to succeed when hotel suggestions alone are unavailable.
- [ ] Show clear errors with a Retry button.
- [ ] Preserve the previous successful itinerary and associated inputs until regeneration succeeds.
- [ ] Replace the saved trip only after successful regeneration.
- [ ] Keep saved trips accessible during generation pauses and provider outages.

Exact retries, timeouts, job states, and cancellation rules remain technical decisions. Failed attempts can still incur provider costs.

## Notifications

- [ ] Request notification permission after the first trip submission.
- [ ] Keep generation and in-app statuses usable when permission is declined.
- [ ] Send a push notification on successful completion, including successful regeneration.
- [ ] Open the relevant trip when its notification is tapped.
- [ ] Show failures on Home without failure push notifications.
- [ ] Keep notification delivery failure separate from trip-generation success.

## Budget and capacity

The total starting operating budget is USD 20 for backend, AI, and place lookups. Build and publishing costs are separate. Gemini uses its free tier only; free usage is not unlimited.

- [ ] Verify model availability and actual Gemini project quotas.
- [ ] Pause generation when Gemini free capacity is exhausted; resume when capacity returns.
- [ ] Avoid automatic fallback to paid Gemini usage.
- [ ] Validate backend and Google Places costs against the starting budget.
- [ ] Gate generation on service capacity and remaining overall budget.
- [ ] Account for in-flight jobs and retries when designing spending controls.
- [ ] Explain generation pauses before users start the form.
- [ ] Preserve drafts and saved-trip access during pauses.

Verify enforceable provider quotas and billing behavior; an application budget check alone is not a guaranteed billing cap.

## Data and architecture decisions still needed

Core data concepts: users, trip inputs, generation jobs/statuses, saved results, itinerary days/places, hotel alternatives, budget estimates, notification registrations/preferences. Form drafts are local to the device.

- [ ] Select backend framework and hosting after screen-design review, as requested.
- [ ] Define client/server boundaries, authoritative data flow, and protected API contracts.
- [ ] Define Drizzle schemas, relationships, constraints, indexes, and Neon migrations.
- [ ] Keep credentials and privileged database operations on the server.
- [ ] Specify authentication verification, ownership checks, duplicate protection, and safe job retries.
- [ ] Choose notification delivery infrastructure and token lifecycle.
- [ ] Define deletion and retention across identity, database, devices, and provider data.
- [ ] Validate Google Places storage, freshness, attribution, and image-use restrictions.
- [ ] Confirm which images may pass through ImageKit; do not assume Google photos may be cached or transformed.
- [ ] Choose a defensible source for approximate hotel prices and other cost estimates.
- [ ] Define currency support and exchange-rate handling if needed.
- [ ] Define privacy-safe logs, monitoring, alerts, and generation health metrics with Sentry.
- [ ] Define development/staging/production setup, secrets, release process, and recovery procedures.

## Verification and launch

- [ ] Verify Google/Apple sign-in, returning sessions, sign out, and account deletion on both platforms.
- [ ] Verify direct-to-Home entry and saved-trip management.
- [ ] Verify form validation, date boundaries, same-day trips, travelers, room counts, currencies, and draft recovery.
- [ ] Verify real-provider generation end to end.
- [ ] Verify leaving/reopening/reconnecting during generation.
- [ ] Verify repeat taps and concurrent requests do not create duplicate active jobs.
- [ ] Verify errors, retries, throttling, and persistent verification failures.
- [ ] Verify regeneration preserves old content on failure and replaces it once on success.
- [ ] Verify missing hotels/photos and budget-shortfall behavior.
- [ ] Verify cross-user reads and mutations are denied.
- [ ] Verify deletion restrictions while generating.
- [ ] Verify notification permission, denial, delivery, and navigation.
- [ ] Verify quota/budget pauses preserve drafts and saved trips.
- [ ] Verify light/dark themes, readability, and accessibility on iOS and Android.
- [ ] Verify Sentry captures useful failures without leaking credentials or sensitive inputs.
- [ ] Run appropriate lint, type checks, and behavior tests before marking implementation complete.
- [ ] Confirm costs, attribution, image rights, privacy disclosures, and release configuration before public launch.

## Assumptions

- Children are aged 0–17, as proposed during the interview.
- Users may fill and save drafts while generation is paused.
- Detailed screen designs are not yet approved; only visual direction and screen structure are settled.

## Open risks / unknowns

- Backend framework and hosting intentionally remain open.
- Exact Gemini model, quotas, generation quality, and reliability need verification.
- Google pricing, photo rights, caching, attribution, and ImageKit compatibility need verification.
- Hotel-price estimates and any exchange-rate source are undecided.
- Worldwide coverage and dietary/accessibility accuracy cannot be assumed.
- USD 20 may constrain public availability; measure real per-trip costs and capacity.
- Detailed schemas, retention rules, retries, monitoring thresholds, and delivery setup are not finalized.

## Completion log

- 2026-10-22: Recorded agreed product specification and implementation checklist. No application features marked complete without verification.
