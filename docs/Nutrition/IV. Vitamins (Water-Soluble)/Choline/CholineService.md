# Choline Service

## Overview
The CholineService calculates recommended choline intake based on age, sex, and special conditions.

## Key Information from DRI Summary
- Recommendations for choline intake are made (page 2)
- AIs rather than RDAs are proposed for choline for persons of all ages (page 6)
- The AI for choline is based on the intake required to maintain liver function as assessed by measuring serum alanine aminotransferase levels (page 11)
- UL for adults (≥ 19 years) is 3.5 g/day (Table S-2, page 13). ULs are also provided for other age groups

## Inputs
- Age (Years, or months for infants)
- Sex
- Special Conditions (Flags):
  - Pregnant (Trimester)
  - Lactating (Months postpartum)

## Core Logic
1. Call DriLookupService to get the AI for choline based on age, sex, and special conditions
2. Call DriLookupService to get the UL for choline based on age, sex, and special conditions
3. No complex calculations like DFE or NE are indicated for choline in the summary

## Output
```json
{
  "ai_g": number,  // Example value, actual lookup needed
  "ul_g": number   // Example value, actual lookup needed
}
```

## Dependencies
- DriLookupService

Note: Units for Choline AI/UL are typically in mg or g. The table uses g/d for UL 