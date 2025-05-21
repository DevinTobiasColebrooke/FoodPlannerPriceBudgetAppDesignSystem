# Comprehensive DRI Calculator Backend Design

## I. Overview
   - **Purpose**: This document outlines the design considerations, backend service architecture, and methodologies for a Dietary Reference Intake (DRI) calculator.
   - **Focus**: It details the mapping of various DRI tables and data sources to specific services, data seeding strategies, the logic for calculating nutrient recommendations, and core user inputs.

## II. Core User Inputs
Collected via UI and passed to services:
1.  Age (Years, or months for infants/toddlers)
2.  Sex (Male, Female)
3.  Height (cm or inches)
4.  Weight (kg or lbs)
5.  Physical Activity Level (PAL) - (e.g., Sedentary, Low Active, Active, Very Active)
6.  Special Conditions (Flags):
    *   Pregnant (Trimester: 1st, 2nd, 3rd)
    *   Lactating (Months postpartum: 0-6, 7-12)
    *   Smoker (e.g., for Vitamin C adjustment)
    *   Vegetarian/Vegan (e.g., for Iron, B12, Zinc, Protein quality considerations)
    *   (Note: Specific medical conditions are generally out of scope for a basic calculator but acknowledged for future DRIs.)

## III. Key Data Sources and Their Roles
The following data sources are crucial for populating the DRI database and informing service logic:

   - **A. DGA Appendix 1: Nutritional Goals (Tables A1-1, A1-2, A1-3, A1-4)**
     - **Table A1-1**: Covers Infants (0-12 months) and Toddlers (12-23 months), providing Adequate Intakes (AIs) and Recommended Dietary Allowances (RDAs). Key for `DriLookupService` for these age groups. Notes "n/a" for Fiber & Total Lipid % for 6-11 month group.
     - **Table A1-2**: Addresses Children and Adults (Ages 2+). Its primary importance for the calculator lies in:
       - Macronutrient Acceptable Macronutrient Distribution Ranges (AMDRs) (e.g., Protein 10-35% kcal).
       - Fiber recommendation (14g/1000 kcal).
       - Limits for Added Sugars and Saturated Fatty Acids (e.g., <10% kcal).
       - AIs for Linoleic and Alpha-Linolenic Acid (g/day).
       - Absolute gram RDAs for Protein (e.g., 56g) and Carbohydrates (e.g., 130g) as minimums.
     - **Table A1-3**: Focuses on Pregnant Women. Crucial for macronutrient AMDRs, EFA AIs, and notes that while micronutrient RDAs/AIs often increase, the underlying macronutrient %AMDRs or g/kg recommendations might remain similar to non-pregnant, applied to a higher EER.
     - **Table A1-4**: Focuses on Lactating Women. Logic similar to A1-3 for macronutrient goals, applied to a lactation-adjusted EER.
     - **Overall Importance**: DGA Appendix 1 tables are vital for macronutrient rules, fiber, added sugar/saturated fat limits, and EFA AIs. The "Calorie Level Assessed" rows are illustrative; the underlying percentages or g/kg recommendations are key.

   - **B. DGA Appendix 2: Estimated Calorie Needs (Tables A2-1, A2-2, A2-3)**
     - **Table A2-1**: Provides direct EER values for Toddlers (12-23 months) based on age and sex, used by `EnergyRequirementService`.
     - **Table A2-2**: Underpins the EER equations for individuals aged 2 and older. `EnergyRequirementService` will implement these equations. This table can serve as a reference for testing.
     - **Table A2-3**: Provides the additional kilocalories needed during Pregnancy (Trimester 2: +340 kcal, Trimester 3: +452 kcal) and Lactation (0-6 months: +330 kcal, 7-12 months: +400 kcal) to be added to the pre-pregnancy EER by the `EnergyRequirementService`.
     - **Overall Importance**: These tables are the foundation for the `EnergyRequirementService` and subsequently for calculating gram targets for macronutrients.

   - **C. NCBI DRI Summary Tables (e.g., NBK545442 - Elements, NBK56068 - Vitamins)**
     - **"Dietary Reference Intakes: Elements" (NBK545442)**: Contains AIs, RDAs, and ULs for minerals (Sodium, Potassium, Chloride, Calcium, Phosphorus, Magnesium, Iron, Zinc, Copper, Iodine, Selenium, Manganese, Molybdenum, Fluoride, Chromium).
     - **"Dietary Reference Intakes: Vitamins" (NBK56068)**: Contains RDAs (bolded) and AIs (asterisked) for vitamins. ULs are in a separate sub-table within this source.
     - **Overall Importance**: These tables are the primary, consolidated source for seeding micronutrient EARs (if provided, though these tables mostly give RDA/AI directly), RDAs, AIs, and ULs. They effectively summarize information found in more detailed S-series tables.

   - **D. Individual DRI Study Reports (e.g., S-1 to S-10 from IOM reports)**
     - These are the original, detailed reports from the Institute of Medicine (IOM).
     - While NCBI summary tables are preferred for direct seeding of RDA/AI/UL values, these reports contain:
       - The **"Criterion" column**: Invaluable for understanding the physiological basis for a DRI (e.g., "Maximizing plasma glutathione peroxidase activity" for Selenium RDA). Useful for documentation.
       - Detailed EER formulas and coefficients.
       - Specific methodologies for extrapolation or adjustments (e.g., for pregnancy, lactation, children).
     - **Overall Importance**: Contextual understanding, detailed background, specific calculation methodologies, and criterion for DRIs.

   - **E. Footnotes (Across all DRI tables)**
     - **Critical Importance**: Footnotes often contain vital information that must be incorporated into service logic:
       - **Units**: Specific units like RAE (Retinol Activity Equivalents), DFE (Dietary Folate Equivalents), AT (Alpha-Tocopherol), NE (Niacin Equivalents).
       - **Specific Populations**: Adjustments for smokers (e.g., Vitamin C UL), vegetarians (e.g., Iron).
       - **UL Derivations**: Clarifications if ULs are from food only or include supplements.
       - **"ND" (Not Determinable)**: Especially for ULs in infants; services must handle these cases.

## IV. Database Schema Considerations
1.  **`Nutrients` Table**:
    *   Must include all nutrients listed in relevant DRI tables (e.g., NCBI "Elements" and "Vitamins" summary tables).
    *   Also needs entries for "Total Fat," "Carbohydrate," "Protein," "Fiber," "Added Sugars," "Saturated Fatty Acids," "Linoleic Acid," "Alpha-Linolenic Acid," and "Water."
    *   Columns for `nutrient_name`, `default_unit`, `category` (e.g., vitamin, mineral, macronutrient).
2.  **`AgeGroups` Table**:
    *   Define specific age ranges (in months or years).
    *   Needs to map to pregnancy-specific and lactation-specific stages (e.g., "Pregnancy 14-18y T1", "Lactation 19-30y 0-6m").
3.  **`Sexes` Table**: Standard "Male", "Female".
4.  **`SpecialConditions` Table**:
    *   To capture states like "Pregnancy Trimester 1", "Pregnancy Trimester 2", "Pregnancy Trimester 3", "Lactation 0-6 months", "Lactation 7-12 months".
    *   These would be linked to relevant `AgeGroups`.
5.  **`DietaryReferenceIntakes` Table (Primary DRI storage)**:
    *   Columns: `id`, `nutrient_id` (FK to `Nutrients`), `age_group_id` (FK to `AgeGroups`), `sex_id` (FK to `Sexes`), `special_condition_id` (FK to `SpecialConditions`, nullable), `dri_type` (enum: EAR, RDA, AI, UL, AMDR_min_percent, AMDR_max_percent, CDRR, EER_add_kcal_preg_T2, etc.), `value`, `unit`, `source_document_reference`, `criterion_notes` (optional, for basis of DRI).
    *   Critical to correctly store `dri_type` based on source table distinctions (RDA vs. AI).
6.  **(Optional) `ReferenceAnthropometry` Table & `GrowthFactors` Table**:
    *   Would be needed if the system dynamically calculates child DRIs using extrapolation from adult values (e.g., IOM Table 1-1, p.41 for reference weights; IOM Table 2-1, p.53 for growth factors). Alternatively, pre-calculate and store all extrapolated child DRIs directly if IOM provides clear, static values or simple extrapolation logic.

## V. Core Services and Responsibilities

### 1. `DriDataSeederService` (Initial Setup)
   - **Responsibility**: Meticulously parse and seed data from all specified DRI tables into the database. This is a foundational, one-time or infrequent setup step.
   - **Primary Data Sources**:
     - **Micronutrient EARs, RDAs, AIs, ULs**: Use "Dietary Reference Intakes: Elements" (NBK545442) and "Dietary Reference Intakes: Vitamins" (NBK56068) tables, and individual IOM reports.
     - **Macronutrient AMDRs, Fiber Rules, Added Sugar/Saturated Fat Limits, EFA AIs**: Use Tables A1-2, A1-3, and A1-4 from DGA Appendix 1, and IOM Macronutrient reports.
     - **EER Components**: Coefficients for EER equations, additional kcal for pregnancy/lactation (from DGA Appendix 2, Table A2-3 and IOM Energy reports).
   - **Key Tasks**:
     - Include source document/table references for each database entry.
     - Accurately distinguish between RDA (meaning an EAR was established) and AI (no EAR set) when seeding.
     - Store values in their base units, noting any specific forms (e.g., alpha-tocopherol for Vitamin E).

### 2. `NutrientProfileCalculatorService` (Main Orchestrator Service)
   - **Input**: User attributes (age, sex, weight, height, PAL, special conditions).
   - **Responsibility**:
     - Acts as the main coordinator.
     - Calls `EnergyRequirementService` to get EER.
     - Calls individual macronutrient, micronutrient, and water services to get their respective recommendations and limits.
     - Aggregates all results and formats them into a comprehensive nutrient profile for the user.
   - **Output**: A hash/object containing all calculated nutrient needs (EER, RDAs/AIs, ULs for all relevant nutrients, AMDRs, etc.).
   - **Dependencies**: All major calculation and lookup services.

### 3. `EnergyRequirementService` (EER Service)
   - **Input**: Age, sex, weight, height, PAL, pregnancy/lactation status.
   - **Responsibility**: Calculate the user's specific Estimated Energy Requirement (EER) based on established EER formulas (e.g., from IOM).
   - **Calculation Logic**:
     - **Toddlers (12-23 months)**: Directly look up EER values from DGA Table A2-1 if applicable, or use IOM formulas.
     - **Ages 2+**: Implement the EER equations provided in IOM reports (as referenced by DGA Table A2-2).
     - **Pregnancy**: Add additional kcal to the pre-pregnancy EER: Trimester 1 EER is same as pre-pregnancy, +340 kcal for Trimester 2, +452 kcal for Trimester 3 (from DGA Table A2-3 or IOM reports).
     - **Lactation**: Add additional kcal to the pre-pregnancy EER: +330 kcal for 0-6 months, +400 kcal for 7-12 months (from DGA Table A2-3 or IOM reports). These figures account for milk production cost minus energy mobilization.
   - **Output**: Estimated Energy Requirement (kcal/day).
   - **Dependencies**: `PhysicalActivityCoefficientService` (to convert descriptive activity levels to PAL coefficients used in EER equations).

### 4. `PhysicalActivityCoefficientService`
   - **Input**: Description of daily activities or selection from predefined categories (e.g., "Sedentary", "Low Active", "Active", "Very Active").
   - **Responsibility**: Map user-inputted descriptive physical activity levels to the specific Physical Activity Level (PAL) coefficients required by the EER equations (e.g., from DGA Table A2-2 or IOM Energy reports).
   - **Output**: PAL value (e.g., 1.0, 1.11, 1.25, 1.48 for males 19+).

### 5. Macronutrient Services
   Consolidated under individual services or a single `MacronutrientTargetService`.
   - **Common Inputs**: Age, sex, EER (from `EnergyRequirementService`), weight (for protein), special conditions.
   - **Common Dependencies**: `DriLookupService`, `EnergyRequirementService`.

   - **`CarbohydrateService`**
     - **Responsibility**:
       - Calculate RDA/AI for carbohydrates (g/day).
       - Calculate AMDR for carbohydrates (% of EER, and converted to g/day).
       - Recommend limits for Added Sugars (% of EER, converted to g/day, from DGA).
     - **Output**: `{ rda_g: X, ai_g: Y, amdr_percent_min: A, amdr_percent_max: B, amdr_g_min: C, amdr_g_max: D, added_sugars_limit_percent: E, added_sugars_limit_g: F }`

   - **`ProteinService`**
     - **Responsibility**:
       - Calculate RDA/AI for protein (g/day and g/kg body weight).
       - Calculate AMDR for protein (% of EER, and converted to g/day).
     - **Output**: `{ rda_g_per_kg: X, rda_g_total: Y, ai_g_total: Z, amdr_percent_min: A, amdr_percent_max: B, amdr_g_min: C, amdr_g_max: D }`

   - **`FatService` (Total Fat and Fatty Acids)**
     - **Responsibility**:
       - Calculate AI for Total Fat (infants).
       - Calculate AMDR for Total Fat (% of EER, and g/day).
       - Calculate AIs for Essential Fatty Acids (Linoleic, Alpha-Linolenic) (g/day).
       - Recommend limits for Saturated Fat (% of EER, from DGA, converted to g/day). (Trans Fat limit is "as low as possible").
     - **Output**: `{ total_fat_ai_g: (infants), total_fat_amdr_percent_min: A, ..., linoleic_ai_g: X, ala_ai_g: Y, saturated_fat_limit_percent: Z, saturated_fat_limit_g: W, ... }`

   - **`FiberService`**
     - **Responsibility**: Calculate AI for Total Fiber (g/day), often based on g/1000 kcal of EER.
     - **Output**: `{ ai_g: X }`

### 6. `WaterService` (or `WaterTargetService`)
   - **Input**: Age, sex, potentially PAL and environmental factors (if advanced).
   - **Responsibility**: Provide the AI for total water (L/day, including from beverages and food).
   - **Output**: `{ ai_liters: X }`
   - **Dependencies**: `DriLookupService`.

### 7. Micronutrient Services (Vitamins and Minerals)
   - **Approach**: Either individual services (e.g., `VitaminDService`, `CalciumService`) or a generic `MicronutrientTargetService`.
   - **Common Input**: Age, sex, special conditions (e.g., pregnancy, lactation, smoking status for Vitamin C, vegetarian status for iron/zinc).
   - **Common Responsibility**: Determine RDA or AI, and the UL for each micronutrient. For Sodium, also retrieve the Chronic Disease Risk Reduction Intake (CDRR).
   - **Common Output Structure**: `{ rda: X, ai: Y, ul: Z, cdrr: W (for sodium) }` (units specific to nutrient, e.g., µg, mg, IU).
   - **Common Dependencies**: `DriLookupService`, potentially `UnitConversionService`.
   - **Special Considerations**:
     - **Vitamin A**: RAE conversions. UL applies to preformed Vitamin A.
     - **Niacin**: NE conversions from tryptophan (if assessing intake). Requirements are in NE.
     - **Folate**: DFE conversions (if assessing intake). Requirements are in DFE. UL applies to synthetic folic acid.
     - **Vitamin E**: Alpha-tocopherol forms are primary.
     - **Vitamin C**: Increased need for smokers.
     - **Sodium**: Has CDRR.
     - **Iron**: Different bioavailability assumptions for vegetarians may require adjustment.
     - **Magnesium**: UL applies to supplemental magnesium.

### 8. Utility / Helper Services (Runtime)

   - **`DriLookupService`**
     - **Overview**: Core runtime service for fetching DRI values (EAR, RDA, AI, UL, AMDR percentages, CDRR, EER coefficients, etc.) from the database.
     - **Responsibilities**:
       - Retrieve specific DRI values based on nutrient, user's profile (age, sex, special conditions).
       - Query DRI database tables, handling age group bracketing and special condition lookups.
     - **Core Logic**:
       - Implement robust logic to find the correct `AgeGroup` given age (in months or years) and any special conditions.
       - Query the `DietaryReferenceIntakes` table using the determined `AgeGroup`, sex, and nutrient.
     - **Data Sources (examples of what it fetches from DB)**:
       - DGA Appendix 1 derived values (macronutrient AMDRs, EFA AIs, etc.).
       - NCBI DRI Summary Tables derived values (micronutrient RDAs/AIs/ULs).
       - Water DRI Report derived values (AI for total water).
     - **Input**: Nutrient name/identifier, age, sex, special conditions.
     - **Output**: The specific DRI value(s) requested (e.g., `{ "ear": number | null, "ai": number | null, "rda": number | null, "ul": number | "ND", "unit": string }`).
     - **Dependencies**: Database access.

   - **`UnitConversionService`**
     - **Overview**: Handles unit conversions.
     - **Responsibility**: Convert between units (e.g., lbs to kg, inches to cm, IU to µg for certain vitamins like Vitamin D: 1 µg cholecalciferol = 40 IU). Implement conversions for RAE, DFE, NE if user input is in constituent forms rather than the DRI summary unit.
     - **Reference**: DGA Appendix F for some conversions, specific DRI reports for others.
     - **Input**: Value, from_unit, to_unit.
     - **Output**: Converted value.

## VI. DRI Calculation Principles and Methodologies

### A. General Principles (Lookup vs. Calculation)
The determination of a DRI value can be a direct lookup from pre-established tables or involve calculations based on other parameters or DRI values.

   - **Primarily Lookup-Based Values (via `DriLookupService`)**:
     - **EAR (Estimated Average Requirement)**: Direct value from DRI tables.
     - **AI (Adequate Intake)**: Direct value when EAR/RDA cannot be set.
     - **UL (Tolerable Upper Intake Level)**: Direct value.
     - **CDRR (Chronic Disease Risk Reduction Intake)**: For Sodium, a direct lookup.
     - **AMDR Percentages (Macronutrients)**: e.g., Protein 10-35% of energy.
     - **Fixed Recommendations**: e.g., 130g/day RDA for carbohydrates for adults; AIs for essential fatty acids (g/day); AI for Water (L/day).
     - **Pre-calculated RDAs**: If RDAs are stored directly in the database (as often found in summary tables).

   - **Primarily Calculation-Based Values (by specific services)**:
     - **RDA (Recommended Dietary Allowance)**:
       - Often calculated from EAR: `RDA = EAR + 2 * SD_EAR` (if Standard Deviation of EAR is known).
       - Or `RDA = EAR * 1.2` (if SD is unknown, assumes a Coefficient of Variation (CV) of 10%).
       - This calculation can be done by the nutrient-specific service or by `DriLookupService` if EAR is stored.
     - **EER (Estimated Energy Requirement)**: Calculated by `EnergyRequirementService` using complex formulas based on age, sex, weight, height, PAL, and life stage (coefficients are looked up).
     - **Macronutrient Grams from AMDRs**: Calculated by respective macronutrient services using EER.
       - Min/Max grams = `(AMDR % / 100) * EER (kcal) / (kcal_per_gram_of_macronutrient)`. (4 kcal/g for protein/carbs, 9 kcal/g for fat).
     - **Protein (Total g/day from g/kg RDA/AI)**: Calculated by `ProteinService`.
       - Total grams = `(g/kg/day RDA/AI) * user's body weight (kg)`.
     - **Fiber (Total g/day from g/1000 kcal AI)**: Calculated by `FiberService` if AI is in this form.
       - Total grams = `(AI in g/1000 kcal) * (EER / 1000)`.
     - **Specific Limits in Grams (e.g., Added Sugars, Saturated Fat)**: Calculated from % EER recommendations and EER.
       - Gram limit = `(Limit % / 100) * EER (kcal) / (kcal_per_gram)`.
     - **Unit Conversions**: By `UnitConversionService` (e.g., Vitamin D IU to µg, or RAE/DFE/NE if assessing intake from components).
     - **Adjustments for Specific Conditions**: E.g., adding 35mg Vitamin C for smokers to the base RDA.

### B. Specific Methodologies

   - **Extrapolation for Children (if not directly tabulated)**:
     - EAR_child = EAR_adult × F
     - Where F = (Weight_child_ref / Weight_adult_ref)^0.75 × (1 + growth factor)
     - Growth factors are provided in IOM tables (e.g., Table 2-1, p. 53 of DRI overview). Reference weights also from IOM tables.
     - This is a key implementable logic if direct child DRIs are unavailable for a specific age subgroup.
   - **UL Extrapolation for Children (if not directly tabulated)**:
     - UL_child = UL_adult × (Weight_child_ref / Weight_adult_ref)
     - Adjusted based on relative body weights. This method scales down from adult ULs.
     - (Note: For many nutrients, child ULs are directly provided in DRI tables).
   - **Pregnancy Needs Adjustments**:
     - May involve adding fixed amounts to non-pregnant DRIs (e.g., Magnesium EAR).
     - For some nutrients, extrapolation based on weight gain (e.g., +16 kg to non-pregnant reference weight for chromium, manganese, molybdenum during pregnancy) is used by DRI committees to set values.
     - Specific additions/increases for RDA/AI are detailed in DRI tables for nutrients like Vitamin A, Iron, Iodine, Copper, Zinc, Molybdenum, Folate. `DriLookupService` would fetch these specific pregnancy values.
   - **Lactation Needs Adjustments**:
     - Typically: Non-pregnant requirement + increment for amount secreted in milk, adjusted for bioavailability/inefficiencies.
     - Specific additions/increases for RDA/AI are detailed in DRI tables for nutrients like Vitamin A, Iron, Iodine, Copper, Zinc, Molybdenum, Folate. `DriLookupService` would fetch these specific lactation values.
   - **Infant AIs**:
     - **0-6 months**: Generally based on average nutrient intake from human milk (assuming average volume of 0.78 L/day). These are direct lookups.
     - **7-12 months**: Sum of nutrient from human milk (e.g., 0.6 L/day) + usual intake from complementary foods, OR extrapolation from younger infants or other age groups. These are typically direct lookups from DRI tables.

## VII. Important General Considerations

   - **"Criterion" Column (in DRI tables)**:
     - This column (e.g., in S-series tables from IOM reports) explains the physiological basis for setting a DRI value.
     - While not directly used in calculation logic by the services (which use the final DRI values), it's excellent for documentation, understanding the science, and providing context to users if desired. Store this in the database if possible.
   - **Handling "ND" (Not Determinable) / "n/a" (Not Applicable)**:
     - Services querying DRI values must be prepared for cases where a DRI is not determinable (e.g., UL for Vitamin K for infants) or not applicable (e.g., Fiber for infants 0-6m).
     - The system should handle this gracefully, likely by not providing a value or providing a specific indicator ("ND", "n/a") rather than defaulting to zero or erroring.
   - **Footnotes are Paramount**:
     - Pay EXTREME attention to all footnotes in the source DRI tables. They often contain critical, nuanced information regarding:
       - Specific units (RAE, DFE, NE, AT).
       - Adjustments for specific populations (e.g., higher Vitamin C UL for smokers, potentially different iron needs for vegetarians – these may require additional user inputs).
       - How ULs were derived (e.g., from food only, or including supplements, like for magnesium or folate). This impacts advice given.
       - Clarifications on the form of the nutrient (e.g., UL for folate applies to folic acid from fortified foods/supplements).
   - **Distinguishing RDA vs. AI**:
     - Ensure the system correctly distinguishes between RDA (Estimated Average Requirement (EAR) was established) and AI (EAR not established). This has implications for assessing population-level adequacy vs. individual goals.
   - **DRI Definitions**:
     - **EAR (Estimated Average Requirement)**: Intake at which the risk of inadequacy is 0.5 (50%) for an individual in a specific life stage and gender group.
     - **RDA (Recommended Dietary Allowance)**: Average daily dietary intake level sufficient to meet the nutrient requirement of nearly all (97-98%) healthy individuals in a particular life stage and gender group. `RDA = EAR + 2 SD_EAR`.
     - **AI (Adequate Intake)**: Recommended average daily intake level based on observed or experimentally determined approximations or estimates of nutrient intake by a group (or groups) of apparently healthy people that are assumed to be adequate. Used when an RDA cannot be determined.
     - **UL (Tolerable Upper Intake Level)**: Highest average daily nutrient intake level likely to pose no risk of adverse health effects to almost all individuals in the general population.
     - **CDRR (Chronic Disease Risk Reduction Intake)**: Nutrient intake level expected to reduce chronic disease risk in an otherwise healthy population.

## VIII. Expected Outputs
From the `NutrientProfileCalculatorService`, a comprehensive hash or object containing:
*   Estimated Energy Requirement (EER) in kcal.
*   For each Macronutrient (Protein, Carbohydrates, Fat):
    *   RDA or AI (in grams, and g/kg for protein).
    *   AMDR (Acceptable Macronutrient Distribution Range) as a percentage of EER and corresponding gram range.
    *   Specific limits (e.g., added sugars limit in %EER and grams, saturated fat limit in %EER and grams).
    *   AIs for Essential Fatty Acids (g/day).
*   For Fiber: AI (in grams).
*   For Water: AI (in Liters).
*   For each Vitamin: RDA or AI (in appropriate units, e.g., mg, µg, RAE, DFE, NE), and UL (if established).
*   For each Mineral: RDA or AI (in appropriate units, e.g., mg, µg), UL (if established), and CDRR (for Sodium).
*   Units for all values.
*   Indication of "ND" or "n/a" where applicable.

## IX. Example Data Flow (Simplified for Carbohydrates)
1.  `NutrientProfileCalculatorService` receives user inputs (age, sex, weight, height, PAL, etc.).
2.  Calls `EnergyRequirementService` with (age, sex, wt, ht, PAL) -> returns EER (e.g., 2000 kcal).
3.  Calls `CarbohydrateService` with (age, sex, EER=2000 kcal).
    *   `CarbohydrateService` calls `DriLookupService` for Carb RDA based on (age, sex) -> returns, e.g., 130g RDA.
    *   `CarbohydrateService` calls `DriLookupService` for Carb AMDR % -> returns, e.g., 45-65%.
    *   `CarbohydrateService` calculates AMDR grams: Min `(0.45 * 2000 / 4)` to Max `(0.65 * 2000 / 4)`.
    *   `CarbohydrateService` calls `DriLookupService` for Added Sugar limit % (e.g., from DGA) -> returns <10%.
    *   `CarbohydrateService` calculates Added Sugar limit grams: `(0.10 * 2000 / 4)`.
    *   `CarbohydrateService` returns its hash of results (RDA, AMDR %, AMDR g, Added Sugar limit %, Added Sugar limit g).
4.  `NutrientProfileCalculatorService` aggregates this with results from other nutrient services (Protein, Fat, Vitamins, Minerals, etc.).
5.  `NutrientProfileCalculatorService` returns the complete nutrient profile to the caller.

## X. Out of Scope / Next Steps

### Out of Scope (For Initial DRI Calculator Implementation)
   - **USDA Dietary Patterns (DGA Appendix 3 - Tables A3-1 to A3-5)**:
     - These tables illustrate example food group amounts to meet DRIs at various calorie levels.
     - They are intended for dietary guidance and menu planning, not for calculating an individual's specific nutrient needs based on their profile.
     - This could be a valuable **future enhancement** if the application expands to include meal planning or diet analysis tools.
   - **Food Composition Database Integration & Intake Analysis**: Analyzing user's dietary intake against the calculated DRIs.
   - **Detailed Medical Condition Adjustments**: Beyond general states like pregnancy/lactation.

### Next Steps
1.  **Finalize Database Schema**: Refine the tables and columns for `Nutrients`, `AgeGroups`, `Sexes`, `SpecialConditions`, and `DietaryReferenceIntakes` based on the exact structure of chosen source DRI tables.
2.  **Implement `DriDataSeederService`**: Develop scripts to parse source DRI tables (CSVs, PDFs, APIs if available) and populate the database accurately. This is a critical and meticulous task.
3.  **Develop Core Services Incrementally**:
    *   Start with `EnergyRequirementService` as EER is fundamental.
    *   Implement `DriLookupService` for basic data retrieval.
    *   Develop macronutrient services.
    *   Gradually add micronutrient services.
4.  **Unit Testing**: Rigorously test each service, especially `EnergyRequirementService` and calculations within other services, against reference values from DRI tables or examples.
5.  **Integration Testing**: Test the `NutrientProfileCalculatorService` and the overall data flow.
6.  **Documentation**: Document sources for each DRI value, calculation logic, and assumptions made.