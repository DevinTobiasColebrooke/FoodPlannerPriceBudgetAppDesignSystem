# User Model Implementation

## Model Definition

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy

  # Goals
  has_many :user_goals, dependent: :destroy
  has_many :goals, through: :user_goals
  has_one :primary_user_goal, -> { where(is_primary: true) }, class_name: 'UserGoal'
  has_one :primary_goal, through: :primary_user_goal, source: :goal

  # Allergies
  has_many :user_allergies, dependent: :destroy
  has_many :allergies, through: :user_allergies

  # Dietary Restrictions
  has_many :user_dietary_restrictions, dependent: :destroy
  has_many :dietary_restrictions, through: :user_dietary_restrictions

  # Kitchen Equipment
  has_many :user_kitchen_equipments, dependent: :destroy
  has_many :kitchen_equipments, through: :user_kitchen_equipments

  # Recipes
  has_many :recipes, foreign_key: :creator_id
  has_many :meal_plan_entries, dependent: :destroy
  has_many :planned_recipes, through: :meal_plan_entries, source: :recipe

  # Shopping Lists
  has_many :shopping_lists, dependent: :destroy

  # Enums
  enum cooking_time_preference: { quick: 'quick', moderate: 'moderate', leisurely: 'leisurely' }, _prefix: true
  enum meal_difficulty_preference: { easy: 'easy', medium: 'medium', involved: 'involved' }, _prefix: true
  enum shopping_difficulty_preference: { convenient: 'convenient', cheapest: 'cheapest', balanced: 'balanced' }, _prefix: true
  enum location_preference_type: { auto: 'auto', region: 'region' }, _prefix: true
  enum sex: { male: 'male', female: 'female', intersex_consideration: 'intersex_consideration' }
  enum physical_activity_level: { sedentary: 'sedentary', low_active: 'low_active', active: 'active', very_active: 'very_active' }, _prefix: true
  enum lactation_period: { zero_to_six_months: '0-6 months', seven_to_twelve_months: '7-12 months' }, _prefix: true

  # Validations
  validates :age_in_months, presence: true, numericality: { only_integer: true, greater_than_or_equal_to: 0 }
  validates :sex, presence: true
  validates :height_cm, presence: true, numericality: { greater_than: 0 }
  validates :weight_kg, presence: true, numericality: { greater_than: 0 }
  validates :physical_activity_level, presence: true
  validates :pregnancy_trimester, inclusion: { in: [1, 2, 3], allow_nil: true }, if: :is_pregnant?
  validates :lactation_period, presence: true, if: :is_lactating?

  # Normalizations
  normalizes :email, with: ->(email) { email.strip.downcase }
end
```

## Related Models

### UserGoal
```ruby
# app/models/user_goal.rb
class UserGoal < ApplicationRecord
  belongs_to :user
  belongs_to :goal
  validates :user_id, uniqueness: { scope: :goal_id }
  scope :primary, -> { where(is_primary: true) }
  scope :secondary, -> { where(is_primary: false) }
end
```

### UserAllergy
```ruby
# app/models/user_allergy.rb
class UserAllergy < ApplicationRecord
  belongs_to :user
  belongs_to :allergy
  validates :user_id, uniqueness: { scope: :allergy_id }
end
```

### UserDietaryRestriction
```ruby
# app/models/user_dietary_restriction.rb
class UserDietaryRestriction < ApplicationRecord
  belongs_to :user
  belongs_to :dietary_restriction
  validates :user_id, uniqueness: { scope: :dietary_restriction_id }
end
```

### UserKitchenEquipment
```ruby
# app/models/user_kitchen_equipment.rb
class UserKitchenEquipment < ApplicationRecord
  belongs_to :user
  belongs_to :kitchen_equipment
  validates :user_id, uniqueness: { scope: :kitchen_equipment_id }
end
```

## Usage Examples

### Creating a User
```ruby
user = User.create!(
  email: 'user@example.com',
  password: 'password123',
  age_in_months: 360, # 30 years
  sex: 'male',
  height_cm: 175.0,
  weight_kg: 70.0,
  physical_activity_level: 'active'
)
```

### Adding Goals
```ruby
weight_loss = Goal.find_by(name: 'Weight Loss')
user.user_goals.create!(goal: weight_loss, is_primary: true)
```

### Adding Allergies
```ruby
dairy = Allergy.find_by(name: 'Dairy')
user.user_allergies.create!(allergy: dairy, severity: 'severe')
```

### Adding Dietary Restrictions
```ruby
vegetarian = DietaryRestriction.find_by(name: 'Vegetarian')
user.user_dietary_restrictions.create!(dietary_restriction: vegetarian)
```

### Adding Kitchen Equipment
```ruby
blender = KitchenEquipment.find_by(name: 'Blender')
user.user_kitchen_equipments.create!(kitchen_equipment: blender)
``` 