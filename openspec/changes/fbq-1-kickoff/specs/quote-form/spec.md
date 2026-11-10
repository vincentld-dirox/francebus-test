# Spec Delta

## Purpose

The quote form visitors fill in on francebus.com and francebus-wedding.com: the existing JetFormBuilder form, with a stop list for multi-destination trips and a return date and time for round trips, built in JetFormBuilder itself so they keep working without the plugin.

## ADDED Requirements

### Requirement: Stop list for multi-destination trips
The form SHALL show a stop list only when the route type is "Multi-destination". It replaces the single "Stages" text box: one line per stop, each with an address, an arrival time and a waiting time. The visitor SHALL be able to add, remove and reorder stops.

#### Scenario: Stop list shown for multi-destination
- **WHEN** the visitor picks the route type "Multi-destination"
- **THEN** the form shows "Stop 1" with its address, arrival time and waiting time, and a "+ Add a stop" button

#### Scenario: No stop list for other route types
- **WHEN** the visitor picks the route type "One way" or "Round trip"
- **THEN** the form shows no stop list

#### Scenario: Stops added, reordered and removed
- **WHEN** the visitor adds a second and a third stop, moves the third above the first, then removes the second
- **THEN** the form keeps two stops, in the order the visitor set

### Requirement: Return date and time for round trips
The form SHALL ask for a dedicated return date and time when the route type is "Round trip".

#### Scenario: Return date shown for a round trip
- **WHEN** the visitor picks the route type "Round trip"
- **THEN** the form shows a return date field and a return time field

### Requirement: Stops written in today's e-mail fields
The form SHALL write the addresses as text in the e-mail fields used today, and the stops as text under "Stages", in the visitor's order, so that the CRM reading the e-mail is not affected.

#### Scenario: Stops listed under Stages in order
- **GIVEN** a multi-destination request with the stops "Lyon Part-Dieu station" then "Annecy, Place de l'Hôtel de Ville"
- **WHEN** the form's e-mail is written
- **THEN** its "Stages" field lists "Lyon Part-Dieu station" before "Annecy, Place de l'Hôtel de Ville", each with its arrival and waiting times

### Requirement: Upgrades work without the plugin
The stop list and the return date and time SHALL keep working when the plugin is switched off or not installed, because they are built in JetFormBuilder.

#### Scenario: Stops sent with the plugin off
- **GIVEN** the plugin is switched off
- **WHEN** the visitor submits a multi-destination request with two stops
- **THEN** JetFormBuilder's e-mail lists both stops under "Stages"

### Requirement: Form keeps its look
The form SHALL keep its current look on both websites, on desktop and mobile; only the stop list and the return date and time are new.

#### Scenario: Look unchanged on desktop and mobile
- **WHEN** the form page of either website is shown at desktop and mobile widths
- **THEN** it matches the screenshots taken before the change, apart from the stop list and the return date and time
