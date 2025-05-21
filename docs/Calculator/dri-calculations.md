# DRI Calculations and Lookups

## Overview
This document outlines which Dietary Reference Intake (DRI) values are calculated versus looked up from DRI tables.

## Core DRI Components

### Lookup-Based Components
- **EAR (Estimated Average Requirement)**: Direct value from DRI tables
- **AI (Adequate Intake)**: Direct value when EAR/RDA cannot be set
- **UL (Tolerable Upper Intake Level)**: Direct value
- **AMDR Percentages**: Direct values (e.g., 45-65% for carbohydrates)
- **Folate Needs (DFE)**: DRI values expressed in DFE
- **Niacin Needs (NE)**: DRI values expressed in NE

### Calculation-Based Components
- **RDA (Recommended Dietary Allowance)**:
  - Calculated from EAR: `RDA = EAR + 2 * SD_EAR`
  - Or if SD unknown: `RDA = 1.2 * EAR` (assuming 10% CV)
- **EER (Estimated Energy Requirement)**: Based on formulas
- **AMDR Grams**: Requires EER and kcal/g for macronutrient
- **Other % EER based limits**: Requires EER

## Service-Specific Requirements

### EnergyRequirementService
- **Primary Action**: Calculation
- **Inputs**: Age, Sex, Height, Weight, PAL
- **Lookups**: Formula coefficients
- **Output**: EER value

### Macronutrient Services
#### CarbohydrateService, ProteinService, FatService
- **Lookups**:
  - RDA/AI in grams
  - AMDR percentages
  - Percentage-based limits
- **Calculations**:
  - RDA in total grams (if primary DRI is g/kg)
  - AMDR in grams (requires EER)
  - Gram-based limits from % EER

### FiberService
- **Lookup**: AI for fiber (g/day)
- **Calculation** (if needed): `(target_g_per_1000_kcal * EER_kcal) / 1000`

### Vitamin Services
#### VitaminAService
- **Lookups**:
  - EAR, RDA, AI in µg RAE
  - UL for preformed Vitamin A
- **Calculations**:
  - RAE conversions from various sources
  - Sum of all RAE sources

#### VitaminCService
- **Lookup**: Base EAR/RDA/AI/UL
- **Calculation**: Smoker adjustment (+35mg)

### Mineral Services
#### IronService
- **Lookups**: EAR, RDA, AI, UL
- **Calculations** (if needed):
  - Bioavailability adjustments
  - Menarche status adjustments

#### ZincService
- **Lookups**: EAR, RDA, AI, UL
- **Calculations** (if needed):
  - Bioavailability adjustments
  - Dietary factor adjustments

## Special Cases

### Child DRI Extrapolation
For EAR/RDA:
```
EAR_child = EAR_adult × (Weight_child / Weight_adult)^0.75 × (1 + growth factor)
```

For ULs:
```
UL_child = UL_adult × (Weight_child / Weight_adult)
```

### ChromiumService
- **Lookup**: AI values
- **Calculation** (if needed): `AI = EER * (13.4 µg / 1000 kcal)`

## Service Responsibilities

### DriLookupService
- Primary interface for DRI data tables
- Stores reference weights and growth factors
- Handles basic lookups

### Individual Nutrient Services
- Call DriLookupService for base values
- Perform specific calculations
- Handle special conditions
- Implement bioavailability adjustments

### NutrientProfileCalculatorService
- Orchestrates service calls
- Aggregates results
- No primary DRI calculations 