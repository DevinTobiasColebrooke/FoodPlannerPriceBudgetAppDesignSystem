# Phosphorus Service

## Overview
The PhosphorusService calculates Adequate Intake (AI), Estimated Average Requirement (EAR), Recommended Dietary Allowance (RDA), and Tolerable Upper Intake Level (UL) for phosphorus based on age, sex, and special conditions.

## Inputs
- Age (Years, or months for infants)
- Sex (Male, Female)
- Pregnancy/Lactation Status

## Core Logic
### AI/EAR/RDA Determination
- Infants (0-6m, 7-12m): AI based on human milk content + solid food
- Children/Adolescents (1-18y): EAR based on factorial approach (accretion + urinary loss) / absorption %
- Adults (19y+): EAR based on maintenance of serum Pi at lower end of normal (0.87 mmol/L or 2.7 mg/dL)
  - EAR: 580 mg/d
- Pregnancy/Lactation: Same as non-pregnant/non-lactating peers

### RDA Calculation
- RDA = EAR * 1.2 (assuming 10% CV)

### UL Determination
- 0-12 months: Not Determinable (ND)
- Older groups: Varies (3g, 4g, 3.5g)

## Output
```json
{
  "nutrient": "Phosphorus",
  "ai_mg": number | null,
  "ear_mg": number,
  "rda_mg": number,
  "ul_mg": number | "ND"
}
```

## Dependencies
- DriLookupService
- RdaCalculationService 