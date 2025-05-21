# Nutrient Calculator: Service-Oriented Outline

## I. Core User Inputs

Collected via UI and passed to services:
1.  Age (Years, or months for infants/toddlers)
2.  Sex (Male, Female, or specific intersex considerations if applicable to DRIs)
3.  Height (cm or inches)
4.  Weight (kg or lbs)
5.  Physical Activity Level (PAL) - (Sedentary, Low Active, Active, Very Active - or a way to calculate this)
6.  Special Conditions (Flags):
    *   Pregnant (Trimester: 1st, 2nd, 3rd)
    *   Lactating (Months postpartum: 0-6, 7-12)
    *   Smoker (for Vitamin C, etc.)
    *   Vegetarian/Vegan (for Iron, B12, Zinc, Protein quality considerations)
    *   Specific medical conditions (Generally out of scope for a basic calculator, but good to acknowledge if future DRIs consider them)

## II. Core Data / Reference Tables
To be stored, likely in database:
1.  Dietary Reference Intake (DRI) Tables for:
    *   Estimated Average Requirements (EARs)
    *   Recommended Dietary Allowances (RDAs)
    *   Adequate Intakes (AIs)
    *   Tolerable Upper Intake Levels (ULs)
    *   Acceptable Macronutrient Distribution Ranges (AMDRs)
    *   Estimated Energy Requirements (EER) formulas/coefficients
2.  These tables will be indexed by: Age group, Sex, Special Condition (Pregnancy/Lactation trimester/month).
3.  Food Composition Database (If the calculator will also analyze food intake - this is a bigger step, let's assume for now it's just calculating needs).

## III. Main Orchestration Service: `NutrientProfileCalculatorService`
*   **Input**: User attributes (age, sex, weight, height, PAL, special conditions).
*   **Responsibility**:
    *   Call individual nutrient services to get their respective recommendations.
    *   Aggregate results into a comprehensive nutrient profile.
*   **Output**: A hash/object containing all calculated nutrient needs (EER, RDAs/AIs, ULs for all relevant nutrients).
*   **Dependencies**: All major nutrient services below.

## IV. Macronutrient Calculation Services

### 1. `EnergyRequirementService` (EER Calculator)
*   **Input**: Age, sex, weight, height, PAL, pregnancy/lactation status.
*   **Responsibility**: Calculate Total Daily Energy Expenditure based on established EER formulas (e.g., from IOM, considering TEE + Energy Deposition for growth/pregnancy/lactation). This will likely involve sub-methods for different life stages.
*   **Output**: Estimated Energy Requirement (kcal/day).
*   **Dependencies**: None external, but complex internal logic.

### 2. `CarbohydrateService`
*   **Input**: Age, sex, EER (from `EnergyRequirementService`), special conditions.
*   **Responsibility**:
    *   Calculate RDA/AI for carbohydrates (g/day) - e.g., based on brain glucose utilization.
    *   Calculate AMDR for carbohydrates (% of EER, and converted to g/day).
    *   Recommend limits for Added Sugars (% of EER, from DGA).
*   **Output**: `{ rda_g: X, ai_g: Y, amdr_percent_min: A, amdr_percent_max: B, amdr_g_min: C, amdr_g_max: D, added_sugars_limit_percent: E, added_sugars_limit_g: F }`
*   **Dependencies**: `DriLookupService` (for RDA/AI), `EnergyRequirementService` (for EER to calculate AMDR grams).

### 3. `ProteinService`
*   **Input**: Age, sex, weight, EER, special conditions (pregnancy, lactation, potentially vegetarian status for quality adjustment if advanced).
*   **Responsibility**:
    *   Calculate RDA/AI for protein (g/day and g/kg body weight).
    *   Calculate AMDR for protein (% of EER, and converted to g/day).
*   **Output**: `{ rda_g_per_kg: X, rda_g_total: Y, ai_g_total: Z, amdr_percent_min: A, ... }`
*   **Dependencies**: `DriLookupService`, `EnergyRequirementService`.

### 4. `FatService` (Total Fat and Fatty Acids)
*   **Input**: Age, sex, EER, special conditions.
*   **Responsibility**:
    *   Calculate AI for Total Fat (infants).
    *   Calculate AMDR for Total Fat (% of EER, and g/day).
    *   Calculate AIs for Essential Fatty Acids (Linoleic, Alpha-Linolenic) (g/day).
    *   Recommend limits for Saturated Fat & Trans Fat (% of EER, from DGA).
*   **Output**: `{ total_fat_amdr_percent_min: A, ..., linoleic_ai_g: X, ala_ai_g: Y, saturated_fat_limit_percent: Z, ... }`
*   **Dependencies**: `DriLookupService`, `EnergyRequirementService`.

### 5. `FiberService`
*   **Input**: Age, sex, EER (often recommended as g/1000 kcal).
*   **Responsibility**: Calculate AI for Total Fiber (g/day).
*   **Output**: `{ ai_g: X }`
*   **Dependencies**: `DriLookupService`, `EnergyRequirementService`.

### 6. `WaterService`
*   **Input**: Age, sex, potentially PAL and environmental factors (if advanced).
*   **Responsibility**: Calculate AI for Total Water (L/day, including from beverages and food).
*   **Output**: `{ ai_liters: X }`
*   **Dependencies**: `DriLookupService`.

## V. Micronutrient Calculation Services - Vitamins
For each vitamin (A, C, D, E, K, Thiamin, Riboflavin, Niacin, B6, Folate, B12, Pantothenic Acid, Biotin, Choline):
*   **`[VitaminName]Service`** (e.g., `VitaminDService`)
    *   **Input**: Age, sex, special conditions (e.g., pregnancy/lactation for folate, smoking for Vit C).
    *   **Responsibility**: Determine RDA/AI and UL.
    *   **Output**: `{ rda: X, ai: Y, ul: Z }` (units specific to vitamin, e.g., µg, mg, IU)
    *   **Dependencies**: `DriLookupService`.
*   **Special Notes**:
    *   Vitamin A: RAE conversions.
    *   Niacin: NE conversions from tryptophan.
    *   Folate: DFE conversions.
    *   Vitamin E: alpha-tocopherol forms.

## VI. Micronutrient Calculation Services - Minerals
For each mineral (Calcium, Phosphorus, Magnesium, Iron, Zinc, Iodine, Selenium, Copper, Manganese, Fluoride, Potassium, Sodium, Chloride):
*   **`[MineralName]Service`** (e.g., `CalciumService`)
    *   **Input**: Age, sex, special conditions (e.g., pregnancy for iron, vegetarian for iron/zinc bioavailability).
    *   **Responsibility**: Determine RDA/AI and UL (and CDRR for Sodium).
    *   **Output**: `{ rda: X, ai: Y, ul: Z, cdrr: W }` (units specific to mineral, e.g., mg, µg)
    *   **Dependencies**: `DriLookupService`.
*   **Special Notes**:
    *   Sodium: Has a CDRR now, not just an AI/UL based on older DRIs.
    *   Iron: Different bioavailability assumptions for vegetarians.

## VII. Utility / Helper Services

### 1. `DriLookupService`
*   **Overview**: Core service for fetching DRI values from the database based on nutrient, age, sex, and special conditions.
*   **Responsibilities**:
    *   Retrieving specific DRI values (EAR, RDA, AI, UL, AMDR percentages, CDRR for Sodium, etc.) based on the user's profile (age, sex, special conditions like pregnancy/lactation).
    *   Fetching AIs and ULs for Water, Potassium, Sodium, and Chloride from the database tables, indexed by age group, sex, and special conditions (pregnancy/lactation).
    *   Query the DRI database tables to fetch the appropriate EAR, RDA, AI, UL, AMDR percentages. Handle age group bracketing and special condition lookups.
*   **Core Logic**:
    *   Implement robust logic to find the correct `AgeGroup` given age (in months or years) and any special conditions.
    *   Query the `DietaryReferenceIntakes` table using the determined `AgeGroup`, sex, and nutrient.
*   **Data Sources & Fetched Data**:
    *   **DGA Appendix 1 (Tables A1-1, A1-2, A1-3, A1-4):** Key for `DriLookupService` for Infants (0-12 months) and Toddlers (12-23 months), providing AIs and RDAs. For Children and Adults (Ages 2+), Pregnant, and Lactating Women, these tables are vital for macronutrient AMDR percentages, fiber g/1000kcal rule, % limits for added sugar/saturated fat, and absolute g/day for EFAs.
    *   **NCBI DRI Summary Tables (NBK545442 - Elements, NBK56068 - Vitamins):** Primary source for micronutrient RDAs/AIs/ULs.
    *   **Water DRI Report (e.g., Table S-2):** Source for AI for total water.
    *   The service will fetch: Micronutrient RDAs/AIs/ULs, Macronutrient AMDR percentages, fiber g/1000kcal rule, % limits for added sugar/saturated fat, absolute g/day for EFAs, Sodium CDRR, AI for total water.
*   **Handling Edge Cases**: Must gracefully handle "n/a" (not applicable) or "ND" (not determinable) values from the tables, likely by not returning a value for those specific nutrient/age group combinations.
*   **DRI Definitions**:
    *   EAR (Estimated Average Requirement): Intake at which the risk of inadequacy is 0.5 (50%) for an individual.
    *   RDA (Recommended Dietary Allowance): Intake at which the risk of inadequacy is very small (2-3%).
    *   AI (Adequate Intake): Used when an RDA cannot be determined. Based on observed or experimentally determined approximations of nutrient intake by a group of apparently healthy people.
    *   UL (Tolerable Upper Intake Level): Highest average daily nutrient intake level likely to pose no risk of adverse health effects to almost all individuals.
*   **RDA Calculation Principle**:
    *   RDA = EAR + 2 SD_EAR (if SD of requirement is known)
    *   RDA = 1.2 × EAR (if SD is unknown, assumes 10% CV)
*   **Input**: Nutrient name, age, sex, special conditions.
*   **Output**: The specific DRI value(s) requested (e.g., `{ "ear": number | null, "ai": number | null, "rda": number | null, "ul": number | "ND", "criteria": string }`).
*   **Dependencies**: Database access.
*   **Database Storage Notes**:
    *   Store DRI Tables for Calcium, Phosphorus, Magnesium, Vitamin D, Fluoride, Water, Potassium, Sodium, Chloride.
    *   Structure tables based on life-stage groups, sex, special conditions (e.g., Tables S-1 to S-6, pp. 15-20, life stage definitions pp. 31-36).
    *   Key values: EAR, RDA (or calculate from EAR), AI, UL, Criteria for EAR/AI derivation.
    *   Store Reference Weights and Heights (Table 1-3, p. 36).
    *   Vitamin D Conversion: 1 µg cholecalciferol = 40 IU.
*   **Dependent Services**: `MacronutrientServices`, `MicronutrientServices`, `WaterTargetService`.
*   **Additional Implementation Notes**: RAE conversions (Vitamin A), Factorial model components (Iron, Zinc), Growth factors for children.

### 2. `UnitConversionService` (Optional, but useful)
*   **Overview**: Handles unit conversions between different measurement systems.
*   **Input**: Value, from_unit, to_unit.
*   **Responsibility**: Convert between units (e.g., lbs to kg, inches to cm, IU to µg for certain vitamins).
*   **Key Conversions**:
    *   Vitamin D: 1 µg cholecalciferol = 40 IU.
*   **Output**: Converted value.

### 3. `RdaCalculationService` (Consider integrating into `DriLookupService` or individual Nutrient Services)
*   **Overview**: Calculates RDA from EAR using standard formulas.
*   **Formulas**:
    *   RDA = EAR + 2 * SD_EAR
    *   RDA = EAR * 1.2 (if CV = 10% assumed, default)
    *   RDA = EAR * 1.3 (if CV = 15% assumed)
*   **Note**: The principle `RDA = 1.2 × EAR` (if SD is unknown, assumes 10% CV) is a core calculation principle. If `DriLookupService` stores EARs, the individual nutrient services (or `DriLookupService` itself) can calculate the RDA.

### 4. `PhysicalActivityCoefficientService` (If PAL is not directly provided)
*   **Input**: Description of daily activities or selection from predefined categories.
*   **Responsibility**: Determine the PAL coefficient for EER calculations.
*   **Output**: PAL value (e.g., 1.2, 1.4, 1.6).

## VIII. Expected Outputs
From `NutrientProfileCalculatorService`:
*   A comprehensive hash or object containing:
    *   EER (kcal)
    *   For each Macronutrient: RDA/AI (grams and/or %EER), AMDR (grams and %EER), specific limits (e.g., added sugar, saturated fat).
    *   For each Vitamin: RDA/AI (in appropriate units), UL.
    *   For each Mineral: RDA/AI (in appropriate units), UL (and CDRR for Sodium).

## IX. Example Data Flow (Simplified for Carbohydrates)
1.  `NutrientProfileCalculatorService` receives user inputs.
2.  Calls `EnergyRequirementService` with (age, sex, wt, ht, PAL) -> returns EER (e.g., 2000 kcal).
3.  Calls `CarbohydrateService` with (age, sex, EER=2000 kcal).
    *   `CarbohydrateService` calls `DriLookupService` for Carb RDA/AI based on (age, sex) -> returns 130g RDA.
    *   `CarbohydrateService` calls `DriLookupService` for Carb AMDR % -> returns 45-65%.
    *   `CarbohydrateService` calculates AMDR grams: `(0.45 * 2000 / 4)` to `(0.65 * 2000 / 4)`.
    *   `CarbohydrateService` calls `DriLookupService` for Added Sugar limit % -> returns <10%.
    *   `CarbohydrateService` calculates Added Sugar limit grams: `(0.10 * 2000 / 4)`.
    *   `CarbohydrateService` returns its hash of results.
4.  `NutrientProfileCalculatorService` aggregates this with results from other services.

## X. Next Steps for You
1.  **Database Schema for DRI Tables**: Design how you'll store the detailed DRI tables (like A1-1 to A3-5 from the PDF you'll be using). Consider tables for `age_groups`, `sexes`, `special_conditions`, `nutrients`, and a main `dri_values` table that links them.
2.  **Prioritize Nutrients**: Decide which nutrients to implement first. Macronutrients and EER are fundamental.
3.  **For each nutrient/study you upload**:
    *   Identify the specific inputs needed for its calculation.
    *   Document the calculation logic/formulas.
    *   Note any sub-calculations or lookups required (which will inform the methods within its service or helper services).
    *   Define the output units and values (RDA, AI, UL, etc.).

This structure should provide a flexible and maintainable system. As you upload individual studies, you can flesh out the Responsibility and internal logic for each `[NutrientName]Service`. Good luck!

## XI. Calculation vs. Lookup Analysis

### Summary Table: DRI Value Type vs. Method

| Nutrient   | DRI Value Type                     | Lookup or Calculation? | Notes                                                                                                                                                                                                     |
| :--------- | :--------------------------------- | :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Calcium    | AI (Adequate Intake)               | Lookup                 | The report sets AIs for all life stages for Calcium (Table S-1).                                                                                                                                          |
|            | EAR                                | N/A                    | Not established for Calcium in this report.                                                                                                                                                               |
|            | RDA                                | N/A                    | Not established for Calcium (as EAR is not established).                                                                                                                                                    |
|            | UL (Tolerable Upper Intake Level)  | Lookup                 | Provided in Table S-6. Note: ND (Not Determinable) for infants 0-12 months.                                                                                                                               |
| Phosphorus | AI (Infants 0-12m)                 | Lookup                 | Table S-2 provides AIs for infants.                                                                                                                                                                       |
|            | EAR (>1 year)                      | Lookup                 | Table S-2 provides EARs for children and adults.                                                                                                                                                          |
|            | RDA (>1 year)                      | Lookup (or Calculation)| Table S-2 provides RDAs. Fundamentally, `RDA = EAR + 2 SD` (or `EAR * 1.2` if CV=10%). Your `DriLookupService` could store the pre-calculated RDA, or your `PhosphorusService` could calculate it from the looked-up EAR. |
|            | UL                                 | Lookup                 | Provided in Table S-6. Note: ND for infants 0-12 months.                                                                                                                                                  |
| Magnesium  | AI (Infants 0-12m)                 | Lookup                 | Table S-3 provides AIs for infants.                                                                                                                                                                       |
|            | EAR (>1 year)                      | Lookup                 | Table S-3 provides EARs for children and adults (note gender differences and pregnancy adjustment).                                                                                                       |
|            | RDA (>1 year)                      | Lookup (or Calculation)| Table S-3 provides RDAs. Similar to Phosphorus, this can be a lookup if your DB stores it, or a calculation from the EAR by `MagnesiumService`.                                                              |
|            | UL (Supplemental)                  | Lookup                 | Provided in Table S-6. Crucially, this UL applies only to magnesium from pharmacological agents (supplements, medications) and NOT from food/water. Your calculator needs to message this if assessing intake against the UL. |
| Vitamin D  | AI                                 | Lookup                 | The report sets AIs for all life stages for Vitamin D (Table S-4). These AIs assume minimal/no sun exposure.                                                                                              |
|            | EAR                                | N/A                    | Not established for Vitamin D in this report.                                                                                                                                                             |
|            | RDA                                | N/A                    | Not established for Vitamin D.                                                                                                                                                                            |
|            | UL                                 | Lookup                 | Provided in Table S-6.                                                                                                                                                                                    |
|            | µg to IU conversion                | Calculation            | 1 µg cholecalciferol = 40 IU. This conversion would be done by `VitaminDService` or `UnitConversionService`.                                                                                               |
| Fluoride   | AI                                 | Lookup                 | The report sets AIs for all life stages for Fluoride (Table S-5). Note gender differences from age 19+.                                                                                                   |
|            | EAR                                | N/A                    | Not established for Fluoride in this report.                                                                                                                                                              |
|            | RDA                                | N/A                    | Not established for Fluoride.                                                                                                                                                                             |
|            | UL                                 | Lookup                 | Provided in Table S-6.                                                                                                                                                                                    |

### Summary of Calculations Your Application Will Likely Perform:
*   **RDA from EAR**: If your database primarily stores EARs (and SD<sub>EAR</sub> or an assumed CV, like 10%), then the individual nutrient services for Phosphorus and Magnesium (for ages >1 year) will need to calculate the RDA.
    *   `RDA = EAR + 2 * SD_EAR`
    *   `RDA = EAR * 1.2` (if a Coefficient of Variation of 10% is assumed, as is common when SD is not available, per page 3 of the report).
*   **Vitamin D Unit Conversion**: Converting between micrograms (µg) and International Units (IU).
*   **Adjustments for Pregnancy (Magnesium)**: The EAR for Magnesium during pregnancy involves adding a fixed amount (+35 mg/day) to the non-pregnant EAR based on lean body mass accretion. This is a simple calculation based on a looked-up base EAR and a fixed addition.
*   **Age/Weight Based Calculations for ULs (Children - less likely needed for these 5)**: While the ULs in Table S-6 are given as absolute amounts for age groups, for some nutrients not in this report, ULs for children might be extrapolated based on body weight from adult ULs. For the 5 nutrients here, the ULs are directly provided for children. The Fluoride UL was derived using mg/kg/day (0.10 mg/kg/day) and reference weights to arrive at the absolute values in Table S-6 (p.309), but your application would lookup the final absolute value.

### Primarily DRI Table Lookups (via `DriLookupService`):
These are values that are generally pre-defined for age/sex/life-stage groups and stored in your database based on the DRI reports.
*   **Protein (RDA/AI in g/kg/day)**:
    *   The fundamental RDA or AI for protein is often expressed per kilogram of body weight. This value is a direct lookup.
    *   *Table Example*: The protein RDA table in the IOM 2002/2005 Macronutrient report.
*   **Amino Acids (EAR/RDA in mg/kg/day)**:
    *   The requirements for each indispensable amino acid are typically listed per kilogram of body weight.
    *   *Table Example*: Appendix E from the "Essential Guide" (page 459+ of your combined PDF).
*   **Vitamins (RDA/AI/UL in specific units like µg, mg, IU)**:
    *   Most vitamin requirements (RDA or AI) and their Tolerable Upper Intake Levels (UL) are direct lookups based on age, sex, and special conditions (pregnancy, lactation).
    *   *Table Example*: Table A1-1, A1-2, A1-3, A1-4 for specific vitamins.
*   **Minerals (RDA/AI/UL/CDRR in specific units like mg, µg)**:
    *   Similar to vitamins, most mineral requirements (RDA or AI), their ULs, and the CDRR for Sodium are direct lookups.
    *   *Table Example*: Table A1-1, A1-2, A1-3, A1-4 for specific minerals.
*   **Total Fat (AI for infants in g/day )**:
    *   The AI for total fat for infants is a direct lookup.
    *   *Table Example*: Table 1 from the "Dietary Fat: Total Fat and Fatty Acids" section of the "Essential Guide" (page 122 of your combined PDF).
*   **Essential Fatty Acids (Linoleic & Alpha-Linolenic AIs in g/day )**:
    *   The AIs for these are direct lookups.
    *   *Table Example*: Table 1 from the "Dietary Fat: Total Fat and Fatty Acids" section.
*   **Total Carbohydrates (RDA/AI in g/day )**:
    *   The RDA (or AI for infants) for total carbohydrates is a direct lookup (often around 130g/day for ages 1+ to support brain function).
    *   *Table Example*: Table 1 from "Dietary Carbohydrates: Sugars and Starches" (page 102 of your combined PDF).
*   **Fiber (AI in g/1000 kcal or g/day )**:
    *   The AI for fiber is often expressed as g/1000 kcal, which is a lookup. Sometimes, DRI summary tables will pre-calculate the g/day based on reference EERs for age/sex groups (like in Table A1-2). Your service might store both or just the g/1000 kcal rate.
    *   *Table Example*: Table 1 from "Fiber" (page 110 of your combined PDF).
*   **Water (AI in L/day)**:
    *   The AI for total water is a direct lookup.
    *   *Table Example*: Table 1 from "Water" (page 156 of your combined PDF).
*   **AMDR Percentages (for Protein, Fat, Carbohydrates)**:
    *   The acceptable percentage ranges for these macronutrients are direct lookups.
    *   *Table Example*: Table 1 from "Macronutrients, Healthful Diets, and Physical Activity" (page 70 of your combined PDF) or Table A1-2.
*   **Qualitative Limits/Recommendations (Added Sugars, Saturated Fat, Trans Fat, Cholesterol)**:
    *   The percentage limits (e.g., <10% for added sugars, <10% for saturated fat) or qualitative advice ("as low as possible") are direct lookups, often sourced from the Dietary Guidelines for Americans (DGA) but referenced within DRI discussions.
    *   *Table Example*: These are often stated in text within the DRI reports or DGA, rather than a structured table like RDAs. Your `DriLookupService` might retrieve these as specific percentage values or text strings.

### Requires Calculations (within specific services, often using looked-up DRI values):
*   **Estimated Energy Requirement (EER in kcal/day) - `EnergyRequirementService`**:
    *   This is a major calculation. It uses complex equations based on age, sex, weight, height, PAL, and life stage (pregnancy, lactation, growth). The coefficients for these equations are looked up (or hardcoded if they are stable across DRI versions), but the final EER is calculated.
    *   *Table Example*: Tables S-1 to S-6 and 5-5 to 5-19 from the "Dietary Reference Intakes for Energy" document detail the equations and their components.
*   **Protein (Total g/day from g/kg RDA/AI) - `ProteinService`**:
    *   Looks up the g/kg/day RDA/AI.
    *   Calculates total grams: `(g/kg/day RDA/AI) * user's body weight (kg)`.
*   **Amino Acids (Total mg/day from mg/kg EAR/RDA) - `AminoAcidsService`**:
    *   Looks up the mg/kg/day EAR/RDA for each amino acid.
    *   Calculates total milligrams: `(mg/kg/day EAR/RDA) * user's body weight (kg)`.
*   **Macronutrient Grams from AMDRs (Protein, Fat, Carbohydrates) - `ProteinService`, `TotalFatService`, `CarbohydrateService`**:
    *   Looks up the AMDR percentage range (e.g., 45-65% for carbs).
    *   Calculates the gram range:
        *   Min grams = `(Min % AMDR / 100) * EER (kcal) / (kcal per gram of that macronutrient: 4 for carbs/protein, 9 for fat)`.
        *   Max grams = `(Max % AMDR / 100) * EER (kcal) / (kcal per gram of that macronutrient)`.
    *   Requires the EER from `EnergyRequirementService`.
*   **Fiber (Total g/day from g/1000 kcal AI) - `FiberService`**:
    *   Looks up the AI in g/1000 kcal (e.g., 14g/1000 kcal).
    *   Calculates total grams: `(AI in g/1000 kcal) * (EER / 1000)`.
    *   Requires the EER from `EnergyRequirementService`.
*   **Added Sugars (Gram limit from % EER) - `AddedSugarsService`**:
    *   Looks up the percentage limit (e.g., <10%).
    *   Calculates gram limit: `(Limit % / 100) * EER (kcal) / 4 (kcal/g for sugars)`.
    *   Requires the EER from `EnergyRequirementService`.
*   **Saturated Fat (Gram limit from % EER) - `SaturatedFatService`**:
    *   Looks up the percentage limit (e.g., <10%).
    *   Calculates gram limit: `(Limit % / 100) * EER (kcal) / 9 (kcal/g for fat)`.
    *   Requires the EER from `EnergyRequirementService`.
*   **Monounsaturated Fatty Acids (Calculated residual range in g/day ) - `MonounsaturatedFattyAcidsService`**:
    *   This is entirely a calculation based on the outputs of other services:
        *   Get Total Fat gram range (from `TotalFatService` using AMDR).
        *   Get Saturated Fat gram limit (from `SaturatedFatService`).
        *   Get Trans Fat recommendation (effectively 0g for calculation purposes, from `TransFattyAcidsService`).
        *   Get Total PUFA gram AIs/range (from `PolyunsaturatedFattyAcidsService`).
        *   MUFA (g) = `Total Fat (g) - Saturated Fat (g) - Trans Fat (g) - PUFA (g)`. This will result in a range.
*   **Prediction Intervals for EER (kcal/day) - `EnergyRequirementService`**:
    *   Looks up the SEPV (Standard Error of Predicted Value) for the relevant EER equation.
    *   Calculates 95% PI: `EER ± (1.96 * SEPV)`.
    *   The SEPV values are usually provided alongside the EER equations (e.g., Table 5-8 from the Energy DRI document).

### Service Responsibilities Summary:
*   `DriLookupService` is the primary interface to your stored DRI data tables.
*   `EnergyRequirementService` is the most calculation-intensive service, applying complex formulas.
*   Other macronutrient services (`ProteinService`, `FatService`, `CarbohydrateService`, `FiberService`, `AddedSugarsService`, `SaturatedFatService`) often perform calculations to convert percentage-based recommendations (AMDRs, DGA limits) or per-body-weight recommendations into absolute gram amounts, frequently depending on the EER output.
*   Micronutrient (`VitaminServices`, `MineralServices`) services are mostly direct lookups for their RDA/AI/UL.
*   Specialized services like `AminoAcidsService` also do g/kg to total g conversions.
*   `MonounsaturatedFattyAcidsService` and potentially some aspects of PUFA ranges within total fat AMDR involve residual calculations.

### Primarily LOOKUP from DRI Tables (via `DriLookupService`):
These are values that the DRI committees have established and tabulated based on age, sex, and life stage. Your `DriLookupService` will be responsible for fetching these.
*   **Water (Total Water)**:
    *   Adequate Intake (AI): This will be a direct lookup (e.g., 3.7 L/day for adult men 19-50). The criteria used to set it (median NHANES III intake) are already factored into the AI value.
    *   Tolerable Upper Intake Level (UL): Not established.
*   **Potassium**:
    *   Adequate Intake (AI): Direct lookup (e.g., 4.7 g/day for adults). The criteria (blood pressure, kidney stones, bone loss) and extrapolations for children are already factored into the AI values in the DRI tables.
    *   Tolerable Upper Intake Level (UL): Not established from food for healthy adults. This "not established" status would be what's "looked up" or noted.
*   **Sodium**:
    *   Adequate Intake (AI): Direct lookup (e.g., 1.5 g/day for adults 19-50).
    *   Tolerable Upper Intake Level (UL): Direct lookup (e.g., 2.3 g/day for adults).
*   **Chloride**:
    *   Adequate Intake (AI): Direct lookup. The DRI tables (like Table S-4 in the document) provide chloride values in g/day, meaning the molar equivalence to sodium AI has already been calculated by the DRI committee.
    *   Tolerable Upper Intake Level (UL): Direct lookup, for the same reason as the AI.
*   **Most Vitamins (RDA or AI)**:
    *   The RDA or AI for vitamins like Vitamin C, Thiamin, Riboflavin, Niacin (as NE), B6, Folate (as DFE), B12, Pantothenic Acid, Biotin, Choline, Vitamin A (as RAE), Vitamin D (as µg or IU), Vitamin E (as mg α-tocopherol), Vitamin K will be direct lookups from DRI tables based on age, sex, and special conditions (e.g., pregnancy, lactation, smoker for Vit C).
    *   ULs for Vitamins: These will also be direct lookups from DRI tables.
*   **Most Minerals (RDA or AI)**:
    *   The RDA or AI for minerals like Calcium, Phosphorus, Magnesium, Iron, Zinc, Iodine, Selenium, Copper, Manganese, Fluoride will be direct lookups from DRI tables.
    *   ULs for Minerals: These will also be direct lookups.
*   **Fiber**:
    *   Adequate Intake (AI): Often expressed as g/day based on age/sex (lookup) or as g/1000 kcal. If the latter, it requires the EER calculation first. The primary AI values in g/day from DRI tables are lookups.

### Primarily CALCULATION (by dedicated services, often using looked-up DRI values as inputs):
These involve formulas or deriving values based on other calculated or looked-up data.
*   **Estimated Energy Requirement (EER) / Total Energy Expenditure (TEE)**:
    *   `EnergyRequirementService`: This is a major calculation. It uses specific formulas based on age, sex, weight, height, Physical Activity Level (PAL), and coefficients for pregnancy/lactation/growth. The PAL itself might be looked up or calculated by a sub-service (`PhysicalActivityCoefficientService`).
*   **Macronutrients (Carbohydrates, Protein, Fat) - Grams based on AMDR**:
    *   `CarbohydrateService`, `ProteinService`, `FatService`:
        *   Acceptable Macronutrient Distribution Range (AMDR) Percentages: The percentage ranges (e.g., Carbs 45-65% of energy) are looked up from DRI tables via `DriLookupService`.
        *   Grams from AMDR: Converting these percentages into gram ranges is a calculation:
            *   `(AMDR % / 100) * EER (kcal) / (kcal per gram of that macronutrient)`
        *   This requires the output from `EnergyRequirementService` (EER).
*   **Specific Macronutrient Sub-component Limits (in grams)**:
    *   `CarbohydrateService` (for Added Sugars):
        *   The recommendation (e.g., <10% of total energy from DGAs) is a looked-up guideline.
        *   Converting this to grams is a calculation: `(Percent limit / 100) * EER (kcal) / 4 (kcal/g for sugar)`. Requires EER.
    *   `FatService` (for Saturated Fat, Trans Fat):
        *   The recommendation (e.g., <10% of total energy for Saturated Fat from DGAs) is a looked-up guideline.
        *   Converting this to grams is a calculation: `(Percent limit / 100) * EER (kcal) / 9 (kcal/g for fat)`. Requires EER.
*   **Protein (if g/kg is the primary DRI)**:
    *   `ProteinService`:
        *   The RDA/AI in g/kg body weight is looked up.
        *   The total grams per day is a calculation: `(g/kg value) * body weight (kg)`.
*   **Fiber (if AI is based on kcal)**:
    *   `FiberService`:
        *   If the AI is expressed as, for example, 14g/1000 kcal, then the daily gram amount is a calculation: `(14g / 1000 kcal) * EER (kcal)`. Requires EER. (Though DRI tables often also provide direct g/day AIs which would be lookups).
*   **Vitamin Conversions (if input data is not in the standard DRI unit)**:
    *   `[VitaminName]Service` or `UnitConversionService`:
        *   If your application takes food data input that lists, for example, beta-carotene instead of RAE for Vitamin A, or different forms of Niacin or Folate, then the conversion to the standard DRI unit (RAE, NE, DFE) is a calculation using established conversion factors. The target RDA/AI in the standard unit is still a lookup.
*   **Sulfate (No direct requirement calculation, but estimation of provision)**:
    *   While there's no DRI for sulfate to calculate against, if your calculator wanted to estimate how much inorganic sulfate is potentially provided from the diet via sulfur amino acid metabolism, this would be a calculation based on the methionine and cysteine content of consumed protein. This is more of an analytical calculation than a requirement calculation.

### In Summary (Lookup vs. Calculation):
*   **Lookup-Heavy**: Water, Potassium, Sodium, Chloride AIs and ULs; most Vitamin and Mineral RDAs/AIs and ULs; AMDR percentages.
*   **Calculation-Heavy**: EER; gram amounts for macronutrients based on AMDRs; gram limits for added sugars/saturated fat; total protein if RDA is per kg; fiber if AI is per 1000 kcal; vitamin unit conversions if necessary.

### General Principles:
*   **Direct DRI Values (Lookup)**:
    *   **EAR (Estimated Average Requirement)**: If an EAR is established for a specific age/sex/condition, it's typically a direct value from DRI tables.
    *   **AI (Adequate Intake)**: When an AI is set (because an EAR cannot be determined), it's a direct value from DRI tables. The summary explicitly states AIs are set for choline, pantothenic acid, and biotin for all ages, and for all nutrients for infants.
    *   **UL (Tolerable Upper Intake Level)**: These are established upper limits and are direct values from DRI tables. The summary (Table S-2) provides some of these.
    *   **AMDR (Acceptable Macronutrient Distribution Range) percentages**: The percentage ranges (e.g., 45-65% for carbs) are direct values from DRI tables/guidelines.
    *   **Fixed Recommendations**: Specific fixed value recommendations (e.g., 400 µg/day folic acid for women capable of becoming pregnant) are essentially lookups based on a condition.
*   **Calculated Values (Requires Logic in Services)**:
    *   **RDA (Recommended Dietary Allowance)**:
        *   The DRI summary (page 3) states: `RDA = EAR + 2 * SD_EAR` or, if SD is unknown, `RDA = 1.2 * EAR` (assuming a 10% coefficient of variation).
        *   So, if your DRI database primarily stores EARs, the `[NutrientName]Service` will calculate the RDA.
        *   If your DRI database is populated with pre-calculated RDAs for every age/sex/condition, then it becomes a lookup. For flexibility and adherence to DRI methodology, calculating it from EAR is often preferred.
    *   **EER (Estimated Energy Requirement)**: This is almost always calculated using specific formulas based on age, sex, weight, height, and physical activity level (PAL). The coefficients for these formulas might be looked up.
    *   **AMDR (grams)**: The gram equivalents for macronutrient ranges are calculated using the EER and the AMDR percentages (e.g., `grams = (AMDR_percentage * EER_kcal) / kcal_per_gram_of_macronutrient`).
    *   **Specific Limits (grams)**: Similar to AMDR grams, if limits like "added sugars" or "saturated fat" are given as a % of EER, their gram equivalents will be calculated.
    *   **Conversions (if inputs are not in the final DRI unit)**:
        *   **DFE (Dietary Folate Equivalents)**: The requirements (EAR/RDA/AI) for folate are expressed in DFE (page 1). So, the `FolateService` will lookup the DFE requirement. If your application later takes user input of food folate and folic acid from supplements to assess intake, then a DFE calculation would be needed at that point.
        *   **NE (Niacin Equivalents)**: Similarly, niacin requirements are expressed in NE (page 10). The `NiacinService` will lookup the NE requirement. An NE calculation would be needed if assessing intake from tryptophan and preformed niacin.

### Breakdown by Service:
1.  **`EnergyRequirementService` (EER Calculator)**
    *   **Lookup**: Coefficients for EER equations (based on age, sex, PAL categories if not calculating PAL directly).
    *   **Calculation**: The EER value itself using the established formulas.
2.  **Macronutrient Services (`CarbohydrateService`, `ProteinService`, `FatService`)**
    *   **Lookup**:
        *   RDA/AI for the macronutrient (e.g., RDA for protein in g/kg).
        *   AMDR percentages for the macronutrient.
        *   Percentage-based limits (e.g., % EER for added sugars, saturated fat).
        *   AIs for essential fatty acids (in `FatService`).
    *   **Calculation**:
        *   RDA in total grams if the primary DRI is g/kg (`grams = g/kg * body_weight`).
        *   AMDR in grams (requires EER from `EnergyRequirementService`).
        *   Gram-based limits derived from % EER (requires EER).
3.  **`FiberService`**
    *   **Lookup**: AI for fiber (g/day).
    *   **Calculation (Potential)**: If DRIs express fiber needs as g/1000 kcal, then the service would calculate `(target_g_per_1000_kcal * EER_kcal) / 1000`.
4.  **`WaterService`**
    *   **Lookup**: AI for water (L/day).
5.  **Vitamin Services (e.g., `ThiaminService`, `RiboflavinService`, `FolateService`, etc.)**
    *   **Lookup**:
        *   EAR for the vitamin (if it exists for the demographic).
        *   AI for the vitamin (if EAR/RDA doesn't exist, e.g., for infants or for pantothenic acid, biotin, choline for all ages).
        *   UL for the vitamin.
        *   For Folate: The EAR/RDA/AI values looked up will be in DFE. The 400 µg folic acid recommendation for women capable of becoming pregnant is a conditional fixed value.
        *   For Niacin: The EAR/RDA/AI values looked up will be in NE.
    *   **Calculation**:
        *   RDA for the vitamin (derived from EAR using `RDA = 1.2 * EAR` or `RDA = EAR + 2 * SD_EAR`, unless RDA is directly stored and looked up).
6.  **Mineral Services (e.g., `CalciumService`, `IronService`, etc.)** (By analogy, as not detailed in this summary)
    *   **Lookup**:
        *   EAR for the mineral (if it exists).
        *   AI for the mineral (if EAR/RDA doesn't exist).
        *   UL for the mineral.
        *   CDRR for sodium.
    *   **Calculation**:
        *   RDA for the mineral (derived from EAR, unless RDA is directly stored and looked up).
7.  **`CholineService`**
    *   **Lookup**:
        *   AI for choline.
        *   UL for choline.
8.  **`PantothenicAcidService`, `BiotinService`**
    *   **Lookup**:
        *   AI for these vitamins.
        *   UL (which is "Not Determinable" (ND) based on the summary).

### Summary Table: Calculation vs. Lookup by DRI Component

| DRI Component                             | Primary Method        | Notes                                                                                             |
| :---------------------------------------- | :-------------------- | :------------------------------------------------------------------------------------------------ |
| EAR (Estimated Average Requirement)       | Lookup                | Direct value from DRI tables.                                                                     |
| RDA (Recommended Dietary Allowance)     | Calculation or Lookup | Calculated from EAR (e.g., `EAR * 1.2`) OR direct lookup if your DB stores pre-calculated RDAs.   |
| AI (Adequate Intake)                      | Lookup                | Direct value when EAR/RDA cannot be set.                                                          |
| UL (Tolerable Upper Intake Level)         | Lookup                | Direct value. Folate & Niacin ULs refer to synthetic forms. Many B-vits have "ND" for UL.         |
| EER (Estimated Energy Requirement)        | Calculation           | Based on formulas (coefficients might be lookup).                                                 |
| AMDR Percentages (Macronutrients)       | Lookup                | e.g., 45-65% for carbohydrates.                                                                   |
| AMDR Grams (Macronutrients)             | Calculation           | Requires EER and kcal/g for the macronutrient.                                                    |
| Folate Needs (DFE)                        | Lookup                | DRI values for folate (EAR/RDA/AI) are expressed in DFE.                                          |
| Niacin Needs (NE)                         | Lookup                | DRI values for niacin (EAR/RDA/AI) are expressed in NE.                                           |
| Specific Folic Acid Rec. (Pregnancy Capable)| Lookup (Conditional)  | 400 µg/day folic acid is a fixed value for a specific condition.                                  |
| Other % EER based limits (grams) (e.g. sugars)| Calculation           | Requires EER.                                                                                     |

### I. Nutrients Where DRIs are Primarily Looked Up (from S-Tables):
For most nutrients, the final EAR, RDA, AI, and UL values for a given life-stage group, sex, and special condition (pregnancy/lactation) will be directly looked up from the database tables you create based on Tables S-1 through S-10.
These include:
*   **Vitamin K (`VitaminKService`)**:
    *   **Lookup**: AI values (Table S-2).
    *   **Calculation**: None for the primary DRI. UL is not established.
*   **Copper (`CopperService`)**:
    *   **Lookup**: EAR, RDA, AI (for infants) (Table S-4), and UL (Table S-10).
    *   **Calculation (Potentially minor)**: If your DRI tables store only EARs and a CV, the service would calculate `RDA = EAR * 1.2` (or using a nutrient-specific CV if provided in the full chapter). However, the S-tables already provide the RDAs.
*   **Iodine (`IodineService`)**:
    *   **Lookup**: EAR, RDA, AI (for infants) (Table S-5), and UL (Table S-10).
    *   **Calculation (Potentially minor)**: Similar to Copper, RDA could be derived from EAR if not pre-tabulated.
*   **Manganese (`ManganeseService`)**:
    *   **Lookup**: AI values (Table S-7) and UL (Table S-10).
    *   **Calculation**: None for the primary DRI.
*   **Molybdenum (`MolybdenumService`)**:
    *   **Lookup**: EAR, RDA, AI (for infants) (Table S-8), and UL (Table S-10).
    *   **Calculation (Potentially minor)**: Similar to Copper.
*   **Boron, Nickel, Vanadium (`BoronService`, `NickelService`, `VanadiumService`)**:
    *   **Lookup**: UL values (Table S-10).
    *   **Calculation**: None. No EAR/RDA/AI established in this document.
*   **Arsenic, Silicon (`ArsenicService`, `SiliconService`)**:
    *   **Lookup**: N/A (No DRI values established in this document).
    *   **Calculation**: N/A. Output is informational.

### II. Nutrients/Situations Requiring More Specific Calculations within Services:
*   **Vitamin A (`VitaminAService`)**:
    *   **Lookup**: EAR, RDA, AI (for infants) in µg RAE (Table S-1), and UL for preformed Vitamin A (Table S-10).
    *   **Calculations (Potentially extensive if handling food input)**:
        *   **RAE Conversion**: This is the major calculation. If your calculator takes food composition data (e.g., from user input or a food database), it needs to:
            *   Convert µg of β-carotene (from food) to RAE: `µg β-carotene / 12`
            *   Convert µg of α-carotene (from food) to RAE: `µg α-carotene / 24`
            *   Convert µg of β-cryptoxanthin (from food) to RAE: `µg β-cryptoxanthin / 24`
            *   Convert IU of supplemental β-carotene to RAE: `IU supplemental β-carotene * 0.15 µg RAE/IU`
            *   Convert IU of preformed Vitamin A (retinol) to RAE: `IU retinol * 0.3 µg RAE/IU`
            *   Sum all RAE sources to get total Vitamin A intake in RAE.
        *   The UL is for preformed Vitamin A only. Carotenoids do not contribute to UL toxicity.
*   **Iron (`IronService`)**:
    *   **Lookup**: EAR, RDA, AI (for young infants) (Table S-6), and UL (Table S-10).
    *   **Calculations (Potentially, for advanced features or if DRIs were not pre-calculated for specific groups)**:
        *   **Factorial Modeling Components**: The DRIs for iron are based on factorial modeling (basal losses, menstrual losses, growth, pregnancy needs). While your service will likely use the final EAR/RDA from the S-tables, if you wanted to:
            *   Provide recommendations adjusted for different iron bioavailability (e.g., for vegetarians, who might have ~10% absorption vs. the ~18% assumed for mixed diets), your service would need to adjust the target dietary iron intake. The EAR in terms of absorbed iron would remain the same, but `Dietary Target = Absorbed EAR / Bioavailability_Fraction`.
            *   Dynamically calculate needs for adolescent girls based on menarche status (as suggested on p. 339 & 573, where girls who have started menstruating before age 14 have higher needs).
        *   The S-tables already incorporate these complexities into the final values, but understanding the underlying calculations would be necessary for such features.
*   **Zinc (`ZincService`)**:
    *   **Lookup**: EAR, RDA, AI (for young infants) (Table S-9), and UL (Table S-10).
    *   **Calculations (Potentially, for advanced features)**:
        *   Similar to iron, zinc DRIs are based on factorial modeling and assumptions about absorption (e.g., ~40% for adult men, page 473).
        *   If you wanted to adjust recommendations based on dietary factors affecting zinc bioavailability (e.g., high phytate diets common in some vegetarian patterns might reduce absorption), the `ZincService` would need logic similar to the iron service to adjust the dietary target.
*   **Chromium (`ChromiumService`)**:
    *   **Lookup**: AI values (Table S-3). UL is not established.
    *   **Calculations (If AIs weren't pre-tabulated for adults)**: The adult AIs are based on "Average chromium intake based on the chromium content of foods/1,000 kcal and average energy intake" (13.4 µg/1000 kcal).
        *   If the user provides their EER (Estimated Energy Requirement), the service could calculate a more personalized AI: `AI = EER * (13.4 µg / 1000 kcal)`. However, the S-table already provides AIs based on typical energy intakes for age/sex groups.

### Summary of Who Does What:
*   **`DriLookupService`**: Primarily responsible for looking up pre-calculated EAR, RDA, AI, and UL values from your DRI database (built from S-Tables). It will also need to store reference weights (Table 1-1) and growth factors (Table 2-1) if extrapolation logic is housed centrally or passed to nutrient services.
*   **Individual Nutrient Services (e.g., `VitaminAService`, `IronService`, `ZincService`)**:
    *   **Lookup**: Call `DriLookupService` to get the base DRI values.
    *   **Calculate**:
        *   Perform RAE conversions (Vitamin A).
        *   Potentially calculate RDA from EAR if not directly stored (`EAR * 1.2` or nutrient-specific CV).
        *   Implement advanced logic for bioavailability adjustments if desired (Iron, Zinc).
        *   Implement child extrapolation from adult DRIs if these values aren't pre-stored in your DRI tables for every detailed age break.
        *   Handle special conditions (pregnancy, lactation) mostly by looking up the specific DRI for that state.
*   **`EnergyRequirementService`**: Calculates EER, which is a key input if the `ChromiumService` (or others) dynamically calculates AIs based on energy intake, though the provided S-tables already give fixed AIs.
*   **`NutrientProfileCalculatorService`**: Orchestrates calls and aggregates results. It doesn't do primary DRI calculations itself but relies on the nutrient services.

### Primarily Calculation-Based (Often Rely on Lookups for Coefficients/Base Values):
*   **`EnergyRequirementService` (EER Calculator)**:
    *   **Calculation**: This is almost entirely calculation-based. It uses specific formulas (e.g., from the Institute of Medicine) that take Age, Sex, Height, Weight, and Physical Activity Level (PAL) as inputs.
    *   **Lookups (for coefficients)**: The formulas themselves use various coefficients that might differ by age group or physiological state (e.g., different coefficients for pregnancy energy deposition by trimester). These coefficients would be "looked up" by the service to plug into the main EER formula.
*   **Macronutrient Services (`CarbohydrateService`, `ProteinService`, `FatService`)**:
    *   **AMDR in Grams**:
        *   **Lookup**: The Acceptable Macronutrient Distribution Range (AMDR) as a percentage of total energy (e.g., Carbs: 45-65%) is a lookup from DRI tables.
        *   **Calculation**: Converting this percentage range into a gram range requires the EER (from `EnergyRequirementService`).
            *   *Example*: Grams of Carbs (min) = `(AMDR_Carb_Percent_Min / 100) * EER_kcal / 4_kcal_per_gram`
    *   **Specific Limits (e.g., Added Sugars, Saturated Fat) in Grams**:
        *   **Lookup**: The recommended limit as a percentage of total energy (e.g., Added Sugars <10% of EER) is typically looked up from dietary guidelines.
        *   **Calculation**: Converting this percentage into grams requires the EER.
            *   *Example*: Grams of Added Sugar Limit = `(Added_Sugar_Limit_Percent / 100) * EER_kcal / 4_kcal_per_gram`
*   **`FiberService`**:
    *   If recommended as g/1000 kcal:
        *   **Lookup**: The g/1000 kcal value.
        *   **Calculation**: Total Fiber (g) = `(Fiber_g_per_1000_kcal) * (EER_kcal / 1000)`
    *   If a direct AI is given (like in newer DRIs): Then it's a lookup (see below).
*   **`VitaminCService`**:
    *   **Lookup**: Base EAR/RDA/AI/UL from DRI tables.
    *   **Calculation**: If the user is a smoker, add 35 mg to the looked-up RDA. `Smoker_RDA = Base_RDA + 35mg`.
*   **`ProvitaminACarotenoidConversionService`**:
    *   **Lookup**: Conversion factors for specific carotenoids to Retinol Activity Equivalents (RAE) (e.g., 12 µg dietary β-carotene = 1 µg RAE). Note: These factors are NOT in the antioxidant document you provided and would come from a Vitamin A specific DRI report.
    *   **Calculation**: Total RAE = `(µg β-carotene / 12) + (µg α-carotene / 24) + ...`

### Primarily Lookup-Based (Values directly from DRI tables via `DriLookupService`):
*   **`CarbohydrateService`, `ProteinService`, `FatService`**:
    *   RDA/AI (in grams): Direct lookup (e.g., the 130g RDA for carbohydrates for adults is a direct value).
    *   AMDR (as a percentage range): Direct lookup.
    *   Specific Limits (as a percentage of EER): Direct lookup (e.g., DGA recommendation for <10% of calories from added sugars).
*   **`FiberService`**:
    *   AI (in grams): If a direct Adequate Intake is provided for age/sex, it's a lookup.
*   **`WaterService`**:
    *   AI (in Liters/day): Direct lookup.
*   **`VitaminCService`**:
    *   EAR, RDA (non-smoker), AI, UL: Direct lookup based on age, sex, pregnancy/lactation (from Table S-1 and S-4).
*   **`VitaminEService`**:
    *   EAR, RDA, AI (for 2R-stereoisomeric forms of α-tocopherol): Direct lookup (from Table S-2).
    *   UL (for any form of supplemental α-tocopherol): Direct lookup (from Table S-4).
*   **`SeleniumService`**:
    *   EAR, RDA, AI, UL: Direct lookup (from Table S-3 and S-4).
*   **Other Vitamin & Mineral Services** (e.g., Vitamin D, Calcium, Iron, etc. - when you add their specific studies):
    *   EAR, RDA, AI, UL, CDRR (for sodium): These will almost always be direct lookups from the respective DRI tables for those nutrients based on age, sex, and special conditions.
*   **`CarotenoidService` (based only on the current document)**:
    *   DRI status (EAR, RDA, AI), UL status: Lookup that returns "Not established" as per the document.

### Summary Table: Service Actions

| Service/Component                       | Primary Action by Nutrient Service (after user inputs & `DriLookupService`) | Relies on Lookups for...                    | Relies on other Services for...     |
| :-------------------------------------- | :------------------------------------------------------------------------- | :------------------------------------------ | :---------------------------------- |
| EER Calculation                         | Calculation (Formulas)                                                     | Coefficients                                | User Inputs (Wt, Ht, Age, PAL)      |
| Macronutrient RDA/AI (grams)            | Lookup                                                                     | DRI Values                                  | -                                   |
| Macronutrient AMDR (%)                  | Lookup                                                                     | DRI Values                                  | -                                   |
| Macronutrient AMDR (grams)              | Calculation                                                                | AMDR % (lookup)                             | EER                                 |
| Macronutrient Specific Limits (%)       | Lookup                                                                     | Guideline Values                            | -                                   |
| Macronutrient Specific Limits (grams)   | Calculation                                                                | Limit % (lookup)                            | EER                                 |
| Fiber AI (grams)                        | Lookup                                                                     | DRI Values                                  | -                                   |
| Fiber (if g/1000 kcal)                  | Calculation                                                                | g/1000kcal value (lookup)                   | EER                                 |
| Water AI                                | Lookup                                                                     | DRI Values                                  | -                                   |
| Vitamin C (non-smoker EAR/RDA/AI/UL)  | Lookup                                                                     | DRI Values                                  | -                                   |
| Vitamin C (smoker RDA)                  | Calculation (+35mg)                                                        | Base RDA (lookup)                           | -                                   |
| Vitamin E (EAR/RDA/AI/UL)               | Lookup                                                                     | DRI Values (noting specific forms)          | -                                   |
| Selenium (EAR/RDA/AI/UL)                | Lookup                                                                     | DRI Values                                  | -                                   |
| Other Vitamins/Minerals (DRIs)          | Lookup (Generally)                                                         | DRI Values                                  | -                                   |
| Carotenoids (non-ProA DRIs/UL)          | Lookup (Returns "Not Established" from current doc)                        | -                                           | -                                   |
| Provitamin A Carotenoid Conversion      | Calculation (in `VitaminAService`)                                         | Conversion Factors (lookup, future doc)     |                                     |

# DRI Calculator Backend Design Notes

## I. Overview
   - **Purpose**: This document outlines the design considerations and backend service architecture for a Dietary Reference Intake (DRI) calculator.
   - **Focus**: It details the mapping of various DRI tables and data sources to specific services, data seeding strategies, and the logic for calculating nutrient recommendations.

## II. Key Data Sources and Their Roles

   - **A. DGA Appendix 1: Nutritional Goals (Tables A1-1, A1-2, A1-3, A1-4)**
     - **Table A1-1**: Covers Infants (0-12 months) and Toddlers (12-23 months), providing Adequate Intakes (AIs) and Recommended Dietary Allowances (RDAs). Key for `DriLookupService` for these age groups. Notes "n/a" for Fiber & Total Lipid % for 6-11 month group.
     - **Table A1-2**: Addresses Children and Adults (Ages 2+). While it provides example micronutrient DRIs at specific calorie levels, its primary importance for the calculator lies in:
       - Macronutrient Acceptable Macronutrient Distribution Ranges (AMDRs) (e.g., Protein 10-35% kcal).
       - Fiber recommendation (14g/1000 kcal).
       - Limits for Added Sugars and Saturated Fatty Acids (e.g., <10% kcal).
       - AIs for Linoleic and Alpha-Linolenic Acid (g/day).
       - Absolute gram RDAs for Protein (e.g., 56g) and Carbohydrates (e.g., 130g) as minimums.
     - **Table A1-3**: Focuses on Pregnant Women. Similar to A1-2, crucial for macronutrient AMDRs, EFA AIs, and notes that while micronutrient RDAs/AIs often increase, the underlying macronutrient %AMDRs or g/kg recommendations might remain similar to non-pregnant, applied to a higher EER.
     - **Table A1-4**: Focuses on Lactating Women. Logic similar to A1-3 for macronutrient goals, applied to a lactation-adjusted EER.
     - **Overall Importance**: DGA Appendix 1 tables are vital for macronutrient rules, fiber, added sugar/saturated fat limits, and EFA AIs. The "Calorie Level Assessed" rows are illustrative; the underlying percentages or g/kg recommendations are key.

   - **B. DGA Appendix 2: Estimated Calorie Needs (Tables A2-1, A2-2, A2-3)**
     - **Table A2-1**: Provides direct EER values for Toddlers (12-23 months) based on age and sex, used by `EnergyRequirementService`.
     - **Table A2-2**: Underpins the EER equations for individuals aged 2 and older. `EnergyRequirementService` will implement these equations. This table can serve as a reference for testing.
     - **Table A2-3**: Provides the additional kilocalories needed during Pregnancy (Trimester 2: +340 kcal, Trimester 3: +452 kcal) and Lactation (0-6 months: +330 kcal, 7-12 months: +400 kcal) to be added to the pre-pregnancy EER by the `EnergyRequirementService`.
     - **Overall Importance**: These tables are the foundation for the `EnergyRequirementService` and subsequently for calculating gram targets for macronutrients.

   - **C. NCBI DRI Summary Tables (NBK545442 - Elements, NBK56068 - Vitamins)**
     - **"Dietary Reference Intakes: Elements" (NBK545442)**: Contains AIs, RDAs, and ULs for minerals (Sodium, Potassium, Chloride, Calcium, Phosphorus, Magnesium, Iron, Zinc, Copper, Iodine, Selenium, Manganese, Molybdenum, Fluoride, Chromium).
     - **"Dietary Reference Intakes: Vitamins" (NBK56068)**: Contains RDAs (bolded) and AIs (asterisked) for vitamins. ULs are in a separate sub-table within this source.
     - **Overall Importance**: These tables are the primary, consolidated source for seeding micronutrient EARs (if provided, though these tables mostly give RDA/AI directly), RDAs, AIs, and ULs. They effectively summarize information found in more detailed S-series tables.

   - **D. Individual DRI Study Tables (e.g., S-1 to S-10 from IOM reports)**
     - These are the original, detailed reports from the Institute of Medicine (IOM).
     - While the NCBI summary tables are preferred for direct seeding of RDA/AI/UL values, these reports contain:
       - The **"Criterion" column**: Invaluable for understanding the physiological basis for a DRI (e.g., "Maximizing plasma glutathione peroxidase activity" for Selenium RDA). Useful for documentation.
     - **Overall Importance**: Contextual understanding and detailed background.

   - **E. Footnotes (Across all DRI tables)**
     - **Critical Importance**: Footnotes often contain vital information that must be incorporated into service logic:
       - **Units**: Specific units like RAE (Retinol Activity Equivalents), DFE (Dietary Folate Equivalents), AT (Alpha-Tocopherol).
       - **Specific Populations**: Adjustments for smokers (e.g., Vitamin C UL), vegetarians (e.g., Iron).
       - **UL Derivations**: Clarifications if ULs are from food only or include supplements.
       - **"ND" (Not Determinable)**: Especially for ULs in infants; services must handle these cases.

## III. Core Services and Responsibilities

   - **1. `DriDataSeederService`**
     - **Responsibility**: Meticulously parse and seed data from all specified DRI tables into the database. This is a foundational step.
     - **Primary Data Sources**:
       - **Micronutrient EARs, RDAs, AIs, ULs**: Use "Dietary Reference Intakes: Elements" (NBK545442) and "Dietary Reference Intakes: Vitamins" (NBK56068) tables.
       - **Macronutrient AMDRs, Fiber Rules, Added Sugar/Saturated Fat Limits, EFA AIs**: Use Tables A1-2, A1-3, and A1-4 from DGA Appendix 1.
       - **EER Components**: Additional kcal for pregnancy/lactation (from DGA Appendix 2, Table A2-3) can be seeded or calculated directly by `EnergyRequirementService`.
     - **Key Tasks**:
       - Include source document/table references for each database entry.
       - Accurately distinguish between RDA (meaning an EAR was established) and AI (no EAR set) when seeding. The NCBI Vitamins table uses bolding/asterisks; the Elements table also makes these distinctions.

   - **2. `EnergyRequirementService` (EER Service)**
     - **Responsibility**: Calculate the user's specific Estimated Energy Requirement (EER).
     - **Calculation Logic**:
       - **Toddlers (12-23 months)**: Directly look up EER values from DGA Table A2-1.
       - **Ages 2+**: Implement the EER equations provided in IOM reports (as referenced by DGA Table A2-2).
       - **Pregnancy**: Add additional kcal to the pre-pregnancy EER: +340 kcal for Trimester 2, +452 kcal for Trimester 3 (from DGA Table A2-3). Trimester 1 EER is the same as pre-pregnancy.
       - **Lactation**: Add additional kcal to the pre-pregnancy EER: +330 kcal for 0-6 months, +400 kcal for 7-12 months (from DGA Table A2-3). These figures already account for milk production cost minus energy mobilization.
     - **Dependencies**: `PhysicalActivityCoefficientService` (to convert descriptive activity levels to PAL coefficients used in EER equations).

   - **3. `PhysicalActivityCoefficientService`**
     - **Responsibility**: Map user-inputted descriptive physical activity levels (e.g., "Sedentary", "Low Active", "Active", "Very Active") to the specific Physical Activity Level (PAL) coefficients required by the EER equations (from DGA Table A2-2).

   - **4. `MacronutrientServices` (or a single `MacronutrientTargetService`)**
     - **Input**: User's calculated EER (from `EnergyRequirementService`).
     - **Dependencies**: `DriLookupService` (to get AMDR percentages, RDAs, AIs).
     - **Protein**:
       - Lookup RDA (g/kg body weight or total grams).
       - Lookup AMDR percentages (e.g., 10-35% of kcal).
       - Calculate gram range: (e.g., 0.10 * EER / 4 kcal/g) to (0.35 * EER / 4 kcal/g).
     - **Carbohydrates**:
       - Lookup RDA (e.g., 130g/day for most, as a minimum).
       - Lookup AMDR percentages.
       - Calculate gram range based on EER.
       - Calculate Added Sugar limit (e.g., <10% of EER, per DGA).
     - **Fat**:
       - Lookup AI for infants (0-12 months).
       - Lookup AMDR percentages for 1 year and older.
       - Calculate gram range based on EER.
       - Lookup AIs for Linoleic Acid and Alpha-Linolenic Acid (essential fatty acids).
       - Calculate Saturated Fat limit (e.g., <10% of EER, per DGA).
     - **Fiber**:
       - Lookup AI (typically based on 14g/1000 kcal of EER).
       - Calculate Fiber AI = (EER / 1000) * 14.

   - **5. `MicronutrientServices` (or a generic `MicronutrientTargetService`)**
     - **Responsibility**: For each individual vitamin and mineral, provide the relevant RDA or AI, and the UL.
     - **Dependencies**: `DriLookupService`, `UnitConversionService`.
     - **Logic per Nutrient**:
       - Use `DriLookupService` to get RDA/AI.
       - Use `DriLookupService` to get UL.
       - For Sodium, also retrieve the Chronic Disease Risk Reduction Intake (CDRR).
       - Utilize `UnitConversionService` if inputs/outputs need conversion (e.g., Vitamin A from IU to RAE, Folate to DFE).
       - Incorporate special considerations from footnotes (e.g., increased Vitamin C for smokers – would require user input about smoking status).

   - **6. `WaterTargetService`**
     - **Responsibility**: Provide the AI for total water.
     - **Dependencies**: `DriLookupService`.
     - **Source**: Use `DriLookupService` to get AI for total water (e.g., from Table S-2 of the Water DRI report).

   - **7. `UnitConversionService`**
     - **Responsibility**: Implement conversions for various nutrient units.
     - **Examples**: Retinol Activity Equivalents (RAE), Dietary Folate Equivalents (DFE), Niacin Equivalents (NE), International Units (IU) for specific vitamins. Also, weight/height conversions (lbs/kg, in/cm).
     - **Reference**: DGA Appendix F for some conversions.

   - **8. `NutrientProfileCalculatorService` (Orchestrator Service)**
     - **Responsibility**: Acts as the main coordinator.
     - **Process**:
       - Takes all necessary user inputs (age, sex, weight, height, physical activity level, special conditions like pregnancy/lactation status).
       - Calls `EnergyRequirementService` to get EER.
       - Calls `MacronutrientServices` to get macronutrient targets.
       - Calls `MicronutrientServices` for each relevant micronutrient's RDA/AI and UL.
       - Calls `WaterTargetService`.
       - Aggregates all results and formats them into a comprehensive nutrient profile for the user.

## IV. Database Schema Considerations

   - **`Nutrients` Table**:
     - Must include all nutrients listed in the NCBI "Elements" and "Vitamins" summary tables.
     - Also needs entries for "Total Fat," "Carbohydrate," "Protein," "Fiber," "Added Sugars," "Saturated Fatty Acids," "Linoleic Acid," "Alpha-Linolenic Acid," and "Water."
     - Columns for `nutrient_name`, `default_unit`, `category` (e.g., vitamin, mineral, macronutrient).
   - **`AgeGroups` Table**:
     - Define specific age ranges (in months or years).
     - Needs to map to pregnancy-specific and lactation-specific stages (e.g., "Pregnancy 14-18y T1", "Lactation 19-30y 0-6m").
   - **`Sexes` Table**: Standard "Male", "Female".
   - **`SpecialConditions` Table**:
     - To capture states like "Pregnancy Trimester 1", "Pregnancy Trimester 2", "Pregnancy Trimester 3", "Lactation 0-6 months", "Lactation 7-12 months".
     - These would be linked to relevant `AgeGroups`.
   - **`DietaryReferenceIntakes` Table (Primary DRI storage)**:
     - Columns: `id`, `nutrient_id` (FK to `Nutrients`), `age_group_id` (FK to `AgeGroups`), `sex_id` (FK to `Sexes`), `special_condition_id` (FK to `SpecialConditions`, nullable), `dri_type` (enum: EAR, RDA, AI, UL, AMDR_min_percent, AMDR_max_percent, CDRR, EER_add_kcal_preg_T2, etc.), `value`, `unit`, `source_document_reference`.
     - Critical to correctly store `dri_type` based on source table distinctions (RDA vs. AI).
   - **(Optional) `ReferenceAnthropometry` Table (IOM Table 1-1, p.41) & `GrowthFactors` Table (IOM Table 2-1, p.53)**:
     - Would be needed if the system dynamically calculates child DRIs using extrapolation from adult values. Alternatively, pre-calculate and store all extrapolated child DRIs directly if IOM provides clear, static values or simple extrapolation logic.

## V. Important General Notes & Considerations

   - **"Criterion" Column (in DRI tables)**:
     - This column (e.g., in S-series tables) explains the basis for setting a DRI value.
     - While not directly used in calculation logic by the services (which use the final DRI values), it's excellent for documentation and understanding the science.
   - **Handling "ND" (Not Determinable) / "n/a" (Not Applicable)**:
     - Services querying DRI values must be prepared for cases where a DRI is not determinable (e.g., UL for Vitamin K for infants) or not applicable (e.g., Fiber for infants 0-6m). The system should handle this gracefully, likely by not providing a value rather than defaulting to zero or erroring.
   - **Footnotes are Paramount**:
     - Re-emphasizing: Pay EXTREME attention to all footnotes in the source DRI tables. They often contain critical, nuanced information regarding:
       - Specific units (RAE, DFE, AT).
       - Adjustments for specific populations (e.g., higher Vitamin C UL for smokers, potentially different iron needs for vegetarians – these may require additional user inputs).
       - How ULs were derived (e.g., from food only, or including supplements). This impacts advice given.
   - **Extrapolation Logic for Children**:
     - If not all DRIs for children are explicitly available in the tables for all age sub-groups, and if extrapolation from adult DRIs is required, this logic (using reference weights, growth factors, etc., as per IOM guidelines) can be complex and needs careful implementation. Seeding pre-calculated values is often simpler if feasible.

## VI. Out of Scope (For Initial DRI Calculator Implementation)

   - **USDA Dietary Patterns (DGA Appendix 3 - Tables A3-1 to A3-5)**:
     - These tables illustrate example food group amounts to meet DRIs at various calorie levels.
     - They are intended for dietary guidance and menu planning, not for calculating an individual's specific nutrient needs.
     - This could be a valuable **future enhancement** if the application expands to include meal planning or diet analysis tools.

# DRI Methodologies

## Extrapolation for Children
- EAR_child = EAR_adult × F
- Where F = (Weight_child/Weight_adult)^0.75 × (1 + growth factor)
- Growth factors are provided in Table 2-1 (p. 53)
- This is a key implementable logic for child nutrient services

## UL Extrapolation for Children
- UL_child = UL_adult × (Weight_child/Weight_adult)
- Adjusted based on relative body weights
- Values for children are generally lower than adults
- The general UL extrapolation method scales down from adult ULs, usually by body weight

## Pregnancy Needs
Considerations include:
- Obligatory fetal transfer
- Increased maternal needs
- Extrapolation by weight gain (e.g., +16 kg to non-pregnant reference weight for chromium, manganese, molybdenum)
- Specific additions are detailed for:
  - Vitamin A
  - Iron
  - Iodine
  - Copper
  - Zinc
  - Molybdenum

## Lactation Needs
- Non-pregnant requirement + increment for amount secreted in milk
- Adjusted for bioavailability/inefficiencies
- Specific additions detailed for:
  - Vitamin A
  - Iron
  - Iodine
  - Copper
  - Zinc
  - Molybdenum

## Infant AIs
### 0-6 months
- Based on average nutrient intake from human milk
- Average volume: 0.78 L/day

### 7-12 months
- Sum of nutrient from human milk (0.6 L/day) + usual intake from complementary foods
- Or extrapolation from younger infants