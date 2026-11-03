# Spec Delta

## Purpose

The window that opens on the form page after Submit, where the visitor reads the summary and the indicative estimate, answers the assistant's questions, edits values, picks alternatives, then confirms the request or asks to be contacted.

## ADDED Requirements

### Requirement: Summary shown after Submit
After Submit, a window SHALL open on the form page showing the rephrased request, the recommended vehicle or vehicles, the itinerary leg by leg with distance and driving time, the constraints, and, when the website's price display is on, the estimate as "From XXXX € incl. VAT".

#### Scenario: Summary after Submit
- **GIVEN** the engine has taken a round-trip request and the price display is on
- **WHEN** the window opens
- **THEN** it shows the rephrased request, the vehicles, each leg with its distance and driving time, the constraints and "From XXXX € incl. VAT"

### Requirement: Estimate marked indicative
The window SHALL always present the estimate as indicative and non-binding, "subject to availability, confirmed by our team".

#### Scenario: Indicative notice next to the estimate
- **WHEN** the window shows an estimate
- **THEN** the estimate is marked indicative and non-binding, subject to availability and confirmed by France Bus's team

### Requirement: Questions with the right input
Each question of the assistant SHALL come with the input that fits what it asks: a date, a time, an address with suggestions, or a number.

#### Scenario: Date question uses a date input
- **WHEN** the assistant asks for the return date
- **THEN** the window shows the question with a date input

#### Scenario: Address question uses an address input
- **WHEN** the assistant asks for the exact departure address
- **THEN** the window shows the question with an address input that offers suggestions

### Requirement: Values editable with instant recalculation
The visitor SHALL be able to edit the passengers, dates, times, addresses, stops and trip type in the window; each change SHALL update the vehicles, itinerary, constraints and estimate at once, without reloading the page.

#### Scenario: Passengers changed
- **GIVEN** an estimate for 30 passengers
- **WHEN** the visitor changes the number of passengers to 45
- **THEN** the vehicles, itinerary, constraints and estimate shown are recalculated for 45 passengers without reloading the page

#### Scenario: Trip type changed
- **GIVEN** a one-way estimate
- **WHEN** the visitor changes the trip type to round trip and enters a return date and time
- **THEN** the itinerary shows the return legs and the estimate is recalculated

### Requirement: Alternatives in one click
When the engine finds that a schedule does not fit, the window SHALL show the engine's alternatives, each selectable in one click; picking one applies it and recalculates the estimate.

#### Scenario: Alternative picked
- **GIVEN** a schedule that does not fit and the engine's alternative "add a second driver"
- **WHEN** the visitor picks that alternative
- **THEN** the request includes a second driver and the window shows the recalculated estimate

### Requirement: Confirmation step
The window SHALL end with a confirmation step; confirming marks the request confirmed and sends it, as `france-bus-email` and `client-email` describe.

#### Scenario: Request confirmed
- **WHEN** the visitor confirms the request
- **THEN** the request is marked confirmed and its France Bus and client e-mails are sent

### Requirement: Talk to us instead
The window SHALL offer "prefer to talk to us"; choosing it sends the request to France Bus as a call request, as `france-bus-email` describes, without a confirmation step.

#### Scenario: Visitor prefers to talk
- **WHEN** the visitor chooses "prefer to talk to us"
- **THEN** the request is sent to France Bus as a call request and the window asks for no confirmation

### Requirement: Window language
The window SHALL be shown in the language of the page the visitor is on, French or English; when that language cannot be told, in the language set in the plugin's settings.

#### Scenario: English page
- **WHEN** the visitor submits the form on an English page
- **THEN** the window is shown in English

#### Scenario: Page language unknown
- **GIVEN** the plugin's language setting is French and the page's language cannot be told
- **WHEN** the window opens
- **THEN** it is shown in French

### Requirement: AI notice
The window SHALL say that the summary and the questions are written by an AI.

#### Scenario: AI notice shown
- **WHEN** the window shows the summary or a question
- **THEN** it says that they are written by an AI

### Requirement: Desktop and mobile
The window SHALL be usable on desktop and mobile screens. Its style follows the website set in the plugin's settings, as `plugin-settings` describes.

#### Scenario: Window on a mobile screen
- **WHEN** the window opens on a mobile-width screen
- **THEN** every part of it can be read and used without horizontal scrolling
