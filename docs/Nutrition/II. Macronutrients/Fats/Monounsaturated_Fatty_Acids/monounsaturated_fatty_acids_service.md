# MonounsaturatedFattyAcidsService

## Input
- EER
- Calculated Total Fat AMDR
- Calculated Saturated Fat limit
- Calculated PUFA AMDR

## Responsibility
State that there is no specific DRI (RDA/AI) or AMDR for Monounsaturated Fatty Acids (MUFAs) themselves. MUFA intake is generally the remainder of fat intake after accounting for essential PUFAs and limiting saturated/trans fats within the total fat AMDR. This service might provide a calculated residual range for MUFAs.

## Output
```typescript
{
  mufa_calculated_range_g_min: number,
  mufa_calculated_range_g_max: number,
  recommendation_text: string
}
```

## Dependencies
- TotalFatService
- SaturatedFatService
- PolyunsaturatedFattyAcidsService
- EnergyRequirementService 