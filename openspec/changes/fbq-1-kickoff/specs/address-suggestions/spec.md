# Spec Delta

## Purpose

Helps visitors enter addresses the engine can measure: Google suggestions (addresses, airports, stations) while they type, in the form's address fields and in the estimate window, with free text still accepted.

## ADDED Requirements

### Requirement: Suggestions while typing in the form
The departure, stop and drop-off fields of the form SHALL suggest Google places (addresses, airports, stations) as the visitor types.

#### Scenario: Airport suggested while typing
- **WHEN** the visitor types "charle" in the departure field
- **THEN** the suggestions include Paris Charles de Gaulle Airport

#### Scenario: Suggestion picked
- **WHEN** the visitor picks a suggestion
- **THEN** the field holds the suggested address as text

### Requirement: Free text accepted
An address field SHALL accept an address typed without picking a suggestion.

#### Scenario: Address typed without a suggestion
- **WHEN** the visitor types "12 rue des Lilas, Annecy" and picks no suggestion
- **THEN** the form accepts the address as typed

### Requirement: Suggestions in the estimate window
Every address input of the estimate window (answers to the assistant's questions, edited addresses and stops) SHALL offer the same suggestions as the form.

#### Scenario: Suggestions when editing a stop
- **WHEN** the visitor edits a stop's address in the estimate window and types "gare de lyon"
- **THEN** the suggestions include Paris Gare de Lyon
