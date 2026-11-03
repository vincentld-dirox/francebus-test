Jira: FBQ-1
Size: big

# Proposal: France Bus Smart Quote Form V1 (kickoff)

Planned from the charter in the description of epic FBQ-1 (label `src:intent`). The charter stays in Jira; this change does not copy it.

## Why

France Bus, a coach operator serving businesses and private clients across mainland France and Monaco, gets about 190 to 290 quote requests a month through a JetFormBuilder form on francebus.com and francebus-wedding.com. One person copies each request by hand into a Gemini Gem to get a price base, and often has to contact the client again because the request is incomplete or wrong; stops and addresses are free text that cannot be measured.

V1 completes and prices each request on the form page itself: an AI assistant asks the visitor for what is missing, a calculation engine prices the trip from France Bus's own tariff grid and driving rules (never from the AI), and France Bus receives a ready-to-process, pre-priced request in the e-mail its CRM already reads. Success is measured on four points: every request sent holds trip type, passengers, exact addresses, stops, dates and times and contact details; on a test set of about 50 past requests the estimates match the prices France Bus quoted within a tolerance agreed in discovery; the CRM detects 100% of the new e-mails; the form keeps its look on both sites, desktop and mobile.

## What Changes

In scope (V1, all on the quote form page of both websites, in French and English, desktop and mobile):
- Two upgrades of the existing JetFormBuilder form, made in JetFormBuilder itself: a stop list for multi-destination trips and a dedicated return date and time for round trips.
- Google address suggestions in the address fields, served through the engine.
- A new **France Bus AI Engine** (one service for both websites): request understanding with Gemini on Vertex AI (EU), vehicle selection, itinerary and distances from Google Maps, driving-rule checks with alternatives, deterministic pricing from a tariff configuration, the e-mails, and an estimate log.
- A new **WordPress plugin** that hands the submitted form to the engine, shows the estimate window, sends the e-mails through the site's Infomaniak mailbox, and falls back to today's JetFormBuilder e-mail when the engine cannot be reached.
- Environments: local, testing and staging on Dirox's servers; production on Cloud Run in France Bus's own Google Cloud account, EU region; CI/CD and monitoring until the end of the 30-day launch support.

Out of scope:

| Out | What happens instead |
|---|---|
| Website assistant (chatbot, V2) | Its own epic later |
| Vehicle and driver availability | The estimate says "subject to availability, confirmed by our team" |
| Back-office (tariff editing, log viewing, request management) | Dirox maintains the tariff configuration; the log is exported as CSV on request |
| Online booking, payment, contracts | The estimate stays indicative; France Bus confirms the final quote as today |
| Changes to the CRM | The CRM keeps receiving the same e-mail |
| Website redesign, content, SEO | Only the form page changes |
| Other languages | French and English only |
| Monitoring, support, recalibration after the 30-day launch support | A separate maintenance agreement |

## Capabilities

### New Capabilities
- `quote-form`: the JetFormBuilder quote form and its two upgrades (stop list, return date), which keep working with the plugin off.
- `address-suggestions`: Google suggestions while the visitor types an address, in the form and in the estimate window.
- `form-submission`: handing a submitted form to the engine, and sending it as today when the engine cannot be reached.
- `engine-access`: the single secure entry to the engine for both websites: website keys, origin checks, rate limiting, request tokens, Google credentials kept on the engine.
- `request-understanding`: the AI assistant that structures the request, asks targeted questions, integrates the answers and rephrases the request, without producing any figure.
- `estimate-window`: what the visitor sees and does after Submit: summary, questions, edits with recalculation, alternatives, confirmation, "prefer to talk to us".
- `vehicle-selection`: choosing the vehicle or vehicles from the number of passengers.
- `itinerary-routing`: the itinerary leg by leg, with distance and driving time from Google Maps.
- `driving-rules`: checking a schedule against the operating rules and computing alternatives when it does not fit.
- `pricing`: the deterministic, explainable price from the tariff configuration, the price display switch, and calibration against past quotes.
- `france-bus-email`: the one e-mail per request that France Bus's CRM reads, in today's format.
- `client-email`: the summary e-mail the visitor receives after confirming.
- `email-delivery`: how the e-mails leave the system: from devis@francebus.com, through the website's mailbox, never lost and never sent twice.
- `estimate-log`: the record of every form and estimate, its retention and its CSV export.
- `plugin-settings`: the WordPress settings page: website (and so the window's style), language, and the on/off switch with what "off" means.

### Modified Capabilities
None: the project has no specs yet.

## Impact

- **Code**: two new deployables in this repository, the engine (`engine/`) and the WordPress plugin (`wordpress-plugin/`), plus the tariff configuration and the calibration set; see `design.md`.
- **Websites**: the JetFormBuilder form of francebus.com and francebus-wedding.com is edited in JetFormBuilder; the plugin is installed on both sites. Hosting at Infomaniak is unchanged and needs no server change.
- **France Bus's Google Cloud account**: Cloud Run, Cloud SQL for PostgreSQL, Secret Manager, Vertex AI and the Google Maps Platform APIs, billed there with a budget alert. Cloud SQL adds a fixed monthly cost on top of the usage-based Gemini and Maps bills (open question 14).
- **Mail**: the plugin sends from devis@francebus.com through France Bus's Infomaniak mailbox. The CRM is unchanged; it is tested on a test mailbox connected to it.
- **People**: Fabrice Dessaint (France Bus) validates the Pricing and Operating Rules document and the final results; a France Bus contact person (not named yet) validates the rules and runs acceptance testing; France Bus approves the production deployment and the go-live. At Dirox: Chinh Phan (executive sponsor), Laurent Beoustes (project manager), Vincent Le Dreau (AI technical lead: engine, Gemini, Google Maps), Tuan Nguyen (WordPress: plugin and form). This kickoff is approved by the Tech lead, the Architect and the PM.
- **Personal data**: name, phone, e-mail and company, with trip details, are processed and stored in the EU, and are not used to train Google's models.

## Backlog

All rows are new: FBQ-1 has no child tickets. `#` numbers are used for dependencies until the tickets have keys. "Discovery" is the France Bus workshops phase that ends with the Pricing and Operating Rules document signed off and the test set of past quotes delivered; it is not a ticket here.

| # | Jira | Title | Size | Depends on | Capability |
|---|---|---|---|---|---|
| 1 | new | Engine service skeleton: TypeScript service, health check, configuration loading, PostgreSQL and migrations, container image, local Docker Compose, test and lint in CI | big | - | tooling |
| 2 | new | WordPress plugin skeleton with its settings page (website, language, on/off, engine connection) and the window's front-end build | big | - | plugin-settings |
| 3 | new | Testing and staging on Dirox's servers, with staging copies of both websites, deployed from CI | big | 1, 2 | tooling |
| 4 | new | Engine access: website keys, origin checks, rate limiting, request tokens | big | 1 | engine-access |
| 5 | new | Tariff configuration: file format, schema validation, versioning, test fixtures | big | 1 | pricing |
| 6 | new | Vehicle selection from the number of passengers, several vehicles above the largest capacity | normal | 5 | vehicle-selection |
| 7 | new | Itinerary legs for each route type, with distance and driving time from Google Maps Routes and the speed coefficient | big | 5 | itinerary-routing |
| 8 | new | Geocoding of addresses typed without a suggestion, and the route cache | big | 7 | itinerary-routing |
| 9 | new | Driving-rule checks per leg and per day (driving time, breaks, working day, rest, minimum notice) | big | 7 | driving-rules |
| 10 | new | Alternatives for a schedule that does not fit: break, second driver, other departure time, larger vehicle | normal | 6, 9 | driving-rules |
| 11 | new | Base price (fixed route, per km, per duration), step-by-step breakdown and rounding | big | 6, 7 | pricing |
| 12 | new | Surcharges (night, Sunday, holidays) and fees (tolls, parking, driver meals and lodging) | normal | 11 | pricing |
| 13 | new | Price display switch per website, off until France Bus approves the calibration | normal | 4, 12 | pricing |
| 14 | new | Request understanding: structured request and targeted questions with Gemini on Vertex AI (EU) | big | 1 | request-understanding |
| 15 | new | Answers integrated, plain-words summary with the engine's figures, phone asked once | normal | 11, 14 | request-understanding |
| 16 | new | The assistant explains why a schedule does not fit | normal | 10, 15 | driving-rules |
| 17 | new | Quote form: stop list for multi-destination trips, stops written under "Stages" (both sites) | normal | 3 | quote-form |
| 18 | new | Quote form: return date and time for round trips (both sites) | normal | 17 | quote-form |
| 19 | new | Submit hands the form to the engine; today's JetFormBuilder e-mail when the engine cannot be reached | big | 2, 4 | form-submission |
| 20 | new | Estimate log: every form and every estimate, with questions, answers, breakdown and status | big | 11, 14, 19 | estimate-log |
| 21 | new | E-mail delivery: engine outbox, sent by the plugin through the site's mailbox, retried without duplicates | big | 4, 19 | email-delivery |
| 22 | new | France Bus e-mail in today's exact format with the estimate block, sent on confirmation | big | 15, 21 | france-bus-email |
| 23 | new | Unconfirmed requests sent after the delay, and call requests, flagged in the France Bus e-mail | normal | 22 | france-bus-email |
| 24 | new | Client e-mail on confirmation, in the visitor's language | normal | 13, 15, 21 | client-email |
| 25 | new | Estimate window: summary after Submit (request, vehicles, legs, constraints, indicative estimate), language, AI notice, both sites' styles | big | 9, 13, 15, 19 | estimate-window |
| 26 | new | Estimate window: the assistant's questions, each with the right input | normal | 25 | estimate-window |
| 27 | new | Estimate window: editable values with instant recalculation | normal | 26 | estimate-window |
| 28 | new | Estimate window: alternatives selectable in one click | normal | 16, 27 | estimate-window |
| 29 | new | Estimate window: confirmation step and "prefer to talk to us" | normal | 23, 24, 28 | estimate-window |
| 30 | new | Address suggestions in the form fields, through the engine | big | 2, 4 | address-suggestions |
| 31 | new | Address suggestions in the estimate window's address inputs | normal | 27, 30 | address-suggestions |
| 32 | new | Estimate log retention and CSV export | normal | 20, discovery (retention) | estimate-log |
| 33 | new | Calibration run on the anonymised test set of past quotes | normal | 10, 13, discovery (test set) | pricing |
| 34 | new | Tariff configuration V1 from the signed rules, passing the test set within the tolerance | normal | 33, discovery (sign-off) | pricing |
| 35 | new | Production on France Bus's Google Cloud: Cloud Run, Cloud SQL, secrets, EU region, budget alert, deployment after approval | big | 3 | tooling |
| 36 | new | Monitoring of errors, response times and API usage, with alerts to Dirox | normal | 35 | tooling |
| 37 | new | Go-live runbook and checks: CRM detection on the test mailbox, Gmail and Outlook inboxes, both sites | normal | 29, 31, 32, 34, 36 | tooling |

Split from the charter's features so that each ticket fits one PR: `estimate-window` (rows 25 to 29), `pricing` (5, 11, 12, 13, 33, 34), `driving-rules` (9, 10, 16), `itinerary-routing` (7, 8), `request-understanding` (14, 15), `quote-form` (17, 18), `france-bus-email` (22, 23), `address-suggestions` (30, 31), `estimate-log` (20, 32). Rows on the same capability are chained by dependencies, so no two of them change its spec at the same time. Rows 28 and 29 add separate requirements and could run side by side if two people are free; they are chained to keep the window's code in one line of work.

Phases: rows 1 to 3 set up both codebases and the environments. Rows 4 to 16 and 19 to 24 are the booking engine phase (rows 19 and 21 include their plugin side); it ends with row 34, when the engine passes the test set, and rows 32 to 34 wait for discovery's results. Rows 17, 18 and 25 to 31 are the WordPress plugin phase, ending feature complete on staging; they can start alongside the engine rows once rows 2, 3 and 4 are done. Rows 35 to 37 are acceptance testing and launch.

## Open questions

1. **The values discovery sets.** The price tolerance for the test set (per quote or a share of the quotes), the rounding of the estimate, whether grid prices are incl. or excl. VAT (and the rate), the estimate log's retention period (deleted or anonymised after it), the coach and minibus speed coefficient (one or per vehicle category), the rule between per-km and per-duration pricing, which public holidays count (France, Monaco). All are configuration in the specs. Who: Fabrice Dessaint. Blocks: rows 7, 11, 12, 32, 34.
2. **What happens below the minimum notice?** No price, a contact prompt, or a warning. The specs only say the engine detects it. Who: Fabrice Dessaint. Blocks: rows 9, 25.
3. **How do depots affect the price?** For example empty legs from the nearest depot; no rule uses them yet. Who: Fabrice Dessaint. Blocks: rows 7, 11.
4. **The real vehicle categories, their capacities, and how vehicles are combined for large groups.** The sources' bands do not match the fleet on the website. Who: Fabrice Dessaint. Blocks: rows 6, 34.
5. **After how long is an unconfirmed request sent, and what if the visitor confirms after that?** The delay is not set; a confirmation after the "not confirmed" e-mail went out would break "one e-mail per request" (options: the request closes at the delay, or the delay counts from the visitor's last action). Who: Fabrice Dessaint (delay), Laurent Beoustes (behaviour). Blocks: rows 23, 29.
6. **Who is the France Bus contact person for the rules and the acceptance tests?** Who: France Bus. Blocks: acceptance (row 37).
7. **Which past examples become the first scenarios?** No worked example with real values yet; the test set comes from discovery. Who: Fabrice Dessaint. Blocks: rows 6, 9, 11, 33.
8. **Confirm the stack.** Proposed: the engine in TypeScript on Node.js (alternative: Python with FastAPI), the estimate window in TypeScript with Preact inside a Shadow DOM; see `design.md` decisions 5 and 11. Who: Vincent Le Dreau, with Tuan Nguyen. Blocks: the foundation (CLAUDE.md, tech stack) and rows 1, 2.
9. **Today's e-mail, exactly.** A raw copy (with headers) of today's "Francebus Service" e-mail from each website and each language; whether today's sender is devis@francebus.com and how WordPress sends mail today; whether the CRM reads the field labels or only the subject; where the new return date and time go among the labels; the CRM's name and its test mailbox. Who: Tuan Nguyen with France Bus. Blocks: rows 17, 18, 21, 22.
10. **Is the wedding site's form the same as francebus.com's?** The charter lists only francebus.com's fields. Who: Tuan Nguyen. Blocks: rows 17, 19, 22.
11. **What do visitors see while price display is off?** The window without a price line, or the plugin left off until France Bus approves the calibration; and does the plugin run in production before that, so the log collects real forms? Who: Laurent Beoustes with France Bus. Blocks: rows 13, 25, 37.
12. **"Prefer to talk to us … with no price shown": where?** In the France Bus e-mail, on the visitor's screen, or both; and does the visitor get a client e-mail then? Who: Laurent Beoustes. Blocks: rows 23, 24, 29.
13. **Environments and access.** Where Dirox's testing and staging servers are (EU or not: they hold synthetic data only either way); which Google Cloud project pays for Gemini and Maps calls from testing and staging; when Dirox gets access to France Bus's Google Cloud account; who sets up the staging copies of both websites; the WordPress, PHP and JetFormBuilder versions (and any JetFormBuilder add-ons) on both sites. Who: Laurent Beoustes, Tuan Nguyen, France Bus. Blocks: rows 2, 3, 35.
14. **A fixed monthly database cost.** Cloud SQL (decision 4) costs a small fixed amount each month, unlike Firestore's free tier; France Bus should accept it. Who: Laurent Beoustes with France Bus. Blocks: row 35.
15. **Code ownership.** France Bus owns the plugin code: does it also own the engine code, and how does it receive the plugin (release zip or a repository of its own)? This affects decision 2 (one repository). Who: Laurent Beoustes. Blocks: rows 2, 35.
16. **What does the optional "data storage consent" cover?** If a visitor leaves it unticked, may the estimate log keep their personal data? Who: France Bus (with Laurent Beoustes). Blocks: row 20.

## Criteria coverage

The epic's criteria are the charter's success measures and the rules that cannot break; each is covered by a requirement in this change's specs.

| # | Criterion (WHEN / THEN) | Source |
|---|---|---|
| 1 | WHEN a request reaches France Bus THEN it holds trip type, passengers, exact addresses, stops, dates and times and contact details | Success measure; `request-understanding` "Missing or unclear information asked", `quote-form` "Stop list for multi-destination trips" |
| 2 | WHEN the engine prices the test set of past quotes THEN the estimates match the quoted prices within the agreed tolerance | Success measure; `pricing` "Calibration against past quotes" |
| 3 | WHEN the new system sends its e-mails to the CRM's test mailbox THEN the CRM detects all of them | Success measure; `france-bus-email` "Today's e-mail format kept"; checked at go-live (row 37) |
| 4 | WHEN the form is opened on either site, desktop or mobile THEN it keeps its look | Success measure; `quote-form` "Form keeps its look" |
| 5 | WHEN anything shows a price, a distance or a date THEN it comes from the engine, never from the AI | Rule; `request-understanding` "No figure from the AI" |
| 6 | WHEN France Bus has not approved the calibration THEN no price is shown to visitors | Rule; `pricing` "Price display switch" |
| 7 | WHEN the same request is priced again THEN it gets the same price, explained step by step | Rule; `pricing` "Same request, same price", "Step-by-step breakdown" |
| 8 | WHEN France Bus receives a request THEN the e-mail keeps today's sender, subject, labels and order | Rule; `france-bus-email` "Today's e-mail format kept" |
| 9 | WHEN the engine cannot be reached, or the visitor never confirms THEN the request still reaches France Bus | Rule; `form-submission` "Fallback to today's e-mail", `france-bus-email` "Unconfirmed request still sent" |
| 10 | WHEN a request is sent THEN France Bus gets one e-mail for it, not two | Rule; `france-bus-email` "One e-mail per request", `email-delivery` "Delivery retried without duplicates" |
| 11 | WHEN an estimate is shown THEN it is marked indicative, non-binding and subject to availability | Rule; `estimate-window` "Estimate marked indicative", `client-email` "Indicative wording in the e-mail" |
| 12 | WHEN the plugin is switched off THEN the form and its two upgrades keep working | Rule; `quote-form` "Upgrades work without the plugin", `plugin-settings` "On/off switch" |
| 13 | WHEN the pages are loaded THEN no Google or Gemini credential reaches the browser or the websites | Rule; `engine-access` "Credentials stay on the engine" |
| 14 | WHEN personal data is processed or stored THEN it stays in the EU and is not used for model training | Rule; `request-understanding` "AI processing in the EU", `estimate-log` "Log stored in the EU" |

Considered, not needed:
- Availability of vehicles and drivers: out of scope; the estimate says "subject to availability".
- Back-office screens and permissions beyond the WordPress administrator: out of scope; Dirox maintains the configuration.
- Performance thresholds: none stated ("instant recalculation" has no number); response times are monitored (row 36).

Open questions: see above.

## Not in Jira yet

Every scenario in this change's specs. They reach Jira as the acceptance criteria of the backlog tickets, which `/dirox-core:jira create` drafts after this change is merged; FBQ-1 itself keeps none.

## Jira acceptance criteria

Read from FBQ-1 on 2026-11-03 08:42

None in Jira
