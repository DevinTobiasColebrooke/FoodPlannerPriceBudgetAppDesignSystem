# Vitamin D Service

## Overview
The VitaminDService calculates Adequate Intake (AI) and Tolerable Upper Intake Level (UL) for vitamin D based on age, sex, special conditions, and sunlight exposure.

## Inputs
- Age (Years, or months for infants)
- Sex (Male, Female)
- Pregnancy/Lactation Status
- Sunlight Exposure Estimate (none, limited, adequate)

## Core Logic
### AI Determination
- All life stages use AI (not RDA)
- Values assume minimal/no sun exposure and include safety factor:
  - 0-50y: 5 µg (200 IU)
  - 51-70y: 10 µg (400 IU)
  - >70y: 15 µg (600 IU)
- Pregnancy/Lactation: Same as non-pregnant/non-lactating peers
- Note: If sunlight exposure is adequate, dietary need may be zero

### UL Determination
- 0-12 months: 25 µg (1000 IU)
- 1 year and older: 50 µg (2000 IU)

## Output
```json
{
  "nutrient": "Vitamin D",
  "ai_ug": number,
  "ai_iu": number,
  "ul_ug": number,
  "ul_iu": number,
  "note": "AI assumes minimal/no sun exposure. Dietary need may be lower with adequate sun exposure."
}
```

## Dependencies
- DriLookupService
- UnitConversionService (µg to IU) 