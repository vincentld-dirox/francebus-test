# Design: France Bus Smart Quote Form V1

## Context

Greenfield: no application code, no specs, and no `okf/decisions/` or `okf/architecture/` files yet. The foundation of this change writes `okf/architecture/overview.md` from this design and one decision file per marked decision below. Motivation and scope: see `proposal.md`.

The constraints that shape the design (from the FBQ-1 charter):
- The two WordPress sites stay at Infomaniak, unchanged: no server change, shared hosting.
- The JetFormBuilder form is kept and edited with its own features; the form keeps its look.
- The CRM is unchanged: France Bus's e-mail keeps today's sender, subject "Francebus Service", labels and order.
- Testing and staging on Dirox's servers; production in France Bus's own Google Cloud account, Cloud Run, EU region, with a budget alert; deployment to production only after France Bus's approval.
- AI on Gemini through Vertex AI, EU endpoint; personal data processed and stored in the EU, not used for training.
- Tariffs and rules are configuration, updated by Dirox without redevelopment; the values come from discovery.
- About 190 to 290 requests a month across both sites; French and English only.

## Goals / Non-Goals

**Goals:**
- One contract between the engine and the plugin, so the engine and WordPress work can go on in parallel.
- Every figure from a deterministic, testable calculation core; the AI only at the language edges.
- No request lost and one France Bus e-mail per request, whatever fails (engine, Gemini, mailbox, browser).
- The same engine image from a developer's machine to Cloud Run.

**Non-Goals:**
- No administration screen: the tariff configuration changes by pull request, the log leaves as CSV.
- No live push to the browser: the window calls the engine and waits for the answer.
- No high availability beyond what Cloud Run and Cloud SQL give by default; the JetFormBuilder fallback covers an engine outage.

## Architecture

```
Visitor's browser (form page: JetFormBuilder form + the plugin's script)
  |-- address suggestions, estimate window ---------------------> Engine API (Cloud Run, EU)
  |      website public key + Origin check + request token           |-- assistant ----> Gemini on Vertex AI (EU)
  |                                                                  |-- calculation core (pure) <-- tariff configuration (versioned files)
  v                                                                  |-- maps ---------> Places, Routes, Geocoding
WordPress at Infomaniak: francebus.com, francebus-wedding.com        |-- e-mail -------> outbox
  plugin (PHP)                                                       '-- store --------> PostgreSQL (EU)
  |-- on Submit (JetFormBuilder hook): hand-off ------------------> engine       (website secret)
  |-- confirm, call request, outbox sweep (WP-Cron) <-------------> engine       (website secret)
  '-- wp_mail via the Infomaniak mailbox devis@francebus.com -----> France Bus CRM mailbox, the visitor
  engine unreachable at Submit: JetFormBuilder's own Send Email, as today
```

### Repository layout

| Folder | Holds |
|---|---|
| `engine/` | the France Bus AI Engine service, its migrations and its container image |
| `contract/` | the API schemas shared by the engine and the window (request and response shapes) |
| `tariff/` | the tariff configuration versions and their schema |
| `calibration/` | the anonymised test set of past quotes and the calibration runner |
| `wordpress-plugin/` | the plugin (PHP) and the window's sources (TypeScript) |
| `jetformbuilder/` | the exported JetFormBuilder forms of both websites, for review and re-import |
| `infra/` | deployment of the engine on Dirox's servers and on Google Cloud |

### Engine modules

| Module | Does | Uses |
|---|---|---|
| `api` | HTTP routes, input validation against `contract/`, website credentials, Origin checks, rate limits, request tokens | all below |
| `requests` | the request's life: hand-off, answers, edits, alternatives, confirmation, call request, unconfirmed delay; writes the estimate log | assistant, calc, maps, email, store |
| `assistant` | Gemini calls with response schemas, output validation, questions, summary, explanations | Vertex AI |
| `calc` | pure functions: vehicle selection, itinerary, driving rules and alternatives, pricing, breakdown, rounding | tariff, legs from maps |
| `maps` | address suggestions (proxy), Routes, Geocoding, route cache | Google Maps Platform, store |
| `tariff` | loads and validates a tariff configuration version, gives its version id | `tariff/` |
| `email` | renders the France Bus and client e-mails from fixed templates, fills the outbox | store |
| `store` | PostgreSQL access and migrations | PostgreSQL |
| commands | `check-config`, `calibrate`, `export-csv`, `purge` (run by Dirox or a scheduled job, not HTTP) | store, calc |

### Plugin parts

| Part | Does |
|---|---|
| settings | the settings page: website, language, on/off, engine address and the website's credentials |
| submission | JetFormBuilder's submission hook: hand-off, skipping Send Email, fallback |
| mail | the confirm and call-request routes, the WP-Cron outbox sweep, `wp_mail`, the record of sent e-mail IDs |
| assets | TypeScript: address suggestions on the form fields, the estimate window, one theme per website |

## Routes and APIs

Engine API, version 1. The shapes live in `contract/`; names may still change in the row that builds each route.

| Route | Caller and credentials | Purpose | Row |
|---|---|---|---|
| `GET /health` | Cloud Run, monitoring | liveness | 1 |
| `POST /v1/requests` | plugin server, website secret | hand off a submitted form; returns the request ID and its token | 19 |
| `GET /v1/requests/{id}` | browser, website key + request token | summary, questions, vehicles, legs, constraints, alternatives, estimate (the price only when the website's price display is on) | 25 |
| `POST /v1/requests/{id}/answers` | browser, same | answers to the assistant's questions | 26 |
| `PATCH /v1/requests/{id}` | browser, same | edited values; returns the recalculated request | 27 |
| `POST /v1/requests/{id}/alternatives/{alternative}` | browser, same | apply an alternative | 28 |
| `POST /v1/requests/{id}/confirm` | plugin server, website secret + request token | confirm; returns the e-mails to send | 29 |
| `POST /v1/requests/{id}/call` | plugin server, website secret + request token | call request; returns the e-mail to send | 29 |
| `GET /v1/places/suggestions` | browser, website key | address suggestions, proxied to Google Places | 30 |
| `GET /v1/outbox` | plugin server, website secret | the website's e-mails due now: unconfirmed requests past the delay, retries | 21 |
| `POST /v1/outbox/{id}/sent` | plugin server, website secret | record that an e-mail was sent | 21 |

Plugin entry points: the JetFormBuilder submission hook (row 19); REST routes `POST /wp-json/francebus-quote/v1/requests/{id}/confirm` and `/call`, which the window calls and which call the engine with the website secret, then send the returned e-mails (rows 21, 29); a WP-Cron event every few minutes for the outbox (rows 21, 23); the settings page (row 2).

## Connections

| Service | Used by | For | Limits and notes |
|---|---|---|---|
| Gemini on Vertex AI, EU location | engine | structuring, questions, summary, explanations | Cloud Run service account, no key; the engine refuses a non-EU location |
| Google Places (autocomplete) | engine, as a proxy | address suggestions | free monthly allowance; Google's attribution shown with the suggestions |
| Google Routes | engine | distance and driving time per leg, traffic-unaware | 10,000 free calls a month; route cache |
| Google Geocoding | engine | addresses typed without a suggestion | free allowance |
| JetFormBuilder | plugin | the form, its submission hook, its Send Email as the fallback | versions on both sites: open question 13 |
| Infomaniak mailbox devis@francebus.com | plugin, through `wp_mail` | sending both e-mails | SPF and DMARC in place; DKIM to check in row 21 |
| France Bus CRM | reads the mailbox | detects "Francebus Service" | unchanged; tested on its test mailbox (open question 9) |
| Google Cloud: Cloud Run, Cloud SQL, Secret Manager, Logging and Monitoring, Billing budget | engine in production | hosting, secrets, monitoring, budget alert | France Bus's account, one EU region |

## Data model

PostgreSQL, in the engine. The estimate log is `request` with its `exchange` and `estimate` rows.

| Table | Holds | Personal data | Row |
|---|---|---|---|
| `request` | ID, website, language, status (`open`, `confirmed`, `call_requested`, `sent_unconfirmed`), the submitted form as received, the structured request, submitted / last activity / closed times | yes: name, phone, e-mail, company, addresses, description | 19 |
| `exchange` | the assistant's questions and the visitor's answers, in order, with times | possibly, in answers | 20 |
| `estimate` | inputs snapshot, measured legs (ends, distance, duration, Google or cache, when), vehicles, feasibility and alternatives, breakdown lines, total and rounded price in integer cents, whether the price was shown, tariff configuration version, engine version, time | through its request | 20 |
| `outbox_email` | request, kind (`france_bus`, `client`), flag (`confirmed`, `not_confirmed`, `call`), recipient, subject, text and HTML bodies, due time, sent time, attempts; unique on (request, kind) | yes | 21 |
| `route_cache` | origin and destination keys, distance, duration, fetched and expiry times | no link to a person | 8 |

Not in the database: the tariff configuration (files in `tariff/`); the websites' registry (ID, domains, style, default language, price display switch) as engine configuration, with the website keys and secrets in environment secrets (Secret Manager in production). Retention (row 32) deletes a request with everything attached to it.

## Environments

| Environment | Where | Engine | Database | Websites | Data |
|---|---|---|---|---|---|
| local | developer machine, Docker Compose | container | PostgreSQL container | WordPress with JetFormBuilder in a container; Mailpit catches the e-mails | synthetic |
| testing | Dirox's servers | same image, deployed on each merge to `main` | PostgreSQL | a test WordPress | synthetic |
| staging | Dirox's servers | same image, deployed on demand | PostgreSQL | staging copies of both websites | synthetic; QC and France Bus acceptance |
| production | France Bus's Google Cloud, one EU region | Cloud Run | Cloud SQL for PostgreSQL | francebus.com and francebus-wedding.com at Infomaniak | real |

CI/CD on GitHub Actions: each PR runs lint, tests, the tariff schema check, the calibration run and the Dirox checks; `main` deploys to testing; staging on demand; production through a manual workflow behind a GitHub environment approval, run once France Bus has approved, signing in to France Bus's project with Workload Identity Federation (no key files). Tests never call Google or Gemini: the adapters are tested against recorded responses, and a few live checks run on staging. Which Google Cloud project pays for testing and staging calls: open question 13.

## Decisions

### 1. One engine for both websites; the plugin only displays and passes data
All AI, calculation, e-mail preparation and logging live in the engine. The plugin hands over the form, shows the window, sends the e-mails the engine prepared and falls back to JetFormBuilder. Both websites use the same engine, told apart by their credentials.
- Why: already decided in the charter. Shared hosting limits execution time, cannot keep API keys safe, and would need one copy per site.
- Alternatives: the logic in the WordPress plugin (rejected for those reasons); one engine per website (twice the operations and the calibration, for nothing that differs).
- Decision record needed.

### 2. One repository for the engine, the plugin and the configuration
This repository holds every folder in the layout above, with one `openspec/` and one `okf/` for all of them.
- Why: a ticket that changes the API changes both sides in one PR, against one spec; the contract is shared by import, never copied; the kit's one ticket, one change, one PR holds for every row.
- Alternatives: two repositories, one per codebase (the contract and the specs drift apart, and most features need two PRs); one repository per website (nothing differs per site but its settings).
- Consequence: France Bus owns the plugin code. It receives the built plugin, and if it asks, the history of `wordpress-plugin/` extracted with `git subtree split`. Whether it also owns the engine: open question 15.
- Decision record needed.

### 3. One container image: Dirox's servers before production, Cloud Run in production
The engine ships as one container image, configured by environment variables. Testing and staging run it on Dirox's servers; production runs it on Cloud Run in France Bus's Google Cloud project, in one EU region shared with Cloud SQL and the Vertex AI location.
- Why: the charter sets Cloud Run in France Bus's account and staging on Dirox's servers; one image means staging tests what production runs.
- Alternatives: Cloud Functions (one function per route, hard to run on Dirox's servers); GKE or a virtual machine (more to operate, for about 300 requests a month); staging in France Bus's project (closer to production, against the charter's split).
- Decided in row 35: the region (an EU region where the chosen Gemini model runs; Paris, `europe-west9`, if it does).
- Decision record needed.

### 4. PostgreSQL for requests, the estimate log, the outbox and the route cache
Cloud SQL for PostgreSQL in production, in the engine's region; a PostgreSQL container on Dirox's servers and on developers' machines. Migrations are versioned in `engine/`.
- Why: the same database in every environment (staging on Dirox's servers rules out a Google-only database); transactions and a unique constraint make "one France Bus e-mail per request" a database property; SQL makes the CSV export and the calibration analysis simple.
- Alternatives: Firestore (no fixed cost and a free tier, but only an emulator off Google Cloud, so no real staging on Dirox's servers, and weaker queries for exports); SQLite on Cloud Run (no shared, durable disk); BigQuery for the log (analytics, not the request's transactional state).
- Cost: Cloud SQL has a fixed monthly cost, open question 14.
- Decision record needed.

### 5. The engine in TypeScript on Node.js
Strict TypeScript on the Node.js LTS version current when row 1 starts. The API contract is a set of schemas in `contract/`: one source for input validation, the window's types and Gemini's response schemas. Money in integer cents; dates and times in Europe/Paris. To be confirmed in review (open question 8).
- Why: one language for the engine and the window, so the contract is shared code rather than generated copies; Google's Gen AI SDK and the Maps APIs support it; quick container start on Cloud Run; the plugin's front-end build needs Node tooling anyway.
- Alternatives: Python with FastAPI and Pydantic (strong for calibration analysis and decimal arithmetic, and Pydantic gives Gemini schemas directly, but a third language next to the window's TypeScript and the plugin's PHP, with types generated across the boundary); PHP inside WordPress (ruled out by decision 1).
- Decided in row 1: the HTTP framework, database access and test runner.
- Decision record needed.

### 6. The AI handles language only
Gemini on Vertex AI at an EU location, called with a response schema for each call, its output validated before use. The assistant produces questions, the summary's wording, explanations and proposed field values. A proposed value (a stop read in the description, a corrected address) enters the request only when the visitor accepts or enters it. Prices, distances and dates are never taken from the model: the engine puts its own figures into the summary. The assistant receives the trip fields and the description, not the contact fields (name, e-mail, phone, company), only whether the phone is missing.
- Why: the AI never produces a figure; the same request gets the same price; EU processing, no training and data minimisation for GDPR.
- Alternatives: the consumer Gemini API (no EU commitment; ruled out by the charter); the AI proposes a price that the engine checks (breaks determinism and the rule); free-text answers parsed by the engine (fragile).
- Decided in row 14: the model, the prompts, and what the engine does when Gemini fails or is slow. Decided in row 15: how the summary carries the engine's figures.
- Decision record needed.

### 7. Deterministic pricing from versioned tariff files
The calculation core is pure: from the structured request, the measured legs, a tariff configuration version and a reference time, it returns vehicles, feasibility, alternatives, price and breakdown, with no clock, network or randomness inside. The tariff configuration is versioned files in `tariff/`, checked against a schema in CI and at start-up. A new version is a PR that changes no code, runs the calibration set in CI and deploys like a release. Each estimate stores its legs and the configuration version, so any logged price can be computed again. Legs come from Routes with traffic-unaware routing, so a leg's time does not depend on when it was asked.
- Why: the same request gets the same price, every price is explained; prices change without redevelopment, with a reviewed history; calibration becomes a test.
- Alternatives: configuration in database tables changed by script (no review, no history, no calibration in CI); a Google Sheet France Bus edits (a back-office, out of scope); configuration in a Cloud Storage bucket read at run time (no deploy needed, but its versions and reviews live outside the repo); traffic-aware routing (times closer to the day, but a price that changes with the hour it is asked, and a costlier Routes tier).
- Decided in row 5: the file format and the schema.
- Decision record needed.

### 8. How the browser, the websites and the engine authenticate
- The browser calls the engine directly for address suggestions and the estimate window, with the website's public key. The engine checks the call's Origin against the website's domains, applies rate limits, and for a request requires the token it issued at Submit. Window responses carry no contact data.
- The plugin's PHP calls the engine server to server for the hand-off, the confirmation, call requests and the outbox, with the website's secret, which stays on the WordPress server.
- Google Maps keys and Vertex AI access exist only on the engine (Secret Manager and the Cloud Run service account in production). Address suggestions go through the engine.
- Why: API keys stay on the engine; a key in a page is public, so it is bound to origins and opens no personal data; proxying every keystroke through shared-hosting PHP would be slow; the e-mails hold personal data and are read server to server only.
- Alternatives: every call through WordPress (hides the engine, but adds PHP latency to each keystroke and ties up shared-hosting workers); a Google browser key restricted by referrer (breaks the rule); a single key per website, as the charter sketches (a key the browser sees cannot also guard the outbox, so each website gets a public key and a secret).
- Decided in row 2: where the plugin stores its secret (settings or `wp-config.php`). Decided in row 4: key format, rotation and rate values.
- Decision record needed.

### 9. E-mails through an engine outbox, sent by the plugin
The engine writes each e-mail into the outbox; a unique constraint keeps one France Bus e-mail per request. The plugin of the request's website sends it with WordPress's mailer, through the site's Infomaniak mailbox, from devis@francebus.com, then records it as sent with the engine. Confirmation and call requests go browser, plugin, engine, and the plugin sends the returned e-mails in the same call. A WP-Cron sweep every few minutes sends the rest: unconfirmed requests past the delay, and retries. The plugin keeps the IDs it sent, so a lost "sent" record never causes a second e-mail.
- Why: the charter has the plugin send through the Infomaniak mailbox; pulling needs no inbound endpoint on shared hosting; the outbox makes "never lost, never twice" a property of the data.
- Alternatives: the engine sends through Infomaniak's SMTP itself (simplest timing, no WP-Cron, but the mailbox password moves to Google Cloud, against the charter's choice; kept as the way out if WP-Cron proves unreliable); the engine pushes e-mails to a WordPress endpoint (inbound calls to shared hosting, a webhook to secure); SendGrid (dropped by the PM).
- Consequence: unconfirmed e-mails leave when WP-Cron runs, which needs site traffic or an Infomaniak cron task calling `wp-cron.php` (row 21).
- Decision record needed.

### 10. The plugin takes over at JetFormBuilder's submission hook; JetFormBuilder's e-mail is the fallback
The plugin uses JetFormBuilder's server-side submission hook. When the engine takes the form, the plugin stops JetFormBuilder's Send Email action for that submission and returns the request ID and token to the page, which opens the window. When the engine fails or does not answer in the configured time, the plugin steps aside and JetFormBuilder sends its e-mail as today. The two form upgrades are built in JetFormBuilder, and each site's form is exported to `jetformbuilder/` for review.
- Why: the charter keeps the form and connects through JetFormBuilder's hook; the fallback works even when the page's script fails; one e-mail per request.
- Alternatives: catching the submit in the browser before JetFormBuilder (the fallback then depends on the script running); replacing the form with the plugin's own (against the charter); letting JetFormBuilder always send and the engine send a second e-mail (two e-mails per request).
- Decided in row 19, after a spike on the sites' JetFormBuilder version: which hook, how Send Email is skipped, and the time limit.
- Decision record needed.

### 11. The estimate window in TypeScript with Preact, inside a Shadow DOM
The window and the suggestion lists are TypeScript with Preact, built by the plugin's front-end build into one script, mounted inside a Shadow DOM and themed per website with CSS custom properties. To be confirmed in review (open question 8).
- Why: the window holds real state (questions, edits, instant recalculation, alternatives) that a component library keeps simple; Preact is small; the Shadow DOM keeps the site's theme and the window's styles from breaking each other, so the form keeps its look; TypeScript shares the contract with the engine.
- Alternatives: plain TypeScript (lighter, more hand-written state and rendering); React (the same model, several times heavier); WordPress's own React bundle (loads React on a public page); jQuery (probably on the page already, but no state model).
- Decision record needed.

### Left to the backlog rows

Each is recorded by its row's change if it turns out costly to reverse.

| Choice | Decided in |
|---|---|
| HTTP framework, database access, test runner, Node.js version | row 1 |
| PHP and WordPress versions supported, plugin build and release, where the website secret is stored | row 2 |
| Deployment on Dirox's servers, staging copies of the websites | row 3 |
| Key format, rotation, rate-limit values | row 4 |
| Tariff file format and schema | row 5 |
| Route cache key and lifetime, within Google Maps' terms | row 8 |
| How a request matches a fixed route | row 11, with discovery |
| Gemini model, prompts, behaviour when Gemini fails or is slow | row 14 |
| How the summary carries the engine's figures | row 15 |
| JetFormBuilder hook, skipping Send Email, the Submit time limit | row 19 |
| Outbox sweep interval, the cron task on Infomaniak, DKIM | row 21 |
| Whether the France Bus e-mail carries a copy of the client e-mail | row 22, with Fabrice Dessaint |
| Places session tokens and attribution | row 30 |
| CSV columns and how the export is run | row 32 |
| How the test set is anonymised | row 33 |
| Google Cloud region, infrastructure as code, minimum instances, Vertex AI data caching | row 35 |
| Monitoring tools and alert channel | row 36 |

### Design rules

- Dates and times in Europe/Paris; money in integer cents; rounding only on the visitor-facing price.
- No personal data in the repository: the calibration set keeps no names, e-mails, phones or companies, and private addresses become a town or a nearby public place, or their measured legs.
- Testing and staging hold synthetic data only. Application logs carry request IDs, never personal data.
- The engine refuses to start on an invalid configuration: a tariff version that fails its schema, a non-EU Vertex AI location.
- Every response or e-mail that can show a price follows the website's price display switch (row 13).

## Risks / Trade-offs

- [Discovery values arrive late] → everything reads configuration and tests use fixtures; only row 34 needs the signed rules.
- [JetFormBuilder's hook cannot skip Send Email cleanly] → a spike first in row 19; fallback: a condition on the Send Email action that reads a field the plugin sets.
- [WP-Cron runs only with traffic] → an Infomaniak cron task calling `wp-cron.php`; the way out in decision 9.
- [Google Maps' terms limit caching route data, and the log keeps leg figures] → row 8 checks the terms and sets the cache lifetime; row 20 keeps only what the terms allow, or the figures the engine derived.
- [Gemini takes several seconds at Submit] → the window opens at once in a waiting state; each call has a time limit (rows 14, 25).
- [Gemini's output varies] → response schemas, validation, no figures from it; the test set also serves to check the questions.
- [Deliverability: SPF and DMARC exist, DKIM is not mentioned] → check DKIM signing in row 21; Gmail and Outlook inbox tests before go-live (row 37).
- [How the CRM reads the e-mail is unknown] → a raw copy of today's e-mail and the CRM's test mailbox early (open question 9); the template is compared with the copy, label by label.
- [Bots or spam use up the Gemini and Maps allowances] → rate limits, the budget alert, usage monitoring (row 36).
- [Cloud Run cold starts slow the first request] → minimum instances weighed against cost in row 35.
- [A visitor confirms after the "not confirmed" e-mail went out] → open question 5.
- [Dirox's servers may be outside the EU] → synthetic data only on testing and staging (open question 13).

## Migration Plan

1. Install the plugin on both sites switched off: the form behaves as today, with its two upgrades.
2. Keep the price display off for both websites.
3. On staging: acceptance testing, the CRM test mailbox, Gmail and Outlook inbox tests.
4. Deploy to production once France Bus approves; switch the plugin on, one website at a time.
5. Switch the price display on once France Bus approves the calibration results.

Rollback: switching the plugin off returns the form to today's behaviour at once; the engine goes back to its previous Cloud Run revision; a tariff change goes back to the previous configuration version.

## Open Questions

See `proposal.md`, Open questions. Questions 5, 8, 9, 13, 14 and 15 touch this design; none of them changes the backlog's structure.
