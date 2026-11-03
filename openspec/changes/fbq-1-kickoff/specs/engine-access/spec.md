# Spec Delta

## Purpose

The engine's single secure entry for both websites: only the known websites can call it, each with its own credentials and from its own domains, within rate limits, and the Google and Gemini credentials never leave the engine.

## ADDED Requirements

### Requirement: Known websites only
The engine SHALL refuse every call that does not carry valid credentials of a website it knows. Each website (francebus.com, francebus-wedding.com) has its own credentials.

#### Scenario: Call without credentials refused
- **WHEN** a call reaches the engine without website credentials
- **THEN** the engine refuses it and does nothing else

#### Scenario: Revoked credentials refused
- **GIVEN** the credentials of a website have been replaced
- **WHEN** a call uses the old credentials
- **THEN** the engine refuses it

### Requirement: Browser calls checked against the website's domains
A call made from a browser SHALL be refused when its origin is not one of the domains of the website whose credentials it carries.

#### Scenario: Credentials used from another origin
- **GIVEN** the browser credentials of francebus.com
- **WHEN** a browser call carrying them comes from another domain
- **THEN** the engine refuses it

### Requirement: Server calls need the website's secret
Handing over a submitted form and fetching e-mails to send SHALL require the website's secret, which is kept on the website's server and never sent to the browser.

#### Scenario: E-mails not readable from the browser
- **WHEN** a call with only the browser credentials asks for the e-mails to send
- **THEN** the engine refuses it

### Requirement: Rate limiting
The engine SHALL refuse calls above the rates set in its configuration, and tell the caller to try again later.

#### Scenario: Too many calls refused
- **GIVEN** a configured limit of calls per minute
- **WHEN** one more call than the limit arrives from the same visitor within a minute
- **THEN** the engine refuses it and answers that the visitor should try again later

### Requirement: Request readable only with its token
A request SHALL be read or changed only with the token the engine issued for it at Submit.

#### Scenario: Request without its token refused
- **WHEN** a call asks for a request without its token
- **THEN** the engine refuses it and reveals nothing about the request

#### Scenario: Token of another request refused
- **WHEN** a call asks for request A with the token issued for request B
- **THEN** the engine refuses it

### Requirement: Credentials stay on the engine
The Google Maps and Gemini credentials SHALL stay on the engine: they SHALL NOT appear in the pages, in the browser's calls, or in the websites' settings.

#### Scenario: No Google credential in the page
- **WHEN** the form page loads and the visitor gets address suggestions and an estimate
- **THEN** no Google Maps or Gemini credential appears in the page, its scripts or its network calls
