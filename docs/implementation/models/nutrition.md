# Nutrition Model Implementation

## Core Models

### Nutrient
```ruby
# app/models/nutrient.rb
class Nutrient < ApplicationRecord
  has_many :recipe_nutrition_items, dependent: :destroy
  has_many :recipes, through: :recipe_nutrition_items
  has_many :dri_values, dependent: :destroy

  validates :name, presence: true
  validates :dri_identifier, presence: true, uniqueness: true
  validates :nutrient_category, presence: true
  validates :default_dri_unit, presence: true
  validates :recipe_analysis_unit, presence: true

  scope :macronutrients, -> { where(nutrient_category: 'macronutrient') }
  scope :micronutrients, -> { where(nutrient_category: 'micronutrient') }
  scope :vitamins, -> { where(nutrient_category: 'vitamin') }
  scope :minerals, -> { where(nutrient_category: 'mineral') }
end
```

### LifeStageGroup
```ruby
# app/models/life_stage_group.rb
class LifeStageGroup < ApplicationRecord
  has_many :dri_values, dependent: :destroy
  has_many :nutrients, through: :dri_values
  has_many :eer_profiles, dependent: :destroy
  has_many :eer_additive_components, dependent: :destroy
  has_one :reference_anthropometry, dependent: :destroy
  has_one :growth_factor, dependent: :destroy
  has_many :pal_definitions, dependent: :destroy

  validates :name, presence: true, uniqueness: true
  validates :min_age_months, presence: true, numericality: { only_integer: true, greater_than_or_equal_to: 0 }
  validates :max_age_months, presence: true, numericality: { only_integer: true, greater_than: :min_age_months }
  validates :sex, presence: true

  scope :for_age, ->(age_months) { where('min_age_months <= ? AND max_age_months >= ?', age_months, age_months) }
  scope :for_sex, ->(sex) { where(sex: sex) }
  scope :for_pregnancy, ->(trimester) { where(special_condition: 'pregnancy', trimester: trimester) }
  scope :for_lactation, ->(period) { where(special_condition: 'lactation', lactation_period: period) }
end
```

### DriValue
```ruby
# app/models/dri_value.rb
class DriValue < ApplicationRecord
  belongs_to :nutrient
  belongs_to :life_stage_group

  validates :dri_type, presence: true
  validates :unit, presence: true
  validates :nutrient_id, uniqueness: { scope: [:life_stage_group_id, :dri_type] }

  scope :rda, -> { where(dri_type: 'RDA') }
  scope :ai, -> { where(dri_type: 'AI') }
  scope :ul, -> { where(dri_type: 'UL') }
  scope :amdr, -> { where(dri_type: 'AMDR') }
end
```

## EER Calculation Models

### EerProfile
```ruby
# app/models/eer_profile.rb
class EerProfile < ApplicationRecord
  belongs_to :life_stage_group, optional: true

  validates :name, presence: true, uniqueness: true
  validates :sex_filter, presence: true
  validates :equation_basis, presence: true

  scope :for_sex, ->(sex) { where(sex_filter: sex) }
  scope :for_age, ->(age_months) { where('age_min_months_filter <= ? AND age_max_months_filter >= ?', age_months, age_months) }
end
```

### EerAdditiveComponent
```ruby
# app/models/eer_additive_component.rb
class EerAdditiveComponent < ApplicationRecord
  belongs_to :life_stage_group, optional: true

  validates :component_type, presence: true
  validates :value_kcal_day, presence: true, numericality: { only_integer: true }

  scope :for_pregnancy, ->(trimester) { where(component_type: 'pregnancy', condition_pregnancy_trimester_filter: trimester) }
  scope :for_lactation, ->(period) { where(component_type: 'lactation', condition_lactation_period_filter: period) }
end
```

### ReferenceAnthropometry
```ruby
# app/models/reference_anthropometry.rb
class ReferenceAnthropometry < ApplicationRecord
  belongs_to :life_stage_group

  validates :life_stage_group_id, uniqueness: true
  validates :reference_height_cm, numericality: { greater_than: 0 }, allow_nil: true
  validates :reference_weight_kg, numericality: { greater_than: 0 }, allow_nil: true
  validates :median_bmi, numericality: { greater_than: 0 }, allow_nil: true
end
```

### GrowthFactor
```ruby
# app/models/growth_factor.rb
class GrowthFactor < ApplicationRecord
  belongs_to :life_stage_group

  validates :life_stage_group_id, uniqueness: true
  validates :factor_value, presence: true, numericality: { greater_than: 0 }
end
```

### PalDefinition
```ruby
# app/models/pal_definition.rb
class PalDefinition < ApplicationRecord
  belongs_to :life_stage_group

  validates :pal_category, presence: true
  validates :pal_range_min_value, presence: true, numericality: { greater_than: 0 }
  validates :pal_range_max_value, presence: true, numericality: { greater_than: :pal_range_min_value }
  validates :life_stage_group_id, uniqueness: { scope: :pal_category }

  scope :for_category, ->(category) { where(pal_category: category) }
end
```

## Usage Examples

### Finding DRI Values
```ruby
# Find RDA for protein for a 30-year-old male
life_stage = LifeStageGroup.for_age(360).for_sex('male').first
protein = Nutrient.find_by(dri_identifier: 'PROCNT')
dri_value = DriValue.find_by(
  nutrient: protein,
  life_stage_group: life_stage,
  dri_type: 'RDA'
)
```

### Calculating EER
```ruby
# Find EER profile for a 30-year-old male
profile = EerProfile.for_sex('male').for_age(360).first

# Get PAL value for active lifestyle
pal = PalDefinition.find_by(
  life_stage_group: life_stage,
  pal_category: 'active'
)

# Calculate EER using profile coefficients
eer = profile.coefficient_intercept +
      (profile.coefficient_age_years * 30) +
      (profile.coefficient_height_cm * height_cm) +
      (profile.coefficient_weight_kg * weight_kg) +
      (profile.coefficient_pal_value * pal.coefficient_for_eer_equation)
```

### Adding Pregnancy/Lactation Components
```ruby
# Add pregnancy component for second trimester
pregnancy_component = EerAdditiveComponent.for_pregnancy(2).first
eer += pregnancy_component.value_kcal_day if user.is_pregnant?

# Add lactation component for 0-6 months
lactation_component = EerAdditiveComponent.for_lactation('0-6 months').first
eer += lactation_component.value_kcal_day if user.is_lactating?
```

### Reference Values
```ruby
# Get reference height/weight for age/sex
reference = ReferenceAnthropometry.find_by(life_stage_group: life_stage)
reference_height = reference.reference_height_cm
reference_weight = reference.reference_weight_kg

# Get growth factor for children
growth = GrowthFactor.find_by(life_stage_group: life_stage)
growth_factor = growth.factor_value
``` 