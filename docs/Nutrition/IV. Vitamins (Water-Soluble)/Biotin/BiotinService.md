# Biotin Service

## Overview
The BiotinService calculates recommended biotin intake based on age, sex, and special conditions.

## Key Information from DRI Summary
- AIs rather than RDAs are proposed for biotin for persons of all ages (page 6)
- The AI for biotin is based on limited intake data (page 11)
- ULs could NOT be established for biotin due to lack of suitable data (Table S-2, page 13)

## Inputs
- Age (Years, or months for infants)
- Sex
- Special Conditions (Flags):
  - Pregnant (Trimester)
  - Lactating (Months postpartum)

## Core Logic
1. Call DriLookupService to get the AI for biotin based on age, sex, and special conditions
2. Report that a UL is not determinable

## Output
```json
{
  "ai_mcg": number,  // Example value, actual lookup needed
  "ul_mcg": "ND"     // Not Determinable
}
```

## Dependencies
- DriLookupService

Note: Units for Biotin AI are typically µg/day