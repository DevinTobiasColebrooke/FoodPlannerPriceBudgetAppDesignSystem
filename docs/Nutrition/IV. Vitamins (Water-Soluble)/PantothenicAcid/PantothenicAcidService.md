# Pantothenic Acid Service (Vitamin B5)

## Overview
The PantothenicAcidService calculates recommended pantothenic acid intake based on age, sex, and special conditions.

## Key Information from DRI Summary
- AIs rather than RDAs are proposed for pantothenic acid for persons of all ages (page 6)
- The AI for pantothenic acid is based on data on intake sufficient to replace urinary excretion (page 11)
- ULs could NOT be established for pantothenic acid due to lack of suitable data (Table S-2, page 13)

## Inputs
- Age (Years, or months for infants)
- Sex
- Special Conditions (Flags):
  - Pregnant (Trimester)
  - Lactating (Months postpartum)

## Core Logic
1. Call DriLookupService to get the AI for pantothenic acid based on age, sex, and special conditions
2. Report that a UL is not determinable

## Output
```json
{
  "ai_mg": number,  // Example value, actual lookup needed
  "ul_mg": "ND"     // Not Determinable
}
```

## Dependencies
- DriLookupService

Note: Units for Pantothenic Acid AI are typically mg/day 