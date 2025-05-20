# Vitamin E Service

## Overview
The VitaminEService calculates Vitamin E requirements specifically for 2R-stereoisomeric forms of α-tocopherol.

## Inputs
- Age (Years)
- Sex (Male, Female)
- Special Conditions:
  - Pregnancy Status
  - Lactation Status

## Core Logic
1. Determine requirements from DriLookupService:
   - EAR for 2R-stereoisomeric forms of α-tocopherol
   - RDA for 2R-stereoisomeric forms of α-tocopherol
   - AI for 2R-stereoisomeric forms of α-tocopherol if applicable
   - UL for any form of supplemental α-tocopherol

## Output
```json
{
  "nutrient": "Vitamin E",
  "ear_alpha_tocopherol_mg": number,
  "rda_alpha_tocopherol_mg": number,
  "ai_alpha_tocopherol_mg": number | null,
  "ul_supplemental_alpha_tocopherol_mg": number
}
```

## Dependencies
### DriLookupService
- **Input**: Age, sex, pregnancy/lactation status
- **Responsibility**: Provide DRI values for α-tocopherol
- **Output**: EAR, RDA, AI, UL values

## Notes
- All requirements refer specifically to 2R-stereoisomeric forms of α-tocopherol
- UL applies to any form of supplemental α-tocopherol
- Conversion may be needed if food composition data uses IU or total tocopherols 