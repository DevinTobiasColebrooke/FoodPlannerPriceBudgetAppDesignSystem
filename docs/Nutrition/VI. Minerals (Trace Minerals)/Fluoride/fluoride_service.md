# Fluoride Service

## Overview
The FluorideService calculates Adequate Intake (AI) and Tolerable Upper Intake Level (UL) for fluoride based on age, sex, and special conditions.

## Inputs
- Age (Years, or months for infants)
- Sex (Male, Female)
- Pregnancy/Lactation Status

## Core Logic
### AI Determination
- All life stages use AI (not RDA)
- Based on estimated intakes that reduce dental caries maximally without unwanted side effects
- General criterion: 0.05 mg/kg/day
- Values (mg/day):
  - 0-6m: 0.01
  - 7-12m: 0.5
  - 1-3y: 0.7
  - 4-8y: 1.0
  - 9-13y: 2.0
  - 14-18y: 3.0 (both sexes)
  - 19+ M: 4.0
  - 19+ F: 3.0
- Pregnancy/Lactation: Same as non-pregnant/non-lactating peers

### UL Determination
- Based on preventing moderate enamel fluorosis (up to age 8) or skeletal fluorosis (>8 years)
- Values (mg/day):
  - 0-6m: 0.7
  - 7-12m: 0.9
  - 1-3y: 1.3
  - 4-8y: 2.2
  - >8y: 10

## Output
```json
{
  "nutrient": "Fluoride",
  "ai_mg": number,
  "ul_mg": number
}
```

## Dependencies
- DriLookupService 