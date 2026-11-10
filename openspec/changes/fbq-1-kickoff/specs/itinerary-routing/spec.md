# Spec Delta

## Purpose

Builds a request's itinerary leg by leg, for one-way, round-trip and multi-destination trips, with the distance and driving time of every leg from Google Maps, adjusted to the vehicle's speed.

## ADDED Requirements

### Requirement: Legs for each route type
The engine SHALL build the itinerary leg by leg: one way goes from the departure to the drop-off; a round trip has its outbound and return legs, on the same day or on different days; a multi-destination trip goes from the departure through each stop, in order, to the drop-off.

#### Scenario: One-way itinerary
- **WHEN** a one-way request goes from Lyon Part-Dieu station to Annecy
- **THEN** the itinerary has one leg, from Lyon Part-Dieu station to Annecy

#### Scenario: Round trip on different days
- **WHEN** a round trip leaves Paris for Reims on 3 March and comes back on 5 March
- **THEN** the itinerary has an outbound leg on 3 March and a return leg on 5 March

#### Scenario: Multi-destination in stop order
- **WHEN** a multi-destination request goes from Nice through Monaco, then Menton, to Nice
- **THEN** the itinerary has three legs: Nice to Monaco, Monaco to Menton, Menton to Nice

### Requirement: Distance and driving time from Google Maps
The engine SHALL get each leg's distance and driving time from Google Maps Routes; the itinerary's totals SHALL be the sums of its legs.

#### Scenario: Totals add up the legs
- **GIVEN** two legs of 120 km and 80 km
- **WHEN** the itinerary is built
- **THEN** its total distance is 200 km, and its total driving time is the sum of the two legs' times

### Requirement: Driving time adjusted to the vehicle
The engine SHALL adjust the car driving times from Google Maps to the speed of the recommended vehicle, with the coefficient set in the configuration (validated in discovery).

#### Scenario: Coach driving time
- **GIVEN** a configured coefficient of 1.2 for coaches and a leg of 100 minutes by car
- **WHEN** the recommended vehicle is a coach
- **THEN** the leg's driving time is 120 minutes

### Requirement: Typed addresses located
An address typed without a suggestion SHALL be located with Google Geocoding so that its legs can be measured.

#### Scenario: Typed address measured
- **GIVEN** the drop-off address was typed without a suggestion
- **WHEN** the itinerary is built
- **THEN** the leg to that address has a distance and a driving time

### Requirement: Repeated routes cached
The engine SHALL reuse the distance and time of a leg it measured before, within the cache lifetime set in the configuration, instead of asking Google Maps again.

#### Scenario: Repeated airport transfer
- **GIVEN** the leg from Paris Charles de Gaulle Airport to Disneyland Paris was measured within the cache lifetime
- **WHEN** another request needs the same leg
- **THEN** the engine uses the stored distance and time without calling Google Maps
