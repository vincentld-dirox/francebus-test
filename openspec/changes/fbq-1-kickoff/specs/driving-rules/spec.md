# Spec Delta

## Purpose

Checks that a request's schedule can be driven under France Bus's operating rules (driving time, breaks, length of the working day, daily rest, minimum notice), and computes alternatives when it cannot.

## ADDED Requirements

### Requirement: Schedule checked against the operating rules
The engine SHALL check each schedule against the operating rules in the configuration: driving time per leg and per day, breaks after driving periods, length of the driver's working day (amplitude) and daily rest. The values are set from the operating rules validated in discovery.

#### Scenario: Schedule within the rules
- **GIVEN** a schedule that keeps every configured limit
- **WHEN** the engine checks it
- **THEN** the schedule is feasible and no alternative is proposed

#### Scenario: Working day too long
- **GIVEN** a configured working day of at most 12 hours
- **WHEN** a same-day round trip needs the driver for 14 hours
- **THEN** the engine flags the schedule as not feasible and names the working-day rule

#### Scenario: Missing break
- **GIVEN** a configured break after a set driving period
- **WHEN** a leg is longer than that period and its departure and arrival times leave no room for the break
- **THEN** the engine flags the schedule as not feasible and names the break rule

### Requirement: Minimum notice detected
The engine SHALL detect a request whose departure is sooner than the minimum notice set in the operating rules, and mark it as below the minimum notice.

#### Scenario: Departure too soon
- **GIVEN** a configured minimum notice
- **WHEN** a request departs sooner than that notice after it is submitted
- **THEN** the engine marks the request as below the minimum notice

### Requirement: Alternatives from the engine
When a schedule does not fit, the engine SHALL compute alternatives that do fit the rules: add a break, add a second driver, move a departure time, or a larger vehicle.

#### Scenario: Alternatives fit the rules
- **GIVEN** a schedule flagged as not feasible
- **WHEN** the engine computes its alternatives
- **THEN** each alternative passes every operating-rule check

#### Scenario: Second driver for a long day
- **GIVEN** a same-day trip that breaks the configured working day for one driver
- **WHEN** the engine computes its alternatives
- **THEN** the alternatives include adding a second driver

### Requirement: Why a schedule does not fit
The assistant SHALL explain in plain words, in the visitor's language, why a schedule does not fit; the alternatives offered SHALL be only those the engine computed.

#### Scenario: Explanation names the rule
- **GIVEN** a schedule flagged for the working-day rule
- **WHEN** the window shows it
- **THEN** the assistant's explanation says that the driver's working day would be too long

#### Scenario: No alternative from the AI
- **WHEN** the window shows alternatives for a schedule that does not fit
- **THEN** each alternative shown is one the engine computed
