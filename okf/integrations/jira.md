---
type: Integration
title: Jira
description: How this project's Jira is set up - project key, where acceptance criteria live, issue types, the status for each stage, the dependency link type and the ticket labels; read by /dirox-core:jira and the other commands
owner: vincent.ledreau@dirox.net
status: stable
resource: https://dirox.atlassian.net/browse/FBQ
tags: [jira]
---
# Jira

<!-- Saved as okf/integrations/jira.md by /dirox-project:set-up-repo or /dirox-core:jira record-config, in this section order. The commands
     read it instead of guessing. Run /dirox-core:jira record-config again when the Jira setup changes: it keeps these values
     and fills only what is missing. -->

## Project
- Site: `dirox.atlassian.net` (cloud ID `95d0c454-e898-47cf-ae79-94d5aee722c2`)
- Project key: `FBQ`
- Charter epic: `FBQ-1`

## Acceptance criteria
- Where: `description, under the heading "Acceptance criteria"`
- Format: `bullet list`

## Issue types
| Use | Issue type |
|---|---|
| Feature ticket | `Story` |
| Small fix | `Task` |
| Bug found by QC | `Bug` |
| Group of tickets | `Epic` |

## Status for each stage
| Stage | Jira status |
|---|---|
| Ticket ready to start | `To Do` |
| Being built | `In Progress` |
| QC testing (optional) | `none` |
| PR ready for review | `In Progress` (the workflow has no review status) |
| Merged | `Done` (moved by the Jira automation in the kit's `jira-done-rule.md`, or by people) |

## Dependencies
- Link type: `Blocks` (outward `blocks`, inward `is blocked by`)

## Ticket labels
<!-- The ticket contract (the jira skill). Labels by default, so JQL can sort and filter on them.
     A team that already uses a component or a custom field for one of them names it instead. -->
| What | Where | Values |
|---|---|---|
| Source | label | `src:intent`, `src:kickoff`, `src:claude`, `src:pm` |
| Feature | label `cap:<feature>` | the feature name, as in `openspec/specs/`, or `tooling` for work outside the specs |
| Size | label `size:<value>` | `trivial`, `normal`, `large` |
| Difficulty | label `difficulty:<value>` | `routine`, `tricky` |

## Conventions
- Labels or components the team uses: none yet (the project has no tickets)
- Review turnaround: `2 working days` (a PR waiting longer for a review gate is flagged by `/dirox-core:jira report-sprint`)
- Started before the baseline: `none`
