# PolyunsaturatedFattyAcidsService

## Input
- Age
- Sex
- EER

## Responsibility
- Calculate AIs for Linoleic acid (n-6) (g/day)
- Calculate AIs for Alpha-Linolenic acid (n-3) (g/day)
- Calculate AMDR for Total Polyunsaturated Fatty Acids (PUFAs) (often a sub-component of the total fat AMDR, e.g., 5-10% for linoleic, 0.6-1.2% for ALA)

## Output
```typescript
{
  linoleic_ai_g: number,
  ala_ai_g: number,
  pufa_amdr_linoleic_percent_min: number,
  pufa_amdr_linoleic_percent_max: number,
  pufa_amdr_ala_percent_min: number,
  pufa_amdr_ala_percent_max: number,
  pufa_amdr_linoleic_g_min: number,
  pufa_amdr_linoleic_g_max: number,
  pufa_amdr_ala_g_min: number,
  pufa_amdr_ala_g_max: number
}
```

## Dependencies
- DriLookupService
- EnergyRequirementService 