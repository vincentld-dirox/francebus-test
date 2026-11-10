# Spec Delta

## Purpose

Chooses the vehicle or vehicles for a request from the number of passengers, using the vehicle categories and capacities of France Bus's tariff configuration.

## ADDED Requirements

### Requirement: Vehicle from the number of passengers
The engine SHALL recommend the vehicle category whose passenger range, in the tariff configuration, holds the request's number of passengers. Categories and ranges are configuration, set from the tariff grid in discovery.

#### Scenario: Group fits one category
- **GIVEN** a tariff configuration where the category "minibus" holds 8 to 19 passengers
- **WHEN** a request has 12 passengers
- **THEN** the engine recommends one minibus

#### Scenario: Range edge
- **GIVEN** a tariff configuration where "minibus" holds 8 to 19 passengers and "coach" 20 to 55
- **WHEN** a request has 20 passengers
- **THEN** the engine recommends one coach

### Requirement: Several vehicles above the largest capacity
When the number of passengers is above the largest capacity in the tariff configuration, the engine SHALL recommend several vehicles whose capacities together hold every passenger, combined by the rule in the configuration.

#### Scenario: Group above the largest capacity
- **GIVEN** a tariff configuration whose largest capacity is 55 passengers
- **WHEN** a request has 80 passengers
- **THEN** the engine recommends more than one vehicle, with room for all 80 passengers together
