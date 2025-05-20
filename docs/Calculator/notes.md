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

