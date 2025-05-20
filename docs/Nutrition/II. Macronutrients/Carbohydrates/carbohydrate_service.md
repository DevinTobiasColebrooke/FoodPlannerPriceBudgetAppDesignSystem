# Carbohydrate Service

## Overview
The CarbohydrateService calculates recommended carbohydrate intake based on age, sex, and energy requirements.

## Inputs
- Age (Years)
- Sex (Male, Female)
- EER (kcal/day)
- Special Conditions

## Core Logic
1. Calculate RDA/AI for carbohydrates (g/day)
   - Based on brain glucose utilization
   - Uses DriLookupService for RDA/AI values

2. Calculate AMDR for carbohydrates
   - Percentage of EER
   - Convert to grams per day

3. Calculate Added Sugars limits
   - Based on Dietary Guidelines for Americans
   - Convert percentage to grams

## Output
```json
{
  "rda_g": number,
  "ai_g": number,
  "amdr_percent_min": number,
  "amdr_percent_max": number,
  "amdr_g_min": number,
  "amdr_g_max": number,
  "added_sugars_limit_percent": number,
  "added_sugars_limit_g": number
}
```

## Dependencies
- DriLookupService: For RDA/AI values
- EnergyRequirementService: For EER to calculate AMDR grams 