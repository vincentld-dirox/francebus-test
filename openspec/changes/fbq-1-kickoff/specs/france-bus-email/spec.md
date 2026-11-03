# Spec Delta

## Purpose

The single e-mail France Bus receives for each request, in the exact format its CRM detects today, with the assistant's summary and the estimate added below; it is sent on confirmation, and still sent when the visitor does not confirm.

## ADDED Requirements

### Requirement: Today's e-mail format kept
The France Bus e-mail SHALL come from a fixed template that reproduces today's e-mail exactly: sender, subject "Francebus Service", field labels and their order. The assistant's summary, the vehicles, the distance, the constraints and the estimate SHALL come in a separate block after today's fields.

#### Scenario: Same sender, subject and labels as today
- **WHEN** the e-mail for a request is written
- **THEN** its sender, its subject "Francebus Service", and its field labels and their order are those of today's reference e-mail

#### Scenario: Estimate block after today's fields
- **WHEN** the e-mail for a priced request is written
- **THEN** the summary, vehicles, distance, constraints and estimate come in one block after the last of today's fields

### Requirement: Sent on confirmation
The France Bus e-mail of a confirmed request SHALL be sent when the visitor confirms it, not at Submit.

#### Scenario: Sent when the visitor confirms
- **GIVEN** a request taken by the engine
- **WHEN** the visitor confirms it in the estimate window
- **THEN** France Bus's e-mail for it is sent

### Requirement: Unconfirmed request still sent
A request the visitor neither confirms nor turns into a call request SHALL still be sent to France Bus after the delay set in the configuration, flagged "not confirmed", so that no lead is lost.

#### Scenario: Sent after the delay, flagged
- **GIVEN** a request taken by the engine that the visitor leaves without confirming
- **WHEN** the configured delay has passed
- **THEN** France Bus's e-mail for it is sent, flagged "not confirmed"

### Requirement: Call request sent at once
When the visitor chooses "prefer to talk to us", the request SHALL be sent to France Bus at once, as it stands, flagged as wanting a call.

#### Scenario: Call request flagged
- **WHEN** the visitor chooses "prefer to talk to us" before answering every question
- **THEN** France Bus's e-mail is sent at once with the request as it stands, flagged as wanting a call

### Requirement: One e-mail per request
France Bus SHALL receive one e-mail per request, never two.

#### Scenario: No second e-mail after confirmation
- **GIVEN** a confirmed request whose e-mail was sent
- **WHEN** the configured delay for unconfirmed requests passes
- **THEN** no other e-mail is sent to France Bus for that request

#### Scenario: No second e-mail after a call request
- **GIVEN** a call request whose e-mail was sent
- **WHEN** the configured delay for unconfirmed requests passes
- **THEN** no other e-mail is sent to France Bus for that request

### Requirement: Same format on both websites
The France Bus e-mail SHALL have the same format whichever website the request comes from.

#### Scenario: Request from the wedding site
- **WHEN** the e-mail for a request from francebus-wedding.com is written
- **THEN** it uses the same template as for francebus.com
