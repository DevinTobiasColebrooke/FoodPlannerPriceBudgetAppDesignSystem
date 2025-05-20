# Fat Service

## Overview
The FatService calculates recommended fat intake, including essential fatty acids, based on age, sex, and energy requirements.

## Inputs
- Age (Years)
- Sex (Male, Female)
- EER (kcal/day)
- Special Conditions

## Core Logic
1. Calculate AI for Total Fat (infants)
   - Uses DriLookupService for AI values

2. Calculate AMDR for Total Fat
   - Percentage of EER
   - Convert to grams per day

3. Calculate AIs for Essential Fatty Acids
   - Linoleic Acid (g/day)
   - Alpha-Linolenic Acid (g/day)

4. Calculate limits for Saturated Fat & Trans Fat
   - Based on Dietary Guidelines for Americans
   - Convert percentage to grams

## Output
```json
{
  "total_fat_amdr_percent_min": number,
  "total_fat_amdr_percent_max": number,
  "total_fat_amdr_g_min": number,
  "total_fat_amdr_g_max": number,
  "linoleic_ai_g": number,
  "ala_ai_g": number,
  "saturated_fat_limit_percent": number,
  "saturated_fat_limit_g": number,
  "trans_fat_limit_percent": number,
  "trans_fat_limit_g": number
}
```

## Dependencies
- DriLookupService: For RDA/AI values
- EnergyRequirementService: For EER to calculate AMDR grams 