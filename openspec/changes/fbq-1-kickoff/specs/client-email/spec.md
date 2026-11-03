# Spec Delta

## Purpose

The e-mail the visitor receives once they confirm their request: a summary of the request and the indicative estimate, in their language.

## ADDED Requirements

### Requirement: Summary e-mail on confirmation
When the visitor confirms the request, they SHALL receive an e-mail with a summary of the request and the indicative estimate, in the language the estimate window was shown in.

#### Scenario: Client e-mail on confirmation
- **WHEN** the visitor confirms the request
- **THEN** they receive an e-mail with the summary of the request and the indicative estimate

#### Scenario: Client e-mail in English
- **GIVEN** the estimate window was shown in English
- **WHEN** the visitor confirms the request
- **THEN** the e-mail they receive is in English

### Requirement: No client e-mail without confirmation
The client e-mail SHALL be sent only for a request the visitor confirmed.

#### Scenario: Unconfirmed request sends no client e-mail
- **GIVEN** a request the visitor leaves without confirming
- **WHEN** it is sent to France Bus flagged "not confirmed"
- **THEN** the visitor receives no e-mail

### Requirement: Indicative wording in the e-mail
The client e-mail SHALL present the estimate as indicative and non-binding, subject to availability and confirmed by France Bus's team.

#### Scenario: Estimate marked indicative in the e-mail
- **WHEN** the client e-mail shows an estimate
- **THEN** the estimate is marked indicative and non-binding, subject to availability and confirmed by France Bus's team
