# Tasks

The kickoff writes no code: these are the foundation tasks, done after the plan is approved. Each concept follows the `dirox-core:okf` skill (template, frontmatter, `owner` the email the reviewers named); every task is checked by `node .github/scripts/okf-validate.mjs okf` passing, unless it says otherwise.

## 1. Architecture

- [ ] 1.1 Write `okf/architecture/overview.md` from `design.md`: overview, repository layout, engine modules, plugin parts, data flow, connections, environments and deployment; it cites the decision files of group 2
- [ ] 1.2 Write `okf/architecture/tech-stack.md`: TypeScript on Node.js, PostgreSQL, Preact in a Shadow DOM, PHP for the plugin, with the versions the review settled and a TODO for those rows 1 and 2 set (Node.js, PHP, WordPress, JetFormBuilder)

## 2. Decisions

One file per decision marked in `design.md`, each with `Change: openspec/changes/archive/fbq-1-kickoff-<YYYY-MM-DD>/`.

- [ ] 2.1 Write `okf/decisions/FBQ-1-one-engine-thin-plugin.md` (decision 1)
- [ ] 2.2 Write `okf/decisions/FBQ-1-one-repository.md` (decision 2)
- [ ] 2.3 Write `okf/decisions/FBQ-1-one-container-image.md` (decision 3)
- [ ] 2.4 Write `okf/decisions/FBQ-1-postgresql.md` (decision 4)
- [ ] 2.5 Write `okf/decisions/FBQ-1-engine-typescript.md` (decision 5, as the review settled it)
- [ ] 2.6 Write `okf/decisions/FBQ-1-ai-language-only.md` (decision 6)
- [ ] 2.7 Write `okf/decisions/FBQ-1-deterministic-pricing.md` (decision 7)
- [ ] 2.8 Write `okf/decisions/FBQ-1-engine-authentication.md` (decision 8)
- [ ] 2.9 Write `okf/decisions/FBQ-1-email-outbox.md` (decision 9)
- [ ] 2.10 Write `okf/decisions/FBQ-1-jetformbuilder-hook-fallback.md` (decision 10)
- [ ] 2.11 Write `okf/decisions/FBQ-1-estimate-window-preact.md` (decision 11, as the review settled it)

## 3. Domain terms

- [ ] 3.1 Write `okf/domain/quote-request.md` (devis: what a request holds, from Submit to sent)
- [ ] 3.2 Write `okf/domain/route-type.md` (One way, Round trip, Multi-destination)
- [ ] 3.3 Write `okf/domain/stop.md` (Stages, Other steps, étapes; "Stages" as the e-mail field the CRM reads)
- [ ] 3.4 Write `okf/domain/fixed-route.md` (trajet forfaitaire)
- [ ] 3.5 Write `okf/domain/surcharge.md` (majorations: night, Sunday, holidays; and the fees)
- [ ] 3.6 Write `okf/domain/working-day.md` (amplitude)
- [ ] 3.7 Write `okf/domain/operating-rules.md` (driving limits, breaks, daily rest, minimum notice; values from discovery)
- [ ] 3.8 Write `okf/domain/depot.md` (dépôt; its effect on price still open)
- [ ] 3.9 Write `okf/domain/vehicle-category.md` (sedan, minivan or VTC, minibus, midibus, coach, double-decker; capacities from discovery)
- [ ] 3.10 Write `okf/domain/indicative-estimate.md` (TTC, "À partir de XXXX € TTC", non-binding, subject to availability)
- [ ] 3.11 Write `okf/domain/francebus-service-email.md` (the request e-mail the CRM detects by its subject)
- [ ] 3.12 Write `okf/domain/unconfirmed-request.md` (sent after the delay, flagged "not confirmed")
- [ ] 3.13 Write `okf/domain/call-request.md` ("prefer to talk to us")
- [ ] 3.14 Write `okf/domain/calibration.md` (the test set of past quotes, the tolerance, the price display switch it unlocks)

## 4. Integrations

- [ ] 4.1 Write `okf/integrations/jetformbuilder.md` (the form, the submission hook, Send Email as the fallback, the exported forms in `jetformbuilder/`)
- [ ] 4.2 Write `okf/integrations/google-maps-platform.md` (Places, Routes, Geocoding: what each is for, allowances, caching terms, keys only on the engine)
- [ ] 4.3 Write `okf/integrations/vertex-ai-gemini.md` (EU location, business terms, no training, response schemas, what is never sent)
- [ ] 4.4 Write `okf/integrations/infomaniak-mailbox.md` (devis@francebus.com, sending through `wp_mail`, SPF and DMARC, DKIM to check)
- [ ] 4.5 Write `okf/integrations/france-bus-crm.md` (reads the "Francebus Service" e-mail, unchanged, its test mailbox)
- [ ] 4.6 Write `okf/integrations/google-cloud.md` (France Bus's account: Cloud Run, Cloud SQL, Secret Manager, budget alert, who approves production)

## 5. Data

- [ ] 5.1 Write `okf/data/request.md` (with its questions and answers; personal data named)
- [ ] 5.2 Write `okf/data/estimate.md` (the estimate log entry: legs, breakdown, configuration version; retention and CSV export)
- [ ] 5.3 Write `okf/data/outbox-email.md` (one France Bus e-mail per request, sent and retried by the plugin)
- [ ] 5.4 Write `okf/data/route-cache.md`
- [ ] 5.5 Write `okf/data/tariff-configuration.md` (versioned files in `tariff/`, schema, who changes them and how)
- [ ] 5.6 Write `okf/data/website.md` (the registry: domains, style, default language, price display switch; where its credentials are kept)

## 6. Links and log

- [ ] 6.1 Add `Knowledge:` links from the requirements of this change's specs to the concepts of groups 3 to 5, and check `openspec validate "fbq-1-kickoff" --strict` still passes
- [ ] 6.2 Add the kickoff's line to `okf/log.md`, run `node .github/scripts/okf-validate.mjs okf --write` and fix every error

## 7. The project's CLAUDE.md and CODEOWNERS

- [ ] 7.1 Fill "Project" in `.claude/CLAUDE.md`: what the system is in one sentence, the stack with versions (TODO where rows 1 and 2 set them)
- [ ] 7.2 Fill "Commands" in `.claude/CLAUDE.md` with what the plan says, and a TODO naming rows 1 and 2 for the commands that do not exist yet
- [ ] 7.3 Fill "Project rules" in `.claude/CLAUDE.md`: folder layout and naming, things never to touch, and what is tricky here (no figure from the AI, today's e-mail format, one e-mail per request, plugin off and fallback, price display switch, EU data and no personal data in the repository); check the file stays under 150 lines with `wc -l`
- [ ] 7.4 Fill `.github/CODEOWNERS` with the people or teams the review named, and check each line names a GitHub user or team

## Workflow follow-up

- Archive the change once every task above is ticked (`/dirox-techlead:kickoff` step 8), and rename the archived folder to `fbq-1-kickoff-<YYYY-MM-DD>`.
- After the merge, `/dirox-core:jira create` drafts the backlog rows marked `new`.
