# Spec Delta

## Purpose

How the France Bus and client e-mails leave the system: prepared by the engine, sent by the plugin from devis@francebus.com through France Bus's Infomaniak mailbox, and never lost or sent twice when sending fails.

## ADDED Requirements

### Requirement: Sent from devis@francebus.com through the website's mailbox
The engine SHALL prepare each e-mail, and the plugin of the website the request came from SHALL send it from devis@francebus.com through France Bus's Infomaniak mailbox.

#### Scenario: E-mails sent by the website
- **WHEN** a request from francebus-wedding.com is confirmed
- **THEN** its France Bus and client e-mails are sent by the plugin of francebus-wedding.com, from devis@francebus.com

### Requirement: Delivery retried without duplicates
When sending an e-mail fails, it SHALL be tried again until it is sent, and an e-mail that was sent SHALL NOT be sent again.

#### Scenario: Mailbox unavailable, then back
- **GIVEN** the mailbox refuses a France Bus e-mail
- **WHEN** the mailbox works again
- **THEN** the e-mail is sent once

#### Scenario: Sent e-mail not sent again
- **GIVEN** an e-mail the plugin sent, whose sending the engine has not yet recorded
- **WHEN** the plugin next asks the engine for e-mails to send
- **THEN** that e-mail is not sent a second time
