# Folate Service

## Overview
The FolateService calculates recommended folate intake based on age, sex, and special conditions, including specific recommendations for women capable of becoming pregnant.

## Key Information from DRI Summary
- Major new recommendation: Use of dietary folate equivalents (DFEs). 1 µg food folate = 0.6 µg folate added to foods (folic acid) or taken with food OR 0.5 µg of folate supplements (folic acid) taken on an empty stomach (page 1)
- RDA for folate is the same for men and women (page 2). Table S-1 gives EAR for adults as 320 µg/d DFE
- Special recommendation for women capable of becoming pregnant: intake of folate from fortified food or supplements (page 2, detailed on page 11-12 as 400 µg/day of folic acid in addition to food folate)
- UL for adults (≥ 19 years) is 1,000 µg/day, specifically for folate from supplements, fortified foods, or a combination (folic acid) (Table S-2, page 13)

## Inputs
- Age (Years, or months for infants)
- Sex
- Special Conditions (Flags):
  - Pregnant (Trimester)
  - Lactating (Months postpartum)
  - Capable of becoming pregnant (boolean)

## Core Logic
1. Call DriLookupService to get the EAR/RDA (or AI for infants) for folate in DFE based on age, sex, and special conditions (pregnancy/lactation)
2. If "capable of becoming pregnant" is true and the woman is not already pregnant, add the specific recommendation for folic acid (400 µg/day). This is a separate recommendation in addition to the DFE RDA for general health
3. Call DriLookupService to get the UL for folic acid (from fortified foods/supplements) based on age, sex, and special conditions

## Output
```json
{
  "rda_dfe_mcg": number,         // Example value in DFE
  "ai_dfe_mcg": null,           // Or AI if RDA not applicable
  "ul_folic_acid_mcg": number,  // UL for synthetic folic acid
  "folic_acid_recommendation_for_pregnancy_capable_mcg": number // If applicable
}
```

## Dependencies
- DriLookupService

Note: Units for Folate are µg/day. The DriLookupService should ideally store DRI values in DFE. If the application later includes features to analyze food intake, a separate DfeCalculationService or utility would be needed to convert food folate and folic acid amounts from various sources into a total DFE intake. For calculating needs, the DRIs are already expressed as DFE. 