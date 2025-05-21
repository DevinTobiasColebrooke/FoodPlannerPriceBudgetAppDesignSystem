# Protein Service

## Overview
The ProteinService calculates recommended protein intake based on age, sex, weight, and energy requirements.

## Inputs
- Age (Years)
- Sex (Male, Female)
- Weight (kg)
- EER (kcal/day)
- Special Conditions:
  - Pregnancy
  - Lactation
  - Vegetarian status (for quality adjustment)

## Core Logic
1. Calculate RDA/AI for protein
   - Grams per day
   - Grams per kg body weight
   - Uses DriLookupService for RDA/AI values

2. Calculate AMDR for protein
   - Percentage of EER
   - Convert to grams per day

## Output
```json
{
  "rda_g_per_kg": number,
  "rda_g_total": number,
  "ai_g_total": number,
  "amdr_percent_min": number,
  "amdr_percent_max": number,
  "amdr_g_min": number,
  "amdr_g_max": number
}
```

## Dependencies
- DriLookupService: For RDA/AI values
- EnergyRequirementService: For EER to calculate AMDR grams 