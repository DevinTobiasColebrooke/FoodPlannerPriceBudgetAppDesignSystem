# Utility Services

## DriLookupService

### Overview
Core service for fetching DRI values from the database based on nutrient, age, sex, and special conditions.
Its responsibilities include:
- Retrieving specific DRI values (EAR, RDA, AI, UL, AMDR percentages, CDRR for Sodium, etc.) based on the user's profile (age, sex, special conditions like pregnancy/lactation).
- Fetching AIs and ULs for Water, Potassium, Sodium, and Chloride from the database tables, indexed by age group, sex, and special conditions (pregnancy/lactation).

### Core Logic
- Implement robust logic to find the correct `AgeGroup` given age (in months or years) and any special conditions.
- Query the `DietaryReferenceIntakes` table using the determined `AgeGroup`, sex, and nutrient.

### Data Sources & Fetched Data
- **DGA Appendix 1 (Tables A1-1, A1-2, A1-3, A1-4):** Key for `DriLookupService` for Infants (0-12 months) and Toddlers (12-23 months), providing AIs and RDAs. For Children and Adults (Ages 2+), Pregnant, and Lactating Women, these tables are vital for macronutrient AMDR percentages, fiber g/1000kcal rule, % limits for added sugar/saturated fat, and absolute g/day for EFAs.
- **NCBI DRI Summary Tables (NBK545442 - Elements, NBK56068 - Vitamins):** Primary source for micronutrient RDAs/AIs/ULs.
- **Water DRI Report (e.g., Table S-2):** Source for AI for total water.
- The service will fetch:
    - Micronutrient RDAs/AIs/ULs.
    - Macronutrient AMDR percentages, fiber g/1000kcal rule, % limits for added sugar/saturated fat, and absolute g/day for EFAs.
    - Sodium CDRR.
    - AI for total water.

### Handling Edge Cases
- Must gracefully handle "n/a" (not applicable) or "ND" (not determinable) values from the tables, likely by not returning a value for those specific nutrient/age group combinations.

### DRI Definitions
- EAR (Estimated Average Requirement): Intake at which the risk of inadequacy is 0.5 (50%) for an individual.
- RDA (Recommended Dietary Allowance): Intake at which the risk of inadequacy is very small (2-3%).
- AI (Adequate Intake): Used when an RDA cannot be determined. Based on observed or experimentally determined approximations of nutrient intake by a group of apparently healthy people.
- UL (Tolerable Upper Intake Level): Highest average daily nutrient intake level likely to pose no risk of adverse health effects to almost all individuals.

### RDA Calculation
- RDA = EAR + 2 SD_EAR (if SD of requirement is known)
- RDA = 1.2 × EAR (if SD is unknown, assumes 10% CV)

### Inputs
- Nutrient name
- Age
- Sex
- Special conditions (pregnancy/lactation)

### Output
```json
{
  "ear": number | null,
  "ai": number | null,
  "rda": number | null,
  "ul": number | "ND",
  "criteria": string
}
```

II. Core Data / Reference Tables (Database - DriLookupService will query this):
* DRI Tables for Calcium, Phosphorus, Magnesium, Vitamin D, Fluoride.
* DRI tables for Water, Potassium, Sodium, and Chloride.
* These tables must be structured based on the life-stage groups, sex, and special conditions detailed in the report (Tables S-1 to S-6, pp. 15-20, and life stage definitions pp. 31-36).
* Key values to store per nutrient, per life-stage/sex/condition:
* Estimated Average Requirement (EAR) - if available (e.g., Phosphorus, Magnesium for older ages)
* Recommended Dietary Allowance (RDA) - calculated from EAR if available
* Adequate Intake (AI) - primary reference for Ca, Vit D, Fluoride, and infants for P, Mg
* Tolerable Upper Intake Level (UL) - critical for safety; note specific conditions like Mg UL for non-food sources, or ND for infant Ca/P ULs.
* Criteria for EAR/AI derivation (from Tables S-1 to S-5 and chapter text): This contextual information is vital for understanding the DRI values, even if not directly used in every calculation by the end-user calculator.
* Reference Weights and Heights (Table 1-3, p. 36) - For extrapolations or if user data is missing.
* Vitamin D Conversion: 1 µg cholecalciferol = 40 IU (p.18, p.255).

### Dependencies
`DriLookupService` is a dependency for:
- `MacronutrientServices` (to get AMDR percentages, RDAs, AIs).
- `MicronutrientServices` (to get RDA/AI and UL for each vitamin and mineral, and CDRR for Sodium).
- `WaterTargetService` (to get AI for total water).

1.  **`DriLookupService`** (Existing, role reinforced)
    *   **Responsibility updated to include:** Fetching AIs and ULs for Water, Potassium, Sodium, and Chloride from the database tables, indexed by age group, sex, and special conditions (pregnancy/lactation).
    *   This service will be querying tables that look like the summary tables S-2, S-3, and S-4 (pages 7, 10, 13 of the document), but will draw from the comprehensive DRI dataset.

General DRI Logic:
The process for setting RDAs (RDA = EAR + 2 SDEAR or RDA = 1.2 x EAR if SD is unknown, assuming 10% CV) (Page 3) is a core calculation principle. If your DriLookupService stores EARs, the individual nutrient services (or a shared helper) can calculate the RDA.
The definitions of EAR, RDA, AI, UL in Box S-1 (Page 2) are fundamental to understanding what each value represents and how it should be presented to the user.

Additional Implementation Notes:
- RAE conversions (Vitamin A)
- Factorial model components for breakdown display (Iron, Zinc)
- Growth factors for children (if not pre-calculated in DRI tables)

## UnitConversionService

### Overview
Handles unit conversions between different measurement systems.

### Key Conversions
- Vitamin D: 1 µg cholecalciferol = 40 IU

## RdaCalculationService

### Overview
Calculates RDA from EAR using standard formulas.

### Formulas
- RDA = EAR + 2 * SD_EAR
- RDA = EAR * 1.2 (if CV = 10% assumed)
- RDA = EAR * 1.3 (if CV = 15% assumed)

### Default
- Uses CV of 10% when SD is unknown