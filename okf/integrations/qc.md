---
type: Integration
title: QC
description: How QC gets a build of each PR to test - a preview, or the commands to run it locally; read by /dirox-dev:implement (the PR's "How to test"), /dirox-core:next and QC
owner: vincent.ledreau@dirox.net
status: stable
tags: [qc, testing]
---
# QC

<!-- Saved as okf/integrations/qc.md by /dirox-techlead:setup-project or /dirox-qc:test-cases setup.
     The commands read the backticked values; keep the "- Label: `value`" form. The QC reviewers,
     whose approval every PR that changes code needs, are in .github/qc-reviewers, where CI reads
     them; the Jira statuses are in okf/integrations/jira.md. -->

## PR builds
- PR build: `local`
- Preview: `none`
- Local: `the Install and Run lines of .claude/CLAUDE.md (not written yet: the app does not exist)`

QC tests every PR that changes code on this build before it merges, then approves it. There is no way around QC: a project with no preview runs the PR locally.

## Conventions
- `none yet`
