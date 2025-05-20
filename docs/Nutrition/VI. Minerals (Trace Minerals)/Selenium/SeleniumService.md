# Selenium Service

## Overview
The SeleniumService calculates Selenium requirements based on age, sex, and special life stages.

## Inputs
- Age (Years)
- Sex (Male, Female)
- Special Conditions:
  - Pregnancy Status
  - Lactation Status

## Core Logic
1. Determine requirements from DriLookupService:
   - EAR (Estimated Average Requirement)
   - RDA (Recommended Dietary Allowance)
   - AI (Adequate Intake) if applicable
   - UL (Tolerable Upper Intake Level)

## Output
```json
{
  "nutrient": "Selenium",
  "ear_mcg": number,
  "rda_mcg": number,
  "ai_mcg": number | null,
  "ul_mcg": number
}
```

## Dependencies
### DriLookupService
- **Input**: Age, sex, pregnancy/lactation status
- **Responsibility**: Provide DRI values for Selenium
- **Output**: EAR, RDA, AI, UL values 