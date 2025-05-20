# Water Service

## Overview
The WaterService calculates recommended water intake based on age, sex, and physical activity level.

## Inputs
- Age (Years)
- Sex (Male, Female)
- PAL (Physical Activity Level)
- Environmental Factors (optional)

## Core Logic
1. Calculate AI for Total Water
   - Includes water from beverages and food
   - Uses DriLookupService for AI values

## Output
```json
{
  "ai_liters": number
}
```

## Dependencies
- DriLookupService: For AI values 