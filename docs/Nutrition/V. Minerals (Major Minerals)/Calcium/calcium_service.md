# Calcium Service

## Overview
The CalciumService calculates Adequate Intake (AI) and Tolerable Upper Intake Level (UL) for calcium based on age, sex, and special conditions.

## Inputs
- Age (Years, or months for infants)
- Sex (Male, Female)
- Pregnancy/Lactation Status

## Core Logic
### AI Determination
- Infants (0-6m, 7-12m): Based on human milk content + solid food contribution
- Children (1-3y, 4-8y): Based on extrapolation, accretion rates, balance studies
- Adolescents (9-18y): Based on desirable calcium retention model (AI=1300 mg/d)
- Adults (19-30y, 31-50y, 51-70y, >70y): Based on desirable calcium retention models, balance studies, BMD/fracture data
  - 19-50y: 1000 mg/d
  - 51-70y: 1200 mg/d
  - >70y: 1200 mg/d
- Pregnancy/Lactation: Same as non-pregnant/non-lactating peers

### UL Determination
- 0-12 months: Not Determinable (ND)
- 1 year and older: 2.5 g/day (2,500 mg/day)

## Output
```json
{
  "nutrient": "Calcium",
  "ai_mg": number,
  "ul_mg": number | "ND"
}
```

## Dependencies
- DriLookupService 