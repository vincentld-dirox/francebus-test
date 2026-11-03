# Spec Delta

## Purpose

Hands each submitted quote form from the website to the engine, and makes sure no request is lost: when the engine cannot take it, the form is sent by JetFormBuilder's e-mail exactly as today.

## ADDED Requirements

### Requirement: Submitted form handed to the engine
With the plugin switched on, Submit SHALL hand the submitted form to the engine, which records it as a request with every form field and the website it came from.

#### Scenario: Form handed to the engine on Submit
- **GIVEN** the plugin is switched on and the engine answers
- **WHEN** the visitor submits the form on francebus-wedding.com
- **THEN** the engine records a request holding every field of the form, marked as coming from francebus-wedding.com

### Requirement: No e-mail at Submit
When the engine has taken the request, JetFormBuilder SHALL NOT send its e-mail: France Bus receives the request's single e-mail later, as `france-bus-email` describes.

#### Scenario: No e-mail sent at Submit
- **GIVEN** the engine has taken the request
- **WHEN** the submission ends
- **THEN** no e-mail has been sent to France Bus or to the visitor

### Requirement: Fallback to today's e-mail
When the engine cannot be reached at Submit (an error, or no answer within the configured time), the plugin SHALL let JetFormBuilder send its e-mail exactly as today.

#### Scenario: Engine unreachable at Submit
- **GIVEN** the engine cannot be reached
- **WHEN** the visitor submits the form
- **THEN** JetFormBuilder sends its e-mail exactly as today, with every field of the form

#### Scenario: Engine too slow at Submit
- **GIVEN** the engine does not answer within the configured time
- **WHEN** the visitor submits the form
- **THEN** JetFormBuilder sends its e-mail exactly as today
