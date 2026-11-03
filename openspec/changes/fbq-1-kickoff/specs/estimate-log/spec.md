# Spec Delta

## Purpose

Keeps a record of every form that reaches the engine and every estimate it makes, so Dirox can calibrate the engine, explain any price afterwards and track unconfirmed requests; stored in the EU, kept for the agreed period, exported as CSV on request.

## ADDED Requirements

### Requirement: Every form logged
The engine SHALL log every form it receives, with its data and the time, even when no estimate can be made yet.

#### Scenario: Form logged before any estimate
- **GIVEN** a form whose departure address is only a city
- **WHEN** the engine receives it
- **THEN** the log holds the form's data and the time it arrived

### Requirement: Every estimate logged
The engine SHALL log every estimate with the request's data, the assistant's questions and the visitor's answers, the price breakdown, the request's status (confirmed or not) and the time.

#### Scenario: Confirmed request logged
- **WHEN** the visitor confirms a request
- **THEN** the log holds its estimates with their breakdowns, its questions and answers, and the status confirmed

#### Scenario: Unconfirmed request logged
- **WHEN** a request is sent to France Bus flagged "not confirmed"
- **THEN** the log holds its estimates and the status not confirmed

### Requirement: Any logged price explained afterwards
A logged estimate SHALL hold what is needed to compute its price again: the request's values, the measured legs and the configuration version.

#### Scenario: Logged estimate priced again
- **WHEN** a logged estimate is priced again from its logged values, legs and configuration version
- **THEN** the price and breakdown are the ones logged

### Requirement: Log stored in the EU
The log SHALL be stored in the EU, in every environment that holds real visitors' data.

#### Scenario: Production log in the EU
- **WHEN** the production environment is set up
- **THEN** the log's database is in an EU region

### Requirement: Retention
Log entries SHALL be removed once they are older than the retention period set in the configuration, as agreed in discovery.

#### Scenario: Entry past retention removed
- **GIVEN** a configured retention period
- **WHEN** a log entry becomes older than that period
- **THEN** it is removed from the log

### Requirement: CSV export on request
Dirox SHALL be able to export the log as CSV for a period; there is no screen to view the log.

#### Scenario: Log exported as CSV
- **WHEN** Dirox exports the log for a period
- **THEN** it gets a CSV file with one line per estimate of that period, with its request's data, status, price and time
