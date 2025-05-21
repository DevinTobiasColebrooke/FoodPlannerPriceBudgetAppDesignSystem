# Carotenoids Service

## Overview
The CarotenoidsService handles carotenoid intake recommendations based on age, sex, and special conditions.

## Key Information from DRI Summary
- No DRI values have been established for carotenoids
- Carotenoids are not considered essential nutrients
- No UL has been established for carotenoids

## Inputs
- Age (Years, or months for infants)
- Sex
- Special Conditions (Flags):
  - Pregnant (Trimester)
  - Lactating (Months postpartum)

## Core Logic
1. No DRI values available for carotenoids
2. No calculations or lookups required

## Output
```json
{
  "rda_mcg": "N/A",
  "ai_mcg": "N/A",
  "ul_mcg": "N/A"
}
```

## Dependencies
None

Note: No DRI values are established for carotenoids. This service will be needed for food intake analysis to track carotenoid consumption, but is not required for DRI calculations. 