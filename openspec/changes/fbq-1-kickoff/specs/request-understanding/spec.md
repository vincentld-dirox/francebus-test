# Spec Delta

## Purpose

The AI assistant that turns a submitted form and its free-text description into a complete, structured request: it finds what is missing, vague or contradictory, asks the visitor short targeted questions, integrates the answers and rephrases the request, while every figure comes from the calculation engine.

## ADDED Requirements

### Requirement: Structured request from the form and the description
On Submit, the assistant SHALL turn the form fields and the free-text description into a structured request: route type, passengers, addresses, stops, dates and times, needs and contact details.

#### Scenario: Complete request needs no question
- **GIVEN** a one-way form with exact addresses, a date and time, the number of passengers, contact details and a description that agrees with them
- **WHEN** the form is submitted
- **THEN** the structured request holds those values and the assistant asks no question

### Requirement: Missing or unclear information asked
The assistant SHALL detect missing, vague or contradictory information and write one short, targeted question for each problem it finds.

#### Scenario: Return before departure
- **GIVEN** a round trip whose return date is before its departure date
- **WHEN** the form is submitted
- **THEN** the assistant asks the visitor to correct the dates

#### Scenario: Year differs from the description
- **GIVEN** the date field says 2026 and the description says the trip is in 2027
- **WHEN** the form is submitted
- **THEN** the assistant asks which year is right

#### Scenario: City instead of an address
- **GIVEN** the departure address is "Lyon"
- **WHEN** the form is submitted
- **THEN** the assistant asks for the exact departure address

#### Scenario: Stops only in the description
- **GIVEN** a multi-destination request whose stops are written only in the description
- **WHEN** the form is submitted
- **THEN** the assistant asks the visitor to confirm those stops as the trip's stops

### Requirement: Questions in the visitor's language
The assistant SHALL write its questions and its summary in French or English, in the language the estimate window is shown in.

#### Scenario: Questions in English
- **GIVEN** the estimate window is shown in English
- **WHEN** the assistant asks a question
- **THEN** the question is written in English

### Requirement: Phone asked once
When the phone number is missing, the assistant SHALL ask for it once, and the visitor MAY skip the question.

#### Scenario: Missing phone asked once
- **GIVEN** a form submitted without a phone number
- **WHEN** the assistant asks its questions, then asks again after the visitor's answers
- **THEN** the phone number is asked for the first time only

#### Scenario: Phone question skipped
- **WHEN** the visitor skips the phone question
- **THEN** the request goes on without a phone number

### Requirement: Answers integrated
The assistant SHALL integrate each answer into the structured request; a question answered is not asked again.

#### Scenario: Answer updates the request
- **GIVEN** the assistant asked for the exact departure address
- **WHEN** the visitor answers "Lyon Part-Dieu station"
- **THEN** the structured request holds that departure address and the question is not asked again

### Requirement: Request rephrased in plain words
The assistant SHALL rephrase the request in plain words after Submit and after each answer or edit.

#### Scenario: Summary rewritten after an answer
- **WHEN** the visitor answers a question
- **THEN** the rephrased request reflects the answer

### Requirement: No figure from the AI
The assistant SHALL NOT produce or change a price, a distance or a date. Prices and distances come from the calculation engine, dates from what the visitor entered; the assistant only words them.

#### Scenario: Date kept until the visitor changes it
- **GIVEN** the date field says 12 June 2026 and the description says 12 June 2027
- **WHEN** the assistant asks which date is right and the visitor has not answered yet
- **THEN** the date the engine uses is still 12 June 2026

#### Scenario: Summary figures match the engine
- **WHEN** the rephrased request and the estimate are shown
- **THEN** every price, distance and date in them equals the value the engine computed or the visitor entered

### Requirement: AI processing in the EU
The assistant SHALL use Gemini through Vertex AI at an EU location, under Google Cloud's business terms, so the data stays in the EU and is not used to train Google's models.

#### Scenario: Non-EU AI location refused
- **WHEN** the engine is configured with a Vertex AI location outside the EU
- **THEN** the engine refuses to start and says which setting is wrong
