# Spec Delta

## Purpose

Prices a request from France Bus's own tariff grid and rules, kept as configuration: the same request always gets the same price, every price is explained step by step, and visitors see it only once France Bus has approved the calibration.

## ADDED Requirements

### Requirement: Base price from the tariff grid
The engine SHALL take the base price from the tariff configuration: the price of a known fixed route first; otherwise a price per km for long-distance trips or a price per duration for local services, split by the rule set in the configuration.

#### Scenario: Known fixed route
- **GIVEN** a tariff configuration with a fixed-route price for a coach from Paris Charles de Gaulle Airport to Disneyland Paris
- **WHEN** a request for one coach on that route is priced
- **THEN** the base price is the fixed-route price

#### Scenario: Long distance priced per km
- **GIVEN** no fixed route matches and the configuration classes the trip as long distance
- **WHEN** the request is priced
- **THEN** the base price is the configured rate per km for the vehicle times the itinerary's distance

#### Scenario: Local service priced per duration
- **GIVEN** no fixed route matches and the configuration classes the trip as a local service
- **WHEN** the request is priced
- **THEN** the base price is the configured rate per duration for the vehicle times the service's duration

### Requirement: Surcharges and fees
The engine SHALL add the surcharges (night, Sunday, public holidays) and the fees (tolls, parking, driver meals and lodging) that the tariff configuration applies to the request.

#### Scenario: Sunday surcharge
- **GIVEN** a configured Sunday surcharge
- **WHEN** a trip on a Sunday is priced
- **THEN** the price includes the Sunday surcharge

#### Scenario: Weekday daytime trip
- **WHEN** a trip on a weekday that is not a public holiday, entirely outside the configured night hours, is priced
- **THEN** the price includes no night, Sunday or holiday surcharge

#### Scenario: Fee added
- **GIVEN** a configured fee whose condition the request meets
- **WHEN** the request is priced
- **THEN** the price includes that fee

### Requirement: Step-by-step breakdown
The engine SHALL save with each estimate a breakdown that explains its price step by step: the base price and how it was found, each surcharge and fee, the total, and the rounding.

#### Scenario: Breakdown explains the price
- **WHEN** a request is priced
- **THEN** its breakdown lists the base price, each surcharge and fee with its amount, and the total they add up to before rounding

### Requirement: Same request, same price
For the same request, the same measured legs and the same configuration version, the engine SHALL always give the same price and the same breakdown.

#### Scenario: Priced twice
- **WHEN** the same request is priced twice with the same legs and configuration version
- **THEN** both prices and both breakdowns are identical

### Requirement: Rounded indicative price incl. VAT
The engine SHALL give the visitor-facing price incl. VAT, rounded by the rule in the configuration, worded "From XXXX € incl. VAT" in English and "À partir de XXXX € TTC" in French.

#### Scenario: Rounded price
- **GIVEN** a configured rounding up to the next 10 €
- **WHEN** a request's total is 1,234.56 € incl. VAT
- **THEN** the price shown is "From 1,240 € incl. VAT" in English and "À partir de 1 240 € TTC" in French

### Requirement: Tariffs and rules are configuration
Prices, vehicle categories, fixed routes, surcharges, fees, operating rules and coefficients SHALL live in a versioned configuration, separate from the code, that Dirox can update without changing the code. Each estimate SHALL record the configuration version it used.

#### Scenario: Rate changed without a code change
- **GIVEN** Dirox publishes a configuration version with a new rate per km for coaches, and no code change
- **WHEN** a long-distance coach request is priced
- **THEN** the price uses the new rate

#### Scenario: Estimate records its configuration version
- **WHEN** a request is priced
- **THEN** the estimate records the version of the configuration it used

### Requirement: Price display switch
Each website SHALL have a price display switch, off until France Bus has approved the calibration results. While it is off, no price SHALL be shown to visitors, in the window or in the client e-mail.

#### Scenario: No price while the switch is off
- **GIVEN** the price display switch of francebus.com is off
- **WHEN** a visitor on francebus.com gets an estimate and confirms the request
- **THEN** neither the window nor the client e-mail shows a price

#### Scenario: Price shown once switched on
- **GIVEN** the price display switch of francebus.com is on
- **WHEN** a visitor on francebus.com gets an estimate
- **THEN** the window shows the rounded indicative price

### Requirement: Calibration against past quotes
On the test set of past requests with the prices France Bus actually quoted, the engine's estimates SHALL match those prices within the tolerance agreed in discovery, set in the calibration run's configuration.

#### Scenario: Test set within tolerance
- **WHEN** the calibration run prices every request of the test set
- **THEN** the run reports each estimate next to the quoted price, and passes only when the results are within the agreed tolerance
