# DRI Data Seeder Service Implementation

## Service Definition

```ruby
# lib/services/dri_data_seeder_service.rb
require 'csv'

class DriDataSeederService
  def self.seed_all_data
    seed_nutrients
    seed_life_stage_groups
    seed_dri_values_from_dga_app1
    seed_eer_profiles
    seed_eer_additive_components
    seed_reference_anthropometries
    seed_growth_factors
    seed_pal_definitions
    seed_food_groups_and_subgroups
    seed_dietary_patterns
    seed_dietary_pattern_components
  end

  private

  def self.seed_nutrients
    nutrients_data = load_nutrients_data
    nutrients_data.each do |data|
      Nutrient.find_or_create_by!(dri_identifier: data[:dri_identifier]) do |nutrient|
        nutrient.name = data[:name]
        nutrient.nutrient_category = data[:category]
        nutrient.default_dri_unit = data[:default_unit]
        nutrient.recipe_analysis_unit = data[:analysis_unit]
        nutrient.conversion_factor_to_dri_unit = data[:conversion_factor]
        nutrient.description = data[:description]
        nutrient.sort_order = data[:sort_order]
      end
    end
  end

  def self.seed_life_stage_groups
    life_stages_data = load_life_stages_data
    life_stages_data.each do |data|
      LifeStageGroup.find_or_create_by!(name: data[:name]) do |group|
        group.min_age_months = data[:min_age_months]
        group.max_age_months = data[:max_age_months]
        group.sex = data[:sex]
        group.special_condition = data[:special_condition]
        group.trimester = data[:trimester]
        group.lactation_period = data[:lactation_period]
        group.notes = data[:notes]
      end
    end
  end

  def self.seed_dri_values_from_dga_app1
    dri_values_data = load_dri_values_data
    dri_values_data.each do |data|
      nutrient = Nutrient.find_by!(dri_identifier: data[:nutrient_identifier])
      life_stage = LifeStageGroup.find_by!(name: data[:life_stage_name])

      DriValue.find_or_create_by!(
        nutrient: nutrient,
        life_stage_group: life_stage,
        dri_type: data[:dri_type]
      ) do |dri_value|
        dri_value.value_numeric = data[:value_numeric]
        dri_value.value_string = data[:value_string]
        dri_value.unit = data[:unit]
        dri_value.source_of_goal = data[:source_of_goal]
        dri_value.criterion = data[:criterion]
        dri_value.footnote_marker = data[:footnote_marker]
        dri_value.notes = data[:notes]
        dri_value.source_document_reference = data[:source_document]
      end
    end
  end

  def self.seed_eer_profiles
    eer_profiles_data = load_eer_profiles_data
    eer_profiles_data.each do |data|
      life_stage = data[:life_stage_name] ? LifeStageGroup.find_by(name: data[:life_stage_name]) : nil

      EerProfile.find_or_create_by!(name: data[:name]) do |profile|
        profile.source_table_reference = data[:source_table]
        profile.life_stage_group = life_stage
        profile.sex_filter = data[:sex_filter]
        profile.age_min_months_filter = data[:age_min_months]
        profile.age_max_months_filter = data[:age_max_months]
        profile.pal_category_applicable = data[:pal_category]
        profile.coefficient_intercept = data[:coefficient_intercept]
        profile.coefficient_age_years = data[:coefficient_age_years]
        profile.coefficient_age_months = data[:coefficient_age_months]
        profile.coefficient_height_cm = data[:coefficient_height_cm]
        profile.coefficient_weight_kg = data[:coefficient_weight_kg]
        profile.coefficient_pal_value = data[:coefficient_pal_value]
        profile.coefficient_gestation_weeks = data[:coefficient_gestation_weeks]
        profile.equation_basis = data[:equation_basis]
        profile.standard_error_of_predicted_value_kcal = data[:standard_error]
        profile.notes = data[:notes]
      end
    end
  end

  def self.seed_eer_additive_components
    components_data = load_eer_components_data
    components_data.each do |data|
      life_stage = data[:life_stage_name] ? LifeStageGroup.find_by(name: data[:life_stage_name]) : nil

      EerAdditiveComponent.find_or_create_by!(
        component_type: data[:component_type],
        life_stage_group: life_stage,
        sex_filter: data[:sex_filter],
        age_min_months_filter: data[:age_min_months],
        age_max_months_filter: data[:age_max_months],
        condition_pregnancy_trimester_filter: data[:pregnancy_trimester],
        condition_pre_pregnancy_bmi_category_filter: data[:pre_pregnancy_bmi],
        condition_lactation_period_filter: data[:lactation_period]
      ) do |component|
        component.value_kcal_day = data[:value_kcal_day]
        component.notes = data[:notes]
        component.source_document_reference = data[:source_document_reference]
      end
    end
  end

  def self.seed_reference_anthropometries
    reference_data = load_reference_anthropometries_data
    reference_data.each do |data|
      life_stage = LifeStageGroup.find_by!(name: data[:life_stage_name])

      ReferenceAnthropometry.find_or_create_by!(life_stage_group: life_stage) do |ref|
        ref.reference_height_cm = data[:reference_height_cm]
        ref.reference_weight_kg = data[:reference_weight_kg]
        ref.median_bmi = data[:median_bmi]
        ref.source_document_reference = data[:source_document]
      end
    end
  end

  def self.seed_growth_factors
    growth_data = load_growth_factors_data
    growth_data.each do |data|
      life_stage = LifeStageGroup.find_by!(name: data[:life_stage_name])

      GrowthFactor.find_or_create_by!(
        life_stage_group: life_stage,
        factor_type: data[:factor_type]
      ) do |factor|
        factor.factor_value = data[:factor_value]
        factor.source_document_reference = data[:source_document]
      end
    end
  end

  def self.seed_pal_definitions
    pal_data = load_pal_definitions_data
    pal_data.each do |data|
      life_stage = LifeStageGroup.find_by!(name: data[:life_stage_name])

      PalDefinition.find_or_create_by!(
        life_stage_group: life_stage,
        pal_category: data[:pal_category]
      ) do |pal|
        pal.pal_range_min_value = data[:pal_range_min]
        pal.pal_range_max_value = data[:pal_range_max]
        pal.percentile_value = data[:percentile]
        pal.pal_value_at_percentile = data[:pal_value_at_percentile]
        pal.coefficient_for_eer_equation = data[:coefficient]
        pal.source_document_reference = data[:source_document]
      end
    end
  end
  
  def self.seed_food_groups_and_subgroups
    food_groups_data = load_food_groups_data
    food_groups_data.each do |data|
      FoodGroup.find_or_create_by!(name: data[:name]) do |fg|
        fg.default_unit_name = data[:default_unit_name]
      end
    end

    food_subgroups_data = load_food_subgroups_data
    food_subgroups_data.each do |data|
      food_group = FoodGroup.find_by!(name: data[:food_group_name])
      FoodSubgroup.find_or_create_by!(food_group: food_group, name: data[:name])
    end
  end

  def self.seed_dietary_patterns
    patterns_data = load_dietary_patterns_data
    patterns_data.each do |data|
      DietaryPattern.find_or_create_by!(name: data[:name]) do |pattern|
        pattern.description = data[:description]
        pattern.source_document_reference = data[:source_document_reference]
      end
    end
  end

  def self.seed_dietary_pattern_components
    components_data = load_dietary_pattern_components_data
    components_data.each do |data|
      pattern = DietaryPattern.find_by!(name: data[:dietary_pattern_name])
      food_group = data[:food_group_name].present? ? FoodGroup.find_by(name: data[:food_group_name]) : nil
      food_subgroup = data[:food_subgroup_name].present? ? FoodSubgroup.find_by(name: data[:food_subgroup_name]) : nil

      DietaryPatternComponent.find_or_create_by!(
        dietary_pattern: pattern,
        calorie_level: data[:calorie_level],
        food_group: food_group,
        food_subgroup: food_subgroup,
        component_name: data[:component_name]
      ) do |component|
        component.amount_value = data[:amount_value]
        component.amount_unit = data[:amount_unit]
        component.frequency = data[:frequency]
        component.notes = data[:notes]
      end
    end
  end

  # Data loading methods
  def self.load_data_from_csv(file_name, &block)
    file_path = Rails.root.join('db', 'data_sources', file_name)
    return [] unless File.exist?(file_path)
    CSV.read(file_path, headers: true, header_converters: :symbol, skip_blanks: true).map do |row|
      cleaned_row = row.to_h.transform_keys { |k| k.to_s.strip.to_sym }.transform_values { |v| v.is_a?(String) ? v.strip : v }
      block.call(cleaned_row)
    end.compact
  end

  def self.load_nutrients_data
    load_data_from_csv('nutrients.csv') do |row|
      {
        dri_identifier: row[:dri_identifier],
        name: row[:name],
        category: row[:category],
        default_unit: row[:default_unit],
        analysis_unit: row[:analysis_unit],
        conversion_factor: row[:conversion_factor]&.to_f,
        description: row[:description],
        sort_order: row[:sort_order]&.to_i
      }
    end
  end

  def self.load_life_stages_data
    load_data_from_csv('life_stages.csv') do |row|
      {
        name: row[:name],
        min_age_months: row[:min_age_months]&.to_i,
        max_age_months: row[:max_age_months]&.to_i,
        sex: row[:sex],
        special_condition: row[:special_condition],
        trimester: row[:trimester]&.to_i,
        lactation_period: row[:lactation_period],
        notes: row[:notes]
      }
    end
  end

  def self.load_dri_values_data
    load_data_from_csv('dri_values.csv') do |row|
      {
        nutrient_identifier: row[:nutrient_identifier],
        life_stage_name: row[:life_stage_name],
        dri_type: row[:dri_type],
        value_numeric: row[:value_numeric]&.to_f,
        value_string: row[:value_string],
        unit: row[:unit],
        source_of_goal: row[:source_of_goal],
        criterion: row[:criterion],
        footnote_marker: row[:footnote_marker],
        notes: row[:notes],
        source_document: row[:source_document]
      }
    end
  end

  def self.load_eer_profiles_data
    load_data_from_csv('eer_profiles.csv') do |row|
      {
        name: row[:name],
        source_table: row[:source_table_reference],
        life_stage_name: row[:life_stage_group_id_placeholder],
        sex_filter: row[:sex_filter],
        age_min_months: row[:age_min_months_filter]&.to_i,
        age_max_months: row[:age_max_months_filter]&.to_i,
        pal_category: row[:pal_category_applicable],
        coefficient_intercept: row[:coefficient_intercept]&.to_f,
        coefficient_age_years: row[:coefficient_age_years]&.to_f,
        coefficient_age_months: row[:coefficient_age_months]&.to_f,
        coefficient_height_cm: row[:coefficient_height_cm]&.to_f,
        coefficient_weight_kg: row[:coefficient_weight_kg]&.to_f,
        coefficient_pal_value: row[:coefficient_pal_value]&.to_f,
        coefficient_gestation_weeks: row[:coefficient_gestation_weeks]&.to_f,
        equation_basis: row[:equation_basis],
        standard_error: row[:standard_error_of_predicted_value_kcal]&.to_f,
        notes: row[:notes]
      }
    end
  end

  def self.load_eer_components_data
    load_data_from_csv('eer_additive_components.csv') do |row|
      {
        component_type: row[:component_type],
        life_stage_name: row[:life_stage_group_id_placeholder],
        sex_filter: row[:sex_filter],
        age_min_months: row[:age_min_months_filter]&.to_i,
        age_max_months: row[:age_max_months_filter]&.to_i,
        pregnancy_trimester: row[:condition_pregnancy_trimester_filter]&.to_i,
        pre_pregnancy_bmi: row[:condition_pre_pregnancy_bmi_category_filter],
        lactation_period: row[:condition_lactation_period_filter],
        value_kcal_day: row[:value_kcal_day]&.to_f,
        notes: row[:notes],
        source_document_reference: row[:source_document_reference]
      }
    end
  end

  def self.load_reference_anthropometries_data
    load_data_from_csv('reference_anthropometries.csv') do |row|
      {
        life_stage_name: row[:life_stage_group_id_placeholder],
        reference_height_cm: row[:reference_height_cm]&.to_f,
        reference_weight_kg: row[:reference_weight_kg]&.to_f,
        median_bmi: row[:median_bmi]&.to_f,
        source_document: row[:source_document_reference]
      }
    end
  end

  def self.load_growth_factors_data
    load_data_from_csv('growth_factors.csv') do |row|
      {
        life_stage_name: row[:life_stage_group_id_placeholder],
        factor_value: row[:factor_value]&.to_f,
        factor_type: row[:factor_type],
        source_document: row[:source_document_reference]
      }
    end
  end

  def self.load_pal_definitions_data
    load_data_from_csv('pal_definitions.csv') do |row|
      {
        life_stage_name: row[:life_stage_name],
        pal_category: row[:pal_category],
        pal_range_min: row[:pal_range_min_value]&.to_f,
        pal_range_max: row[:pal_range_max_value]&.to_f,
        percentile: row[:percentile_value]&.to_f,
        pal_value_at_percentile: row[:pal_value_at_percentile]&.to_f,
        coefficient: row[:coefficient_for_eer_equation]&.to_f,
        source_document: row[:source_document_reference]
      }
    end
  end
  
  def self.load_food_groups_data
    load_data_from_csv('food_groups.csv') do |row|
      { name: row[:name], default_unit_name: row[:default_unit_name] }
    end
  end

  def self.load_food_subgroups_data
    load_data_from_csv('food_subgroups.csv') do |row|
      { food_group_name: row[:food_group_name], name: row[:name] }
    end
  end

  def self.load_dietary_patterns_data
    load_data_from_csv('dietary_patterns.csv') do |row|
      { 
        name: row[:name], 
        description: row[:description],
        source_document_reference: row[:source_document_reference]
      }
    end
  end

  def self.load_dietary_pattern_components_data
    load_data_from_csv('dietary_pattern_components.csv') do |row|
      {
        dietary_pattern_name: row[:dietary_pattern_name],
        calorie_level: row[:calorie_level]&.to_i,
        food_group_name: row[:food_group_name],
        food_subgroup_name: row[:food_subgroup_name],
        component_name: row[:component_name],
        amount_value: row[:amount_value],
        amount_unit: row[:amount_unit],
        frequency: row[:frequency],
        notes: row[:notes]
      }
    end
  end
end
```

## Usage

### Seeding All Data
```ruby
# In db/seeds.rb or a rake task
DriDataSeederService.seed_all_data
```

### Seeding Individual Components
```ruby
# Seed just nutrients
# DriDataSeederService.seed_nutrients

# Seed just life stage groups
# DriDataSeederService.seed_life_stage_groups

# ... (similar for other individual seed methods)
```

## Data Sources

The service expects data files in the following format:

### Nutrients (nutrients.csv)
```csv
dri_identifier,name,category,default_unit,analysis_unit,conversion_factor,description,sort_order
```

### Life Stage Groups (life_stages.csv)
```csv
name,min_age_months,max_age_months,sex,special_condition,trimester,lactation_period,notes
```

### DRI Values (dri_values.csv)
```csv
nutrient_identifier,life_stage_name,dri_type,value_numeric,value_string,unit,source_of_goal,criterion,footnote_marker,notes,source_document
```

### EER Profiles (eer_profiles.csv)
```csv
name,source_table_reference,life_stage_group_id_placeholder,sex_filter,age_min_months_filter,age_max_months_filter,pal_category_applicable,coefficient_intercept,coefficient_age_years,coefficient_age_months,coefficient_height_cm,coefficient_weight_kg,coefficient_pal_value,coefficient_gestation_weeks,equation_basis,standard_error_of_predicted_value_kcal,notes
```

### EER Additive Components (eer_additive_components.csv)
```csv
component_type,life_stage_group_id_placeholder,sex_filter,age_min_months_filter,age_max_months_filter,condition_pregnancy_trimester_filter,condition_pre_pregnancy_bmi_category_filter,condition_lactation_period_filter,value_kcal_day,notes,source_document_reference
```

### Reference Anthropometries (reference_anthropometries.csv)
```csv
life_stage_group_id_placeholder,reference_height_cm,reference_weight_kg,median_bmi,source_document_reference
```

### Growth Factors (growth_factors.csv)
```csv
life_stage_group_id_placeholder,factor_value,factor_type,source_document_reference
```

### PAL Definitions (pal_definitions.csv)
```csv
life_stage_name,pal_category,pal_range_min_value,pal_range_max_value,percentile_value,pal_value_at_percentile,coefficient_for_eer_equation,source_document_reference
```

### Food Groups (food_groups.csv)
```csv
name,default_unit_name
```

### Food Subgroups (food_subgroups.csv)
```csv
food_group_name,name
```

### Dietary Patterns (dietary_patterns.csv)
```csv
name,description,source_document_reference
```

### Dietary Pattern Components (dietary_pattern_components.csv)
```csv
dietary_pattern_name,calorie_level,food_group_name,food_subgroup_name,component_name,amount_value,amount_unit,frequency,notes
```

## Data Source References

The data should be sourced from:
- Dietary Guidelines for Americans (DGA) - Appendices 1, 2, 3, and others as noted
- National Academies of Sciences, Engineering, and Medicine (NASEM) - DRI Reports
- Institute of Medicine (IOM) reports (now NASEM)
- Food and Nutrition Board (FNB) publications

## Error Handling

The service includes basic error handling:

```ruby
def self.seed_with_error_handling
  ActiveRecord::Base.transaction do
    seed_all_data
  end
rescue StandardError => e
  Rails.logger.error "Error seeding DRI data: #{e.message}"
  Rails.logger.error e.backtrace.join("\n")
  raise e
end
```

## Validation

Before seeding, validate the data:

```ruby
def self.validate_data
  validate_nutrients_data
  validate_life_stages_data
  # ... validate other data
end

def self.validate_nutrients_data
  data = load_nutrients_data
  data.each do |row|
    raise "Invalid nutrient data: #{row}" unless valid_nutrient_data?(row)
  end
end

def self.valid_nutrient_data?(row)
  row[:dri_identifier].present? &&
    row[:name].present? &&
    row[:category].present? &&
    row[:default_unit].present? &&
    row[:analysis_unit].present?
end
```