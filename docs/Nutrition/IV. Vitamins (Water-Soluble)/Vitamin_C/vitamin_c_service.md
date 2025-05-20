# Vitamin C Service

## Overview
The VitaminCService calculates Vitamin C requirements based on age, sex, smoking status, and special life stages.

## Inputs
- Age (Years)
- Sex (Male, Female)
- Smoking Status (Boolean)
- Special Conditions:
  - Pregnancy Status
  - Lactation Status

## Core Logic
1. Determine base requirements from DriLookupService:
   - EAR (Estimated Average Requirement)
   - RDA (Recommended Dietary Allowance)
   - AI (Adequate Intake) if applicable
   - UL (Tolerable Upper Intake Level)

2. Apply smoking adjustment:
   - For adult smokers: Add 35 mg/day to RDA
   - No adjustment for non-smokers or children

## Output
```json
{
  "nutrient": "Vitamin C",
  "ear_mg": number,
  "rda_mg": number,
  "ai_mg": number | null,
  "ul_mg": number,
  "smoker_adjustment_mg": number
}
```

## Dependencies
### DriLookupService
- **Input**: Age, sex, pregnancy/lactation status
- **Responsibility**: Provide base DRI values for Vitamin C
- **Output**: EAR, RDA, AI, UL values 