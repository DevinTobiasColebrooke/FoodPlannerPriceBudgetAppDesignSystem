# Magnesium Service

## Overview
The MagnesiumService calculates Adequate Intake (AI), Estimated Average Requirement (EAR), Recommended Dietary Allowance (RDA), and Tolerable Upper Intake Level (UL) for magnesium based on age, sex, and special conditions.

## Inputs
- Age (Years, or months for infants)
- Sex (Male, Female)
- Pregnancy/Lactation Status

## Core Logic
### AI/EAR/RDA Determination
- Infants (0-6m, 7-12m): AI based on human milk content + solid food
- Children/Adolescents (1-18y): EAR based on balance studies, accretion goals
  - Younger: 5 mg/kg/day
  - Older: 5.3 mg/kg/day
- Adults (19y+): EAR based on balance studies (maintaining zero balance)
- Pregnancy: EAR increased by +35 mg/day based on LBM accretion
- Lactation: Same as non-lactating peers

### RDA Calculation
- RDA = EAR * 1.2 (assuming 10% CV)

### UL Determination
- 0-12 months: Not Determinable (ND)
- 1 year and older: 65-350 mg/day from supplemental sources only
- Note: UL applies only to pharmacological agents (supplements, medications), not food and water

## Output
```json
{
  "nutrient": "Magnesium",
  "ai_mg": number | null,
  "ear_mg": number,
  "rda_mg": number,
  "ul_mg_supplemental": number | "ND",
  "ul_note": "Applies to supplemental sources only"
}
```

## Dependencies
- DriLookupService
- RdaCalculationService 