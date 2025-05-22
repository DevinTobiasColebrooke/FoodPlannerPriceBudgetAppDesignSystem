# Phase 4: Nutrient Calculation System

## Data Models for DRI

Ensure the following models are correctly set up with associations:
- Nutrient
- LifeStageGroup
- DriValue
- EerProfile
- EerAdditiveComponent
- ReferenceAnthropometry
- GrowthFactor
- PalDefinition

## DRI Data Seeding Service

```ruby
# lib/services/dri_data_seeder_service.rb
class DriDataSeederService
  def self.seed_all_data
    seed_nutrients
    seed_life_stage_groups
    seed_dri_values_from_dga_app1
    seed_eer_profiles
    # ... more seeding methods
  end

  private

  def self.seed_nutrients
    # Parse nutrient data and create Nutrient records
  end

  # ... more methods
end
```

## Calculator Services

### DRI Lookup Service

```ruby
# app/services/nutrient_calculator/dri_lookup_service.rb
module NutrientCalculator
  class DriLookupService
    def initialize(user_input_dto)
      @user_input = user_input_dto
      @life_stage_group = determine_life_stage_group
    end

    def get_dri(nutrient_dri_identifier, dri_type: 'RDA')
      nutrient = Nutrient.find_by(dri_identifier: nutrient_dri_identifier)
      return nil unless nutrient && @life_stage_group

      dri_value = DriValue.find_by(
        nutrient: nutrient,
        life_stage_group: @life_stage_group,
        dri_type: dri_type.to_s.upcase
      )
      format_dri_output(dri_value)
    end

    private

    def determine_life_stage_group
      age_months = @user_input.age_in_months
      sex = @user_input.sex
      LifeStageGroup.where("min_age_months <= ? AND max_age_months >= ?", age_months, age_months)
                    .where(sex: sex)
                    .first
    end

    def format_dri_output(dri_value_record)
      return nil unless dri_value_record
      { value: dri_value_record.value_numeric, unit: dri_value_record.unit }
    end
  end
end
```

## Service Implementation Details

### Core Services
- NutrientProfileCalculatorService (Orchestrator)
- BmiCalculationService
- PhysicalActivityCoefficientService
- EnergyRequirementService
- ProteinService
- Micronutrient services
- Utility services

### Input/Output Data Contracts
Define POROs or Structs for:
- UserInput
- NutrientProfile
- Other data contracts

### Integration
Create a controller or background job that uses NutrientProfileCalculatorService to:
- Calculate user's nutrient profile
- Store/display results
- Analyze recipe nutrition

For detailed service implementations, see:
- `services/nutrient_calculator.md`
- `services/dri_data_seeder.md` 