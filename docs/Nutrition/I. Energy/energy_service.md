# Energy Service

## Overview
The EnergyRequirementService calculates Estimated Energy Requirement (EER) based on age, sex, physical activity level, and special life stages.

## Key Concepts
- **EER (Estimated Energy Requirement)**: Average dietary energy intake to maintain energy balance
- **TEE (Total Energy Expenditure)**: Comprises REE/BMR, TEF/DIT, and PAL
- **PAL (Physical Activity Level)**: Ratio of TEE to BMR
- **SEPV (Standard Error of the Predicted Value)**: Variability of individual requirements

## Inputs
- Age (Years)
- Sex (Male, Female)
- Weight (kg)
- Height (cm)
- PAL Category (Inactive, Low Active, Active, Very Active)
- Special Conditions:
  - Pregnancy (gestational weeks, pre-pregnancy BMI)
  - Lactation (months postpartum, exclusivity)

## Core Logic
### TEE Calculation
1. Select the Correct TEE Equation based on age/sex and life stage (Tables S-1, S-2 / 5-5, 5-15):
   - Men, 19+ years
   - Women, 19+ years
   - Boys, 3–18 years
   - Girls, 3–18 years
   - Boys, 0–2 years (no PAL categories)
   - Girls, 0–2 years (no PAL categories)
   - Pregnant women (2nd/3rd trimester - separate PAL categories)
   - Lactating women (0-6 months exclusive, 7-12 months partial - uses non-pregnant TEE + adjustments)

2. Apply the Selected PAL Category:
   - TEE equations for ages 3+ incorporate PAL directly as different sets of coefficients or intercepts

### Special Adjustments
#### Growth (Children/Adolescents)
- Add Energy Cost of Growth (ECG) based on age/sex (Tables 5-11 & 5-12)
- Consider sub-service: `EnergyCostOfGrowthService(age, sex)`

#### Pregnancy
- Energy Cost of Tissue Accretion varies by pre-pregnancy BMI (Table 5-13):
  - Underweight: +300 kcal/d
  - Normal weight: +200 kcal/d
  - Overweight: +150 kcal/d
  - Obese: -50 kcal/d (mobilization)
- Only applies to 2nd and 3rd trimesters
- First trimester uses non-pregnant TEE
- Consider sub-service: `PregnancyEnergyAdjustmentService(pre_pregnancy_bmi, trimester)`

#### Lactation
- 0-6 months exclusive breastfeeding:
  - TEE + 540 kcal/d (milk production) – 140 kcal/d (energy mobilization)
  - Net: TEE + 400 kcal/d (approx, as per Table S-5, S-6 notes and Table 5-14 showing net 404 kcal/d)
- 7-12 months partial breastfeeding: TEE + 380 kcal/d (milk production)
- Consider sub-service: `LactationEnergyAdjustmentService(months_postpartum, exclusivity)`

## Output
```json
{
  "nutrient": "Energy",
  "eer_kcal": number,
  "sepv_kcal": number,
  "components": {
    "tee": number,
    "growth_adjustment": number | null,
    "pregnancy_adjustment": number | null,
    "lactation_adjustment": number | null
  }
}
```

## Dependencies

### PalCategoryDeterminationService
- **Input**: Description of daily activities (Table 7-1), or potentially step counts if robust correlations were established (though the document suggests this is weak)
- **Responsibility**: Classify the user into Inactive, Low Active, Active, or Very Active based on age-specific cutoffs (Table 5-4)
- **Output**: PAL Category string/enum

### BmiCalculationService
- **Input**: Weight (kg), Height (m)
- **Responsibility**: Calculate BMI
- **Output**: BMI value

### WeightStatusCategorizationService
- **Input**: BMI, age, sex (for children: percentiles; for adults: standard cutoffs; for pregnancy: pre-pregnancy BMI)
- **Responsibility**: Categorize as Underweight, Normal Weight, Overweight, Obese (and classes of obesity)
- **Output**: Weight status category

### ReferenceBodyDataService
- **Input**: Age group, sex, (optionally) weight status category, (optionally) country (US/Canada)
- **Responsibility**: Provide median heights and weights for reference individuals (Tables L-1 to L-8, M-1 to M-8)
- **Output**: Median height (cm), median weight (kg)