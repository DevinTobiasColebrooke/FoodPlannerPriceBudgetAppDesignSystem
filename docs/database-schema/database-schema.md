# Database Schema

This document outlines the tables and attributes required for the application.

## Database Schema Outline

Below are the proposed tables, attributes, and their relationships.

---

### 1. `users` table

Stores information about the users.

| Column Name                     | Data Type                | Constraints & Notes                                  |
|---------------------------------|--------------------------|------------------------------------------------------|
| `id`                            | `BIGSERIAL PRIMARY KEY`  |                                                      |
| `username`                      | `VARCHAR(255)`           | `UNIQUE`, `NOT NULL` (or email)                      |
| `email`                         | `VARCHAR(255)`           | `UNIQUE`, `NOT NULL`                                 |
| `password_digest`               | `VARCHAR(255)`           | `NOT NULL` (for storing hashed passwords)            |
| `avatar_url`                    | `VARCHAR(255)`           | Default avatar if not chosen                         |
| `household_size`                | `VARCHAR(50)`            | e.g., '1-4', '5-10'                                  |
| `cooking_time_preference`       | `VARCHAR(50)`            | e.g., 'quick', 'moderate', 'leisurely' (Rails enum)  |
| `meal_difficulty_preference`    | `VARCHAR(50)`            | e.g., 'easy', 'medium', 'involved' (Rails enum)      |
| `shopping_difficulty_preference`| `VARCHAR(50)`            | e.g., 'convenient', 'cheapest', 'balanced' (Rails enum)|
| `weekly_budget`                 | `DECIMAL(10, 2)`         | Optional                                             |
| `flexible_budget`               | `BOOLEAN`                | Default `true`                                       |
| `location_preference_type`      | `VARCHAR(50)`            | e.g., 'auto', 'region', `null` (Rails enum)          |
| `zip_code`                      | `VARCHAR(20)`            | If location_preference_type is 'region'              |
| `age_in_months`                 | `INTEGER`                | `NOT NULL`. Required for DRI calculations (store total months for precision) |
| `sex`                           | `VARCHAR(20)`            | `NOT NULL`. e.g., 'male', 'female', 'intersex_consideration'. Required for DRI (Rails enum) |
| `height_cm`                     | `DECIMAL(5, 1)`          | `NOT NULL`. Store in a consistent unit (cm). Required for DRI calculations |
| `weight_kg`                     | `DECIMAL(5, 2)`          | `NOT NULL`. Store in a consistent unit (kg). Required for DRI calculations |
| `physical_activity_level`       | `VARCHAR(50)`            | `NOT NULL`. e.g., 'sedentary', 'low_active', 'active', 'very_active'. Required for EER (Rails enum) |
| `is_pregnant`                   | `BOOLEAN`                | `NOT NULL`, Default `false`. Required for DRI        |
| `pregnancy_trimester`           | `INTEGER`                | Nullable. 1, 2, or 3. Required if is_pregnant is true |
| `is_lactating`                  | `BOOLEAN`                | `NOT NULL`, Default `false`. Required for DRI        |
| `lactation_period`              | `VARCHAR(20)`            | Nullable. e.g., '0-6 months', '7-12 months'. Required if is_lactating is true (Rails enum) |
| `is_smoker`                     | `BOOLEAN`                | `NOT NULL`, Default `false`. For certain DRIs (e.g., Vit C) |
| `is_vegetarian_or_vegan`        | `BOOLEAN`                | `NOT NULL`, Default `false`. For certain DRIs (e.g., Iron, B12) |
| `onboarding_completed_at`       | `TIMESTAMP`              | Nullable. Marks completion of initial onboarding.     |
| `created_at`                    | `TIMESTAMP`              | `NOT NULL`                                           |
| `updated_at`                    | `TIMESTAMP`              | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class User < ApplicationRecord
  # Goals
  has_many :user_goals
  has_many :goals, through: :user_goals
  has_one :primary_user_goal, -> { where(is_primary: true) }, class_name: 'UserGoal'
  has_one :primary_goal, through: :primary_user_goal, source: :goal
  has_many :secondary_user_goals, -> { where(is_primary: false) }, class_name: 'UserGoal'
  has_many :secondary_goals, through: :secondary_user_goals, source: :goal

  # Dietary Preferences
  has_many :user_allergies
  has_many :allergies, through: :user_allergies
  has_many :user_dietary_restrictions
  has_many :dietary_restrictions, through: :user_dietary_restrictions

  # Kitchen & Recipes
  has_many :user_kitchen_equipments
  has_many :kitchen_equipments, through: :user_kitchen_equipments
  has_many :created_recipes, class_name: 'Recipe', foreign_key: 'creator_id'

  # Meal Planning & Shopping
  has_many :meal_plan_entries
  has_many :recipes_in_meal_plans, through: :meal_plan_entries, source: :recipe # Recipes in their meal plans
  has_many :shopping_lists

  # Potentially: has_many :user_preferred_stores
  # Potentially: has_one :calculated_nutrient_profile (to store their calculated DRIs if cached)
end
```

### 2. `goals` table
Stores predefined goals users can select.

| Column Name     | Data Type          | Constraints & Notes                                  |
|-----------------|--------------------|------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                      |
| `name`          | `VARCHAR(100)`     | `UNIQUE`, `NOT NULL` (e.g., "Weight Loss", "Eat Healthier") |
| `description`   | `TEXT`             | Optional, for tooltips or further info               |
| `created_at`    | `TIMESTAMP`        | `NOT NULL`                                           |
| `updated_at`    | `TIMESTAMP`        | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class Goal < ApplicationRecord
  has_many :user_goals
  has_many :users, through: :user_goals
end
```

### 3. `user_goals` table (Join Table)
Links users to their selected goals, distinguishing primary and secondary goals.

| Column Name      | Data Type             | Constraints & Notes                                   |
|------------------|-----------------------|-------------------------------------------------------|
| `id`             | `BIGSERIAL PRIMARY KEY` |                                                       |
| `user_id`        | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                    |
| `goal_id`        | `BIGINT`              | `REFERENCES goals(id)`, `NOT NULL`                    |
| `is_primary`     | `BOOLEAN`             | `NOT NULL`, Default `false` (True for the main goal)  |
| `custom_details` | `TEXT`                | Optional, user-specific notes for this goal selection |
| `created_at`     | `TIMESTAMP`           | `NOT NULL`                                            |
| `updated_at`     | `TIMESTAMP`           | `NOT NULL`                                            |
|                  |                       | `UNIQUE (user_id, goal_id)`                           |

**Active Record Associations:**
```ruby
class UserGoal < ApplicationRecord
  belongs_to :user
  belongs_to :goal

  scope :primary, -> { where(is_primary: true) }
  scope :secondary, -> { where(is_primary: false) }
end
```

### 4. `allergies` table
Stores predefined common allergies.

| Column Name  | Data Type             | Constraints & Notes |
|--------------|-----------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`  |
| `created_at` | `TIMESTAMP`           | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`           | `NOT NULL`          |

**Active Record Associations:**
```ruby
class Allergy < ApplicationRecord
  has_many :user_allergies
  has_many :users, through: :user_allergies
end
```

### 5. `user_allergies` table (Join Table)
Links users to their allergies, with optional severity and notes.

| Column Name   | Data Type             | Constraints & Notes                                           |
|---------------|-----------------------|---------------------------------------------------------------|
| `id`          | `BIGSERIAL PRIMARY KEY` |                                                               |
| `user_id`     | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                            |
| `allergy_id`  | `BIGINT`              | `REFERENCES allergies(id)`, `NOT NULL`                        |
| `severity`    | `VARCHAR(50)`         | Nullable (e.g., 'mild', 'moderate', 'severe' - Rails enum) |
| `notes`       | `TEXT`                | Nullable (User-specific notes about the allergy)              |
| `created_at`  | `TIMESTAMP`           | `NOT NULL`                                                    |
| `updated_at`  | `TIMESTAMP`           | `NOT NULL`                                                    |
|               |                       | `UNIQUE (user_id, allergy_id)`                                |

**Active Record Associations:**
```ruby
class UserAllergy < ApplicationRecord
  belongs_to :user
  belongs_to :allergy
end
```

### 6. `dietary_restrictions` table
Stores predefined dietary restrictions (e.g., vegan, vegetarian, gluten-free).

| Column Name   | Data Type             | Constraints & Notes                                  |
|---------------|-----------------------|------------------------------------------------------|
| `id`          | `BIGSERIAL PRIMARY KEY` |                                                      |
| `name`        | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`                                 |
| `description` | `TEXT`                | Optional, further details about the restriction        |
| `created_at`  | `TIMESTAMP`           | `NOT NULL`                                           |
| `updated_at`  | `TIMESTAMP`           | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class DietaryRestriction < ApplicationRecord
  has_many :user_dietary_restrictions
  has_many :users, through: :user_dietary_restrictions
  has_many :recipe_dietary_restrictions # If tagging recipes
  has_many :recipes, through: :recipe_dietary_restrictions # If tagging recipes
end
```

### 7. `user_dietary_restrictions` table (Join Table)
Links users to their dietary restrictions.

| Column Name              | Data Type             | Constraints & Notes                               |
|--------------------------|-----------------------|---------------------------------------------------|
| `id`                     | `BIGSERIAL PRIMARY KEY` |                                                   |
| `user_id`                | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                |
| `dietary_restriction_id` | `BIGINT`              | `REFERENCES dietary_restrictions(id)`, `NOT NULL` |
| `created_at`             | `TIMESTAMP`           | `NOT NULL`                                        |
| `updated_at`             | `TIMESTAMP`           | `NOT NULL`                                        |
|                          |                       | `UNIQUE (user_id, dietary_restriction_id)`        |

**Active Record Associations:**
```ruby
class UserDietaryRestriction < ApplicationRecord
  belongs_to :user
  belongs_to :dietary_restriction
end
```

### 8. `kitchen_equipments` table
Stores predefined common kitchen equipment.

| Column Name  | Data Type             | Constraints & Notes                                  |
|--------------|-----------------------|------------------------------------------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                                                      |
| `name`       | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`                                 |
| `created_at` | `TIMESTAMP`           | `NOT NULL`                                           |
| `updated_at` | `TIMESTAMP`           | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class KitchenEquipment < ApplicationRecord
  has_many :user_kitchen_equipments
  has_many :users, through: :user_kitchen_equipments
end
```

### 9. `user_kitchen_equipments` table (Join Table)
Links users to their kitchen equipment.

| Column Name           | Data Type             | Constraints & Notes                               |
|-----------------------|-----------------------|---------------------------------------------------|
| `id`                  | `BIGSERIAL PRIMARY KEY` |                                                   |
| `user_id`             | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                |
| `kitchen_equipment_id` | `BIGINT`              | `REFERENCES kitchen_equipments(id)`, `NOT NULL` |
| `created_at`          | `TIMESTAMP`           | `NOT NULL`                                        |
| `updated_at`          | `TIMESTAMP`           | `NOT NULL`                                        |
|                       |                       | `UNIQUE (user_id, kitchen_equipment_id)`        |

**Active Record Associations:**
```ruby
class UserKitchenEquipment < ApplicationRecord
  belongs_to :user
  belongs_to :kitchen_equipment
end
```

### Recipe and Food Composition Data

### 10. `recipes` table
Stores details for each recipe.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`                                                               |
| `description`              | `TEXT`                |                                                                          |
| `image_url`                | `VARCHAR(255)`        |                                                                          |
| `prep_time_minutes`        | `INTEGER`             |                                                                          |
| `cook_time_minutes`        | `INTEGER`             |                                                                          |
| `total_time_minutes`       | `INTEGER`             | Calculated or stored, useful for filtering                               |
| `instructions`             | `TEXT`                |                                                                          |
| `serving_size_description` | `VARCHAR(100)`        | e.g., "1 bowl", "2 servings" (user-friendly display)                     |
| `number_of_servings`       | `INTEGER`             | `NOT NULL`, Default 1. More structured for calculations.                 |
| `source_name`              | `VARCHAR(255)`        | Optional: e.g., "USDA", "User Submitted"                                 |
| `source_url`               | `VARCHAR(255)`        | Optional                                                                 |
| `difficulty_level`         | `VARCHAR(50)`         | e.g., 'easy', 'medium', 'hard' (Rails enum)                             |
| `creator_id`               | `BIGINT`              | Nullable, `REFERENCES users(id)`. If users can create recipes            |
| `is_public`                | `BOOLEAN`             | `NOT NULL`, Default `false`. If users can create private/public recipes    |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |

**Active Record Associations:**
```ruby
class Recipe < ApplicationRecord
  has_many :recipe_ingredients
  has_many :ingredients, through: :recipe_ingredients
  has_many :recipe_nutrition_items
  has_many :nutrients_in_recipe, through: :recipe_nutrition_items, source: :nutrient
  has_many :meal_plan_entries
  belongs_to :creator, class_name: 'User', foreign_key: 'creator_id', optional: true

  # For tagging recipes with dietary restrictions (optional extension)
  # has_many :recipe_dietary_restrictions
  # has_many :dietary_restrictions, through: :recipe_dietary_restrictions
end
```

### 11. `ingredients` table
Stores individual ingredients.

| Column Name     | Data Type             | Constraints & Notes                                                           |
|-----------------|-----------------------|-------------------------------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                                               |
| `name`          | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL`                                                          |
| `category`      | `VARCHAR(100)`        | Optional: e.g., 'Dairy', 'Produce', 'Pantry' (Rails enum)                    |
| `default_unit`  | `VARCHAR(50)`         | Optional, e.g., 'g', 'ml', 'piece' (for shopping list)                     |
| `fdc_id`        | `VARCHAR(50)`         | Nullable. USDA FoodData Central ID for nutrient lookup                        |
| `notes`         | `TEXT`                | Optional, for any additional details like common refuse percentage, etc.      |
| `created_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |
| `updated_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |

**Active Record Associations:**
```ruby
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients
  has_many :shopping_list_items
  # Optional: has_many :ingredient_nutrition_facts (if storing nutrition per ingredient directly)
end
```

### 12. `recipe_ingredients` table (Join Table)
Links recipes to their ingredients, including quantity and units.

| Column Name       | Data Type             | Constraints & Notes                                                                 |
|-------------------|-----------------------|-------------------------------------------------------------------------------------|
| `id`              | `BIGSERIAL PRIMARY KEY` |                                                                                     |
| `recipe_id`       | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                |
| `ingredient_id`   | `BIGINT`              | `REFERENCES ingredients(id)`, `NOT NULL`                                            |
| `quantity`        | `DECIMAL(10, 4)`      | `NOT NULL`. e.g., 0.5, 2.0.                                                         |
| `unit`            | `VARCHAR(50)`         | `NOT NULL`. e.g., "cup", "tbsp", "g", "oz", "piece"                                 |
| `notes`           | `VARCHAR(255)`        | Optional: e.g., "chopped", "to taste"                                               |
| `order_in_recipe` | `INTEGER`             | Optional, for displaying ingredients in a specific order                              |
| `created_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
| `updated_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
|                   |                       | `UNIQUE (recipe_id, ingredient_id, notes)` (Notes might differentiate "1 apple" vs "1 apple, cored") |

**Active Record Associations:**
```ruby
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient
end
```

### Dietary Reference Intake (DRI) Core Data Tables

### 13. `nutrients` table
Stores information about each nutrient or dietary component relevant for DRIs and recipe analysis.

| Column Name                       | Data Type             | Constraints & Notes                                                                                                                               |
|-----------------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                              | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                   |
| `name`                            | `VARCHAR(255)`        | `NOT NULL` (User-facing name, e.g., "Vitamin A", "Added Sugars (% kcal)")                                                                          |
| `dri_identifier`                  | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`. Stable internal key for DRI lookups (e.g., "VITAMIN_A_RAE", "ADDED_SUGARS_PERCENT_KCAL", "PROTEIN_G"). Indexed.               |
| `nutrient_category`               | `VARCHAR(100)`        | `NOT NULL` (e.g., 'Macronutrient', 'Vitamin', 'Mineral', 'Essential Fatty Acid', 'Dietary Guideline', 'Energy Component' - Rails enum)              |
| `default_dri_unit`                | `VARCHAR(50)`         | `NOT NULL` (Unit used in DRI tables: "g", "mg", "mcg", "mcg RAE", "mg AT", "IU", "% kcal", "L", " g/1000kcal ")                                    |
| `recipe_analysis_unit`            | `VARCHAR(50)`         | `NOT NULL` (Unit typically from food composition DBs for this nutrient: "g", "mg", "mcg", "IU")                                                      |
| `conversion_factor_to_dri_unit`   | `DECIMAL(12, 6)`      | Nullable. Factor to multiply recipe_analysis_unit value to get default_dri_unit value (e.g., for Vit D IU to mcg is 0.025).                          |
| `description`                     | `TEXT`                | Optional: Notes about the nutrient, RAE/DFE/NE definitions, how it's measured.                                                                      |
| `sort_order`                      | `INTEGER`             | Optional. For consistent display order in UI.                                                                                                     |
| `created_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |
| `updated_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |

**Active Record Associations:**
```ruby
class Nutrient < ApplicationRecord
  has_many :recipe_nutrition_items # Links to nutritional content of recipes
  has_many :dri_values             # Links to DRI reference values
end
```

### 14. `recipe_nutrition_items` table (Join Table)
Stores the value of a specific nutrient for a recipe.

| Column Name         | Data Type             | Constraints & Notes                                                                       |
|---------------------|-----------------------|-------------------------------------------------------------------------------------------|
| `id`                | `BIGSERIAL PRIMARY KEY` |                                                                                           |
| `recipe_id`         | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                      |
| `nutrient_id`       | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                    |
| `value_per_recipe`  | `DECIMAL(12, 5)`      | `NOT NULL`. Total amount of the nutrient in the whole recipe.                             |
| `unit`              | `VARCHAR(50)`         | `NOT NULL`. Unit for value_per_recipe (should match `nutrients.recipe_analysis_unit`).      |
| `value_per_serving` | `DECIMAL(12, 5)`      | `NOT NULL`. Calculated: `value_per_recipe` / `recipe.number_of_servings`.                   |
| `created_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
| `updated_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
|                     |                       | `UNIQUE (recipe_id, nutrient_id)`                                                         |

**Active Record Associations:**
```ruby
class RecipeNutritionItem < ApplicationRecord
  belongs_to :recipe
  belongs_to :nutrient
end
```

### 15. `life_stage_groups` table
Defines the specific demographic groups for which DRIs are set.

| Column Name        | Data Type             | Constraints & Notes                                                                                                       |
|--------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                                                                                                                           |
| `name`             | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL` (e.g., "Infants 6-11 months", "Females 19-30 years", "Pregnancy 19-30 years T2", "Males 2-3 years") |
| `min_age_months`   | `INTEGER`             | `NOT NULL` (Lower bound of age in months, inclusive)                                                                      |
| `max_age_months`   | `INTEGER`             | `NOT NULL` (Upper bound of age in months, inclusive, e.g., 11 for 0-11m, 47 for 2-3y, 9999 for 71+y)                      |
| `sex`              | `VARCHAR(20)`         | `NOT NULL` ('male', 'female', 'all' - Rails enum)                                                                      |
| `special_condition`| `VARCHAR(50)`         | Nullable ('pregnancy', 'lactation' - Rails enum)                                                                       |
| `trimester`        | `INTEGER`             | Nullable. 1, 2, or 3 (if `special_condition` is 'pregnancy')                                                             |
| `lactation_period` | `VARCHAR(20)`         | Nullable. '0-6 months', '7-12 months' (if `special_condition` is 'lactation' - Rails enum)                             |
| `notes`            | `TEXT`                | Optional. For specific descriptions from DRI tables (e.g., from "Life Stage Group" column of S-tables).                   |
| `created_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |
| `updated_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |

**Active Record Associations:**
```ruby
class LifeStageGroup < ApplicationRecord
  has_many :dri_values
  has_many :reference_anthropometries
  has_many :eer_profiles # EER profiles can be specific to life stages
  has_many :eer_additive_components # Additive components can be specific to life stages
  has_many :pal_definitions # PAL definitions can be specific to life stages
end
```

### 16. `dri_values` table
Stores the actual DRI values, linking nutrients and life_stage_groups.

| Column Name                 | Data Type             | Constraints & Notes                                                                                                                                                                                                                            |
|-----------------------------|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                        | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                                                                                                                |
| `nutrient_id`               | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                                                                                                                                                                         |
| `life_stage_group_id`       | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`                                                                                                                                                                                                 |
| `dri_type`                  | `VARCHAR(50)`         | `NOT NULL` ('EAR', 'RDA', 'AI', 'UL', 'AMDR_min_percent', 'AMDR_max_percent', 'CDRR', 'Guideline_absolute', 'Guideline_g_per_1000kcal', 'Guideline_percent_kcal_limit' - Rails enum)                                                 |
| `value_numeric`             | `DECIMAL(12, 5)`      | Nullable. For quantitative DRI values.                                                                                                                                                                                                         |
| `value_string`              | `VARCHAR(50)`         | Nullable. For "n/a", "<10", "ND", or other textual DRI outcomes.                                                                                                                                                                              |
| `unit`                      | `VARCHAR(50)`         | `NOT NULL`. Unit for `value_numeric` (e.g., "g", "mg", "% kcal"). Should match `nutrients.default_dri_unit` or be specific to the guideline type.                                                                                                 |
| `source_of_goal`            | `VARCHAR(100)`        | Nullable. From "Source of Goal" column in DGA tables (e.g., "RDA", "AI", "AMDR", "DGA", "14g/1,000 kcal").                                                                                                                                    |
| `criterion`                 | `TEXT`                | Nullable. Physiological basis for the DRI (from S-tables or other IOM reports).                                                                                                                                                              |
| `footnote_marker`           | `VARCHAR(10)`         | Nullable. e.g., "b", "c", "d" from source tables if a specific footnote applies directly.                                                                                                                                                      |
| `notes`                     | `TEXT`                | Nullable. For detailed footnotes or conditions (e.g., "UL applies to supplemental forms only", "n/a for this age group").                                                                                                                    |
| `source_document_reference` | `VARCHAR(255)`        | Nullable. To track which DRI report/table the value came from (e.g., "DGA App. A1-1", "IOM VitC S-1").                                                                                                                                         |
| `created_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
| `updated_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
|                             |                       | `UNIQUE (nutrient_id, life_stage_group_id, dri_type)` (Needs careful consideration for uniqueness if a nutrient can have multiple values of the same type for the same group under slightly different uncaptured conditions. Postgres handles NULLs as distinct in unique constraints by default.) |

**Active Record Associations:**
```ruby
class DriValue < ApplicationRecord
  belongs_to :nutrient
  belongs_to :life_stage_group
end
```
Meal Planning and Shopping (Tables 17-20)
// ... existing code ...
```

# Clicked Code Block to apply Changes From

// ... existing code ...
end
```

### Recipe and Food Composition Data

### 10. `recipes` table
Stores details for each recipe.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`                                                               |
| `description`              | `TEXT`                |                                                                          |
| `image_url`                | `VARCHAR(255)`        |                                                                          |
| `prep_time_minutes`        | `INTEGER`             |                                                                          |
| `cook_time_minutes`        | `INTEGER`             |                                                                          |
| `total_time_minutes`       | `INTEGER`             | Calculated or stored, useful for filtering                               |
| `instructions`             | `TEXT`                |                                                                          |
| `serving_size_description` | `VARCHAR(100)`        | e.g., "1 bowl", "2 servings" (user-friendly display)                     |
| `number_of_servings`       | `INTEGER`             | `NOT NULL`, Default 1. More structured for calculations.                 |
| `source_name`              | `VARCHAR(255)`        | Optional: e.g., "USDA", "User Submitted"                                 |
| `source_url`               | `VARCHAR(255)`        | Optional                                                                 |
| `difficulty_level`         | `VARCHAR(50)`         | e.g., 'easy', 'medium', 'hard' (Rails enum)                             |
| `creator_id`               | `BIGINT`              | Nullable, `REFERENCES users(id)`. If users can create recipes            |
| `is_public`                | `BOOLEAN`             | `NOT NULL`, Default `false`. If users can create private/public recipes    |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |

**Active Record Associations:**
```ruby
class Recipe < ApplicationRecord
  has_many :recipe_ingredients
  has_many :ingredients, through: :recipe_ingredients
  has_many :recipe_nutrition_items
  has_many :nutrients_in_recipe, through: :recipe_nutrition_items, source: :nutrient
  has_many :meal_plan_entries
  belongs_to :creator, class_name: 'User', foreign_key: 'creator_id', optional: true

  # For tagging recipes with dietary restrictions (optional extension)
  # has_many :recipe_dietary_restrictions
  # has_many :dietary_restrictions, through: :recipe_dietary_restrictions
end
```

### 11. `ingredients` table
Stores individual ingredients.

| Column Name     | Data Type             | Constraints & Notes                                                           |
|-----------------|-----------------------|-------------------------------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                                               |
| `name`          | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL`                                                          |
| `category`      | `VARCHAR(100)`        | Optional: e.g., 'Dairy', 'Produce', 'Pantry' (Rails enum)                    |
| `default_unit`  | `VARCHAR(50)`         | Optional, e.g., 'g', 'ml', 'piece' (for shopping list)                     |
| `fdc_id`        | `VARCHAR(50)`         | Nullable. USDA FoodData Central ID for nutrient lookup                        |
| `notes`         | `TEXT`                | Optional, for any additional details like common refuse percentage, etc.      |
| `created_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |
| `updated_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |

**Active Record Associations:**
```ruby
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients
  has_many :shopping_list_items
  # Optional: has_many :ingredient_nutrition_facts (if storing nutrition per ingredient directly)
end
```

### 12. `recipe_ingredients` table (Join Table)
Links recipes to their ingredients, including quantity and units.

| Column Name       | Data Type             | Constraints & Notes                                                                 |
|-------------------|-----------------------|-------------------------------------------------------------------------------------|
| `id`              | `BIGSERIAL PRIMARY KEY` |                                                                                     |
| `recipe_id`       | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                |
| `ingredient_id`   | `BIGINT`              | `REFERENCES ingredients(id)`, `NOT NULL`                                            |
| `quantity`        | `DECIMAL(10, 4)`      | `NOT NULL`. e.g., 0.5, 2.0.                                                         |
| `unit`            | `VARCHAR(50)`         | `NOT NULL`. e.g., "cup", "tbsp", "g", "oz", "piece"                                 |
| `notes`           | `VARCHAR(255)`        | Optional: e.g., "chopped", "to taste"                                               |
| `order_in_recipe` | `INTEGER`             | Optional, for displaying ingredients in a specific order                              |
| `created_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
| `updated_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
|                   |                       | `UNIQUE (recipe_id, ingredient_id, notes)` (Notes might differentiate "1 apple" vs "1 apple, cored") |

**Active Record Associations:**
```ruby
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient
end
```

### Dietary Reference Intake (DRI) Core Data Tables

### 13. `nutrients` table
Stores information about each nutrient or dietary component relevant for DRIs and recipe analysis.

| Column Name                       | Data Type             | Constraints & Notes                                                                                                                               |
|-----------------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                              | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                   |
| `name`                            | `VARCHAR(255)`        | `NOT NULL` (User-facing name, e.g., "Vitamin A", "Added Sugars (% kcal)")                                                                          |
| `dri_identifier`                  | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`. Stable internal key for DRI lookups (e.g., "VITAMIN_A_RAE", "ADDED_SUGARS_PERCENT_KCAL", "PROTEIN_G"). Indexed.               |
| `nutrient_category`               | `VARCHAR(100)`        | `NOT NULL` (e.g., 'Macronutrient', 'Vitamin', 'Mineral', 'Essential Fatty Acid', 'Dietary Guideline', 'Energy Component' - Rails enum)              |
| `default_dri_unit`                | `VARCHAR(50)`         | `NOT NULL` (Unit used in DRI tables: "g", "mg", "mcg", "mcg RAE", "mg AT", "IU", "% kcal", "L", " g/1000kcal ")                                    |
| `recipe_analysis_unit`            | `VARCHAR(50)`         | `NOT NULL` (Unit typically from food composition DBs for this nutrient: "g", "mg", "mcg", "IU")                                                      |
| `conversion_factor_to_dri_unit`   | `DECIMAL(12, 6)`      | Nullable. Factor to multiply recipe_analysis_unit value to get default_dri_unit value (e.g., for Vit D IU to mcg is 0.025).                          |
| `description`                     | `TEXT`                | Optional: Notes about the nutrient, RAE/DFE/NE definitions, how it's measured.                                                                      |
| `sort_order`                      | `INTEGER`             | Optional. For consistent display order in UI.                                                                                                     |
| `created_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |
| `updated_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |

**Active Record Associations:**
```ruby
class Nutrient < ApplicationRecord
  has_many :recipe_nutrition_items # Links to nutritional content of recipes
  has_many :dri_values             # Links to DRI reference values
end
```

### 14. `recipe_nutrition_items` table (Join Table)
Stores the value of a specific nutrient for a recipe.

| Column Name         | Data Type             | Constraints & Notes                                                                       |
|---------------------|-----------------------|-------------------------------------------------------------------------------------------|
| `id`                | `BIGSERIAL PRIMARY KEY` |                                                                                           |
| `recipe_id`         | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                      |
| `nutrient_id`       | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                    |
| `value_per_recipe`  | `DECIMAL(12, 5)`      | `NOT NULL`. Total amount of the nutrient in the whole recipe.                             |
| `unit`              | `VARCHAR(50)`         | `NOT NULL`. Unit for value_per_recipe (should match `nutrients.recipe_analysis_unit`).      |
| `value_per_serving` | `DECIMAL(12, 5)`      | `NOT NULL`. Calculated: `value_per_recipe` / `recipe.number_of_servings`.                   |
| `created_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
| `updated_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
|                     |                       | `UNIQUE (recipe_id, nutrient_id)`                                                         |

**Active Record Associations:**
```ruby
class RecipeNutritionItem < ApplicationRecord
  belongs_to :recipe
  belongs_to :nutrient
end
```

### 15. `life_stage_groups` table
Defines the specific demographic groups for which DRIs are set.

| Column Name        | Data Type             | Constraints & Notes                                                                                                       |
|--------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                                                                                                                           |
| `name`             | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL` (e.g., "Infants 6-11 months", "Females 19-30 years", "Pregnancy 19-30 years T2", "Males 2-3 years") |
| `min_age_months`   | `INTEGER`             | `NOT NULL` (Lower bound of age in months, inclusive)                                                                      |
| `max_age_months`   | `INTEGER`             | `NOT NULL` (Upper bound of age in months, inclusive, e.g., 11 for 0-11m, 47 for 2-3y, 9999 for 71+y)                      |
| `sex`              | `VARCHAR(20)`         | `NOT NULL` ('male', 'female', 'all' - Rails enum)                                                                      |
| `special_condition`| `VARCHAR(50)`         | Nullable ('pregnancy', 'lactation' - Rails enum)                                                                       |
| `trimester`        | `INTEGER`             | Nullable. 1, 2, or 3 (if `special_condition` is 'pregnancy')                                                             |
| `lactation_period` | `VARCHAR(20)`         | Nullable. '0-6 months', '7-12 months' (if `special_condition` is 'lactation' - Rails enum)                             |
| `notes`            | `TEXT`                | Optional. For specific descriptions from DRI tables (e.g., from "Life Stage Group" column of S-tables).                   |
| `created_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |
| `updated_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |

**Active Record Associations:**
```ruby
class LifeStageGroup < ApplicationRecord
  has_many :dri_values
  has_many :reference_anthropometries
  has_many :eer_profiles # EER profiles can be specific to life stages
  has_many :eer_additive_components # Additive components can be specific to life stages
  has_many :pal_definitions # PAL definitions can be specific to life stages
end
```

### 16. `dri_values` table
Stores the actual DRI values, linking nutrients and life_stage_groups.

| Column Name                 | Data Type             | Constraints & Notes                                                                                                                                                                                                                            |
|-----------------------------|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                        | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                                                                                                                |
| `nutrient_id`               | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                                                                                                                                                                         |
| `life_stage_group_id`       | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`                                                                                                                                                                                                 |
| `dri_type`                  | `VARCHAR(50)`         | `NOT NULL` ('EAR', 'RDA', 'AI', 'UL', 'AMDR_min_percent', 'AMDR_max_percent', 'CDRR', 'Guideline_absolute', 'Guideline_g_per_1000kcal', 'Guideline_percent_kcal_limit' - Rails enum)                                                 |
| `value_numeric`             | `DECIMAL(12, 5)`      | Nullable. For quantitative DRI values.                                                                                                                                                                                                         |
| `value_string`              | `VARCHAR(50)`         | Nullable. For "n/a", "<10", "ND", or other textual DRI outcomes.                                                                                                                                                                              |
| `unit`                      | `VARCHAR(50)`         | `NOT NULL`. Unit for `value_numeric` (e.g., "g", "mg", "% kcal"). Should match `nutrients.default_dri_unit` or be specific to the guideline type.                                                                                                 |
| `source_of_goal`            | `VARCHAR(100)`        | Nullable. From "Source of Goal" column in DGA tables (e.g., "RDA", "AI", "AMDR", "DGA", "14g/1,000 kcal").                                                                                                                                    |
| `criterion`                 | `TEXT`                | Nullable. Physiological basis for the DRI (from S-tables or other IOM reports).                                                                                                                                                              |
| `footnote_marker`           | `VARCHAR(10)`         | Nullable. e.g., "b", "c", "d" from source tables if a specific footnote applies directly.                                                                                                                                                      |
| `notes`                     | `TEXT`                | Nullable. For detailed footnotes or conditions (e.g., "UL applies to supplemental forms only", "n/a for this age group").                                                                                                                    |
| `source_document_reference` | `VARCHAR(255)`        | Nullable. To track which DRI report/table the value came from (e.g., "DGA App. A1-1", "IOM VitC S-1").                                                                                                                                         |
| `created_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
| `updated_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
|                             |                       | `UNIQUE (nutrient_id, life_stage_group_id, dri_type)` (Needs careful consideration for uniqueness if a nutrient can have multiple values of the same type for the same group under slightly different uncaptured conditions. Postgres handles NULLs as distinct in unique constraints by default.) |

**Active Record Associations:**
```ruby
class DriValue < ApplicationRecord
  belongs_to :nutrient
  belongs_to :life_stage_group
end
```
Meal Planning and Shopping (Tables 17-20)
// ... existing code ...
```

# Clicked Code Block to apply Changes From

// ... existing code ...
end
```

### Recipe and Food Composition Data

### 10. `recipes` table
Stores details for each recipe.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`                                                               |
| `description`              | `TEXT`                |                                                                          |
| `image_url`                | `VARCHAR(255)`        |                                                                          |
| `prep_time_minutes`        | `INTEGER`             |                                                                          |
| `cook_time_minutes`        | `INTEGER`             |                                                                          |
| `total_time_minutes`       | `INTEGER`             | Calculated or stored, useful for filtering                               |
| `instructions`             | `TEXT`                |                                                                          |
| `serving_size_description` | `VARCHAR(100)`        | e.g., "1 bowl", "2 servings" (user-friendly display)                     |
| `number_of_servings`       | `INTEGER`             | `NOT NULL`, Default 1. More structured for calculations.                 |
| `source_name`              | `VARCHAR(255)`        | Optional: e.g., "USDA", "User Submitted"                                 |
| `source_url`               | `VARCHAR(255)`        | Optional                                                                 |
| `difficulty_level`         | `VARCHAR(50)`         | e.g., 'easy', 'medium', 'hard' (Rails enum)                             |
| `creator_id`               | `BIGINT`              | Nullable, `REFERENCES users(id)`. If users can create recipes            |
| `is_public`                | `BOOLEAN`             | `NOT NULL`, Default `false`. If users can create private/public recipes    |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |

**Active Record Associations:**
```ruby
class Recipe < ApplicationRecord
  has_many :recipe_ingredients
  has_many :ingredients, through: :recipe_ingredients
  has_many :recipe_nutrition_items
  has_many :nutrients_in_recipe, through: :recipe_nutrition_items, source: :nutrient
  has_many :meal_plan_entries
  belongs_to :creator, class_name: 'User', foreign_key: 'creator_id', optional: true

  # For tagging recipes with dietary restrictions (optional extension)
  # has_many :recipe_dietary_restrictions
  # has_many :dietary_restrictions, through: :recipe_dietary_restrictions
end
```

### 11. `ingredients` table
Stores individual ingredients.

| Column Name     | Data Type             | Constraints & Notes                                                           |
|-----------------|-----------------------|-------------------------------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                                               |
| `name`          | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL`                                                          |
| `category`      | `VARCHAR(100)`        | Optional: e.g., 'Dairy', 'Produce', 'Pantry' (Rails enum)                    |
| `default_unit`  | `VARCHAR(50)`         | Optional, e.g., 'g', 'ml', 'piece' (for shopping list)                     |
| `fdc_id`        | `VARCHAR(50)`         | Nullable. USDA FoodData Central ID for nutrient lookup                        |
| `notes`         | `TEXT`                | Optional, for any additional details like common refuse percentage, etc.      |
| `created_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |
| `updated_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |

**Active Record Associations:**
```ruby
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients
  has_many :shopping_list_items
  # Optional: has_many :ingredient_nutrition_facts (if storing nutrition per ingredient directly)
end
```

### 12. `recipe_ingredients` table (Join Table)
Links recipes to their ingredients, including quantity and units.

| Column Name       | Data Type             | Constraints & Notes                                                                 |
|-------------------|-----------------------|-------------------------------------------------------------------------------------|
| `id`              | `BIGSERIAL PRIMARY KEY` |                                                                                     |
| `recipe_id`       | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                |
| `ingredient_id`   | `BIGINT`              | `REFERENCES ingredients(id)`, `NOT NULL`                                            |
| `quantity`        | `DECIMAL(10, 4)`      | `NOT NULL`. e.g., 0.5, 2.0.                                                         |
| `unit`            | `VARCHAR(50)`         | `NOT NULL`. e.g., "cup", "tbsp", "g", "oz", "piece"                                 |
| `notes`           | `VARCHAR(255)`        | Optional: e.g., "chopped", "to taste"                                               |
| `order_in_recipe` | `INTEGER`             | Optional, for displaying ingredients in a specific order                              |
| `created_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
| `updated_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
|                   |                       | `UNIQUE (recipe_id, ingredient_id, notes)` (Notes might differentiate "1 apple" vs "1 apple, cored") |

**Active Record Associations:**
```ruby
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient
end
```

### Dietary Reference Intake (DRI) Core Data Tables

### 13. `nutrients` table
Stores information about each nutrient or dietary component relevant for DRIs and recipe analysis.

| Column Name                       | Data Type             | Constraints & Notes                                                                                                                               |
|-----------------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                              | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                   |
| `name`                            | `VARCHAR(255)`        | `NOT NULL` (User-facing name, e.g., "Vitamin A", "Added Sugars (% kcal)")                                                                          |
| `dri_identifier`                  | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`. Stable internal key for DRI lookups (e.g., "VITAMIN_A_RAE", "ADDED_SUGARS_PERCENT_KCAL", "PROTEIN_G"). Indexed.               |
| `nutrient_category`               | `VARCHAR(100)`        | `NOT NULL` (e.g., 'Macronutrient', 'Vitamin', 'Mineral', 'Essential Fatty Acid', 'Dietary Guideline', 'Energy Component' - Rails enum)              |
| `default_dri_unit`                | `VARCHAR(50)`         | `NOT NULL` (Unit used in DRI tables: "g", "mg", "mcg", "mcg RAE", "mg AT", "IU", "% kcal", "L", " g/1000kcal ")                                    |
| `recipe_analysis_unit`            | `VARCHAR(50)`         | `NOT NULL` (Unit typically from food composition DBs for this nutrient: "g", "mg", "mcg", "IU")                                                      |
| `conversion_factor_to_dri_unit`   | `DECIMAL(12, 6)`      | Nullable. Factor to multiply recipe_analysis_unit value to get default_dri_unit value (e.g., for Vit D IU to mcg is 0.025).                          |
| `description`                     | `TEXT`                | Optional: Notes about the nutrient, RAE/DFE/NE definitions, how it's measured.                                                                      |
| `sort_order`                      | `INTEGER`             | Optional. For consistent display order in UI.                                                                                                     |
| `created_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |
| `updated_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |

**Active Record Associations:**
```ruby
class Nutrient < ApplicationRecord
  has_many :recipe_nutrition_items # Links to nutritional content of recipes
  has_many :dri_values             # Links to DRI reference values
end
```

### 14. `recipe_nutrition_items` table (Join Table)
Stores the value of a specific nutrient for a recipe.

| Column Name         | Data Type             | Constraints & Notes                                                                       |
|---------------------|-----------------------|-------------------------------------------------------------------------------------------|
| `id`                | `BIGSERIAL PRIMARY KEY` |                                                                                           |
| `recipe_id`         | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                      |
| `nutrient_id`       | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                    |
| `value_per_recipe`  | `DECIMAL(12, 5)`      | `NOT NULL`. Total amount of the nutrient in the whole recipe.                             |
| `unit`              | `VARCHAR(50)`         | `NOT NULL`. Unit for value_per_recipe (should match `nutrients.recipe_analysis_unit`).      |
| `value_per_serving` | `DECIMAL(12, 5)`      | `NOT NULL`. Calculated: `value_per_recipe` / `recipe.number_of_servings`.                   |
| `created_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
| `updated_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
|                     |                       | `UNIQUE (recipe_id, nutrient_id)`                                                         |

**Active Record Associations:**
```ruby
class RecipeNutritionItem < ApplicationRecord
  belongs_to :recipe
  belongs_to :nutrient
end
```

### 15. `life_stage_groups` table
Defines the specific demographic groups for which DRIs are set.

| Column Name        | Data Type             | Constraints & Notes                                                                                                       |
|--------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                                                                                                                           |
| `name`             | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL` (e.g., "Infants 6-11 months", "Females 19-30 years", "Pregnancy 19-30 years T2", "Males 2-3 years") |
| `min_age_months`   | `INTEGER`             | `NOT NULL` (Lower bound of age in months, inclusive)                                                                      |
| `max_age_months`   | `INTEGER`             | `NOT NULL` (Upper bound of age in months, inclusive, e.g., 11 for 0-11m, 47 for 2-3y, 9999 for 71+y)                      |
| `sex`              | `VARCHAR(20)`         | `NOT NULL` ('male', 'female', 'all' - Rails enum)                                                                      |
| `special_condition`| `VARCHAR(50)`         | Nullable ('pregnancy', 'lactation' - Rails enum)                                                                       |
| `trimester`        | `INTEGER`             | Nullable. 1, 2, or 3 (if `special_condition` is 'pregnancy')                                                             |
| `lactation_period` | `VARCHAR(20)`         | Nullable. '0-6 months', '7-12 months' (if `special_condition` is 'lactation' - Rails enum)                             |
| `notes`            | `TEXT`                | Optional. For specific descriptions from DRI tables (e.g., from "Life Stage Group" column of S-tables).                   |
| `created_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |
| `updated_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |

**Active Record Associations:**
```ruby
class LifeStageGroup < ApplicationRecord
  has_many :dri_values
  has_many :reference_anthropometries
  has_many :eer_profiles # EER profiles can be specific to life stages
  has_many :eer_additive_components # Additive components can be specific to life stages
  has_many :pal_definitions # PAL definitions can be specific to life stages
end
```

### 16. `dri_values` table
Stores the actual DRI values, linking nutrients and life_stage_groups.

| Column Name                 | Data Type             | Constraints & Notes                                                                                                                                                                                                                            |
|-----------------------------|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                        | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                                                                                                                |
| `nutrient_id`               | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                                                                                                                                                                         |
| `life_stage_group_id`       | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`                                                                                                                                                                                                 |
| `dri_type`                  | `VARCHAR(50)`         | `NOT NULL` ('EAR', 'RDA', 'AI', 'UL', 'AMDR_min_percent', 'AMDR_max_percent', 'CDRR', 'Guideline_absolute', 'Guideline_g_per_1000kcal', 'Guideline_percent_kcal_limit' - Rails enum)                                                 |
| `value_numeric`             | `DECIMAL(12, 5)`      | Nullable. For quantitative DRI values.                                                                                                                                                                                                         |
| `value_string`              | `VARCHAR(50)`         | Nullable. For "n/a", "<10", "ND", or other textual DRI outcomes.                                                                                                                                                                              |
| `unit`                      | `VARCHAR(50)`         | `NOT NULL`. Unit for `value_numeric` (e.g., "g", "mg", "% kcal"). Should match `nutrients.default_dri_unit` or be specific to the guideline type.                                                                                                 |
| `source_of_goal`            | `VARCHAR(100)`        | Nullable. From "Source of Goal" column in DGA tables (e.g., "RDA", "AI", "AMDR", "DGA", "14g/1,000 kcal").                                                                                                                                    |
| `criterion`                 | `TEXT`                | Nullable. Physiological basis for the DRI (from S-tables or other IOM reports).                                                                                                                                                              |
| `footnote_marker`           | `VARCHAR(10)`         | Nullable. e.g., "b", "c", "d" from source tables if a specific footnote applies directly.                                                                                                                                                      |
| `notes`                     | `TEXT`                | Nullable. For detailed footnotes or conditions (e.g., "UL applies to supplemental forms only", "n/a for this age group").                                                                                                                    |
| `source_document_reference` | `VARCHAR(255)`        | Nullable. To track which DRI report/table the value came from (e.g., "DGA App. A1-1", "IOM VitC S-1").                                                                                                                                         |
| `created_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
| `updated_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
|                             |                       | `UNIQUE (nutrient_id, life_stage_group_id, dri_type)` (Needs careful consideration for uniqueness if a nutrient can have multiple values of the same type for the same group under slightly different uncaptured conditions. Postgres handles NULLs as distinct in unique constraints by default.) |

**Active Record Associations:**
```ruby
class DriValue < ApplicationRecord
  belongs_to :nutrient
  belongs_to :life_stage_group
end
```
Meal Planning and Shopping (Tables 17-20)
// ... existing code ...
```

# Clicked Code Block to apply Changes From

// ... existing code ...
end
```

### Recipe and Food Composition Data

### 10. `recipes` table
Stores details for each recipe.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`                                                               |
| `description`              | `TEXT`                |                                                                          |
| `image_url`                | `VARCHAR(255)`        |                                                                          |
| `prep_time_minutes`        | `INTEGER`             |                                                                          |
| `cook_time_minutes`        | `INTEGER`             |                                                                          |
| `total_time_minutes`       | `INTEGER`             | Calculated or stored, useful for filtering                               |
| `instructions`             | `TEXT`                |                                                                          |
| `serving_size_description` | `VARCHAR(100)`        | e.g., "1 bowl", "2 servings" (user-friendly display)                     |
| `number_of_servings`       | `INTEGER`             | `NOT NULL`, Default 1. More structured for calculations.                 |
| `source_name`              | `VARCHAR(255)`        | Optional: e.g., "USDA", "User Submitted"                                 |
| `source_url`               | `VARCHAR(255)`        | Optional                                                                 |
| `difficulty_level`         | `VARCHAR(50)`         | e.g., 'easy', 'medium', 'hard' (Rails enum)                             |
| `creator_id`               | `BIGINT`              | Nullable, `REFERENCES users(id)`. If users can create recipes            |
| `is_public`                | `BOOLEAN`             | `NOT NULL`, Default `false`. If users can create private/public recipes    |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |

**Active Record Associations:**
```ruby
class Recipe < ApplicationRecord
  has_many :recipe_ingredients
  has_many :ingredients, through: :recipe_ingredients
  has_many :recipe_nutrition_items
  has_many :nutrients_in_recipe, through: :recipe_nutrition_items, source: :nutrient
  has_many :meal_plan_entries
  belongs_to :creator, class_name: 'User', foreign_key: 'creator_id', optional: true

  # For tagging recipes with dietary restrictions (optional extension)
  # has_many :recipe_dietary_restrictions
  # has_many :dietary_restrictions, through: :recipe_dietary_restrictions
end
```

### 11. `ingredients` table
Stores individual ingredients.

| Column Name     | Data Type             | Constraints & Notes                                                           |
|-----------------|-----------------------|-------------------------------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                                               |
| `name`          | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL`                                                          |
| `category`      | `VARCHAR(100)`        | Optional: e.g., 'Dairy', 'Produce', 'Pantry' (Rails enum)                    |
| `default_unit`  | `VARCHAR(50)`         | Optional, e.g., 'g', 'ml', 'piece' (for shopping list)                     |
| `fdc_id`        | `VARCHAR(50)`         | Nullable. USDA FoodData Central ID for nutrient lookup                        |
| `notes`         | `TEXT`                | Optional, for any additional details like common refuse percentage, etc.      |
| `created_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |
| `updated_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |

# Database Schema

This document outlines the tables and attributes required for the application.

## Database Schema Outline

Below are the proposed tables, attributes, and their relationships.

---

### 1. `users` table

Stores information about the users.

| Column Name                     | Data Type                | Constraints & Notes                                  |
|---------------------------------|--------------------------|------------------------------------------------------|
| `id`                            | `BIGSERIAL PRIMARY KEY`  |                                                      |
| `username`                      | `VARCHAR(255)`           | `UNIQUE`, `NOT NULL` (or email)                      |
| `email`                         | `VARCHAR(255)`           | `UNIQUE`, `NOT NULL`                                 |
| `password_digest`               | `VARCHAR(255)`           | `NOT NULL` (for storing hashed passwords)            |
| `avatar_url`                    | `VARCHAR(255)`           | Default avatar if not chosen                         |
| `household_size`                | `VARCHAR(50)`            | e.g., '1-4', '5-10'                                  |
| `cooking_time_preference`       | `VARCHAR(50)`            | e.g., 'quick', 'moderate', 'leisurely' (Rails enum)  |
| `meal_difficulty_preference`    | `VARCHAR(50)`            | e.g., 'easy', 'medium', 'involved' (Rails enum)      |
| `shopping_difficulty_preference`| `VARCHAR(50)`            | e.g., 'convenient', 'cheapest', 'balanced' (Rails enum)|
| `weekly_budget`                 | `DECIMAL(10, 2)`         | Optional                                             |
| `flexible_budget`               | `BOOLEAN`                | Default `true`                                       |
| `location_preference_type`      | `VARCHAR(50)`            | e.g., 'auto', 'region', `null` (Rails enum)          |
| `zip_code`                      | `VARCHAR(20)`            | If location_preference_type is 'region'              |
| `age_in_months`                 | `INTEGER`                | `NOT NULL`. Required for DRI calculations (store total months for precision) |
| `sex`                           | `VARCHAR(20)`            | `NOT NULL`. e.g., 'male', 'female', 'intersex_consideration'. Required for DRI (Rails enum) |
| `height_cm`                     | `DECIMAL(5, 1)`          | `NOT NULL`. Store in a consistent unit (cm). Required for DRI calculations |
| `weight_kg`                     | `DECIMAL(5, 2)`          | `NOT NULL`. Store in a consistent unit (kg). Required for DRI calculations |
| `physical_activity_level`       | `VARCHAR(50)`            | `NOT NULL`. e.g., 'sedentary', 'low_active', 'active', 'very_active'. Required for EER (Rails enum) |
| `is_pregnant`                   | `BOOLEAN`                | `NOT NULL`, Default `false`. Required for DRI        |
| `pregnancy_trimester`           | `INTEGER`                | Nullable. 1, 2, or 3. Required if is_pregnant is true |
| `is_lactating`                  | `BOOLEAN`                | `NOT NULL`, Default `false`. Required for DRI        |
| `lactation_period`              | `VARCHAR(20)`            | Nullable. e.g., '0-6 months', '7-12 months'. Required if is_lactating is true (Rails enum) |
| `is_smoker`                     | `BOOLEAN`                | `NOT NULL`, Default `false`. For certain DRIs (e.g., Vit C) |
| `is_vegetarian_or_vegan`        | `BOOLEAN`                | `NOT NULL`, Default `false`. For certain DRIs (e.g., Iron, B12) |
| `onboarding_completed_at`       | `TIMESTAMP`              | Nullable. Marks completion of initial onboarding.     |
| `created_at`                    | `TIMESTAMP`              | `NOT NULL`                                           |
| `updated_at`                    | `TIMESTAMP`              | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class User < ApplicationRecord
  # Goals
  has_many :user_goals
  has_many :goals, through: :user_goals
  has_one :primary_user_goal, -> { where(is_primary: true) }, class_name: 'UserGoal'
  has_one :primary_goal, through: :primary_user_goal, source: :goal
  has_many :secondary_user_goals, -> { where(is_primary: false) }, class_name: 'UserGoal'
  has_many :secondary_goals, through: :secondary_user_goals, source: :goal

  # Dietary Preferences
  has_many :user_allergies
  has_many :allergies, through: :user_allergies
  has_many :user_dietary_restrictions
  has_many :dietary_restrictions, through: :user_dietary_restrictions

  # Kitchen & Recipes
  has_many :user_kitchen_equipments
  has_many :kitchen_equipments, through: :user_kitchen_equipments
  has_many :created_recipes, class_name: 'Recipe', foreign_key: 'creator_id'

  # Meal Planning & Shopping
  has_many :meal_plan_entries
  has_many :recipes_in_meal_plans, through: :meal_plan_entries, source: :recipe # Recipes in their meal plans
  has_many :shopping_lists

  # Potentially: has_many :user_preferred_stores
  # Potentially: has_one :calculated_nutrient_profile (to store their calculated DRIs if cached)
end
```

### 2. `goals` table
Stores predefined goals users can select.

| Column Name     | Data Type          | Constraints & Notes                                  |
|-----------------|--------------------|------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                      |
| `name`          | `VARCHAR(100)`     | `UNIQUE`, `NOT NULL` (e.g., "Weight Loss", "Eat Healthier") |
| `description`   | `TEXT`             | Optional, for tooltips or further info               |
| `created_at`    | `TIMESTAMP`        | `NOT NULL`                                           |
| `updated_at`    | `TIMESTAMP`        | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class Goal < ApplicationRecord
  has_many :user_goals
  has_many :users, through: :user_goals
end
```

### 3. `user_goals` table (Join Table)
Links users to their selected goals, distinguishing primary and secondary goals.

| Column Name      | Data Type             | Constraints & Notes                                   |
|------------------|-----------------------|-------------------------------------------------------|
| `id`             | `BIGSERIAL PRIMARY KEY` |                                                       |
| `user_id`        | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                    |
| `goal_id`        | `BIGINT`              | `REFERENCES goals(id)`, `NOT NULL`                    |
| `is_primary`     | `BOOLEAN`             | `NOT NULL`, Default `false` (True for the main goal)  |
| `custom_details` | `TEXT`                | Optional, user-specific notes for this goal selection |
| `created_at`     | `TIMESTAMP`           | `NOT NULL`                                            |
| `updated_at`     | `TIMESTAMP`           | `NOT NULL`                                            |
|                  |                       | `UNIQUE (user_id, goal_id)`                           |

**Active Record Associations:**
```ruby
class UserGoal < ApplicationRecord
  belongs_to :user
  belongs_to :goal

  scope :primary, -> { where(is_primary: true) }
  scope :secondary, -> { where(is_primary: false) }
end
```

### 4. `allergies` table
Stores predefined common allergies.

| Column Name  | Data Type             | Constraints & Notes |
|--------------|-----------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`  |
| `created_at` | `TIMESTAMP`           | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`           | `NOT NULL`          |

**Active Record Associations:**
```ruby
class Allergy < ApplicationRecord
  has_many :user_allergies
  has_many :users, through: :user_allergies
end
```

### 5. `user_allergies` table (Join Table)
Links users to their allergies, with optional severity and notes.

| Column Name   | Data Type             | Constraints & Notes                                           |
|---------------|-----------------------|---------------------------------------------------------------|
| `id`          | `BIGSERIAL PRIMARY KEY` |                                                               |
| `user_id`     | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                            |
| `allergy_id`  | `BIGINT`              | `REFERENCES allergies(id)`, `NOT NULL`                        |
| `severity`    | `VARCHAR(50)`         | Nullable (e.g., 'mild', 'moderate', 'severe' - Rails enum) |
| `notes`       | `TEXT`                | Nullable (User-specific notes about the allergy)              |
| `created_at`  | `TIMESTAMP`           | `NOT NULL`                                                    |
| `updated_at`  | `TIMESTAMP`           | `NOT NULL`                                                    |
|               |                       | `UNIQUE (user_id, allergy_id)`                                |

**Active Record Associations:**
```ruby
class UserAllergy < ApplicationRecord
  belongs_to :user
  belongs_to :allergy
end
```

### 6. `dietary_restrictions` table
Stores predefined dietary restrictions (e.g., vegan, vegetarian, gluten-free).

| Column Name   | Data Type             | Constraints & Notes                                  |
|---------------|-----------------------|------------------------------------------------------|
| `id`          | `BIGSERIAL PRIMARY KEY` |                                                      |
| `name`        | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`                                 |
| `description` | `TEXT`                | Optional, further details about the restriction        |
| `created_at`  | `TIMESTAMP`           | `NOT NULL`                                           |
| `updated_at`  | `TIMESTAMP`           | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class DietaryRestriction < ApplicationRecord
  has_many :user_dietary_restrictions
  has_many :users, through: :user_dietary_restrictions
  has_many :recipe_dietary_restrictions # If tagging recipes
  has_many :recipes, through: :recipe_dietary_restrictions # If tagging recipes
end
```

### 7. `user_dietary_restrictions` table (Join Table)
Links users to their dietary restrictions.

| Column Name              | Data Type             | Constraints & Notes                               |
|--------------------------|-----------------------|---------------------------------------------------|
| `id`                     | `BIGSERIAL PRIMARY KEY` |                                                   |
| `user_id`                | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                |
| `dietary_restriction_id` | `BIGINT`              | `REFERENCES dietary_restrictions(id)`, `NOT NULL` |
| `created_at`             | `TIMESTAMP`           | `NOT NULL`                                        |
| `updated_at`             | `TIMESTAMP`           | `NOT NULL`                                        |
|                          |                       | `UNIQUE (user_id, dietary_restriction_id)`        |

**Active Record Associations:**
```ruby
class UserDietaryRestriction < ApplicationRecord
  belongs_to :user
  belongs_to :dietary_restriction
end
```

### 8. `kitchen_equipments` table
Stores predefined kitchen equipment.

| Column Name  | Data Type             | Constraints & Notes |
|--------------|-----------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`  |
| `created_at` | `TIMESTAMP`           | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`           | `NOT NULL`          |

**Active Record Associations:**
```ruby
class KitchenEquipment < ApplicationRecord
  has_many :user_kitchen_equipments
  has_many :users, through: :user_kitchen_equipments
end
```

### 9. `user_kitchen_equipments` table (Join Table)
Links users to the kitchen equipment they own.

| Column Name            | Data Type             | Constraints & Notes                               |
|------------------------|-----------------------|---------------------------------------------------|
| `id`                   | `BIGSERIAL PRIMARY KEY` |                                                   |
| `user_id`              | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                |
| `kitchen_equipment_id` | `BIGINT`              | `REFERENCES kitchen_equipments(id)`, `NOT NULL`   |
| `created_at`           | `TIMESTAMP`           | `NOT NULL`                                        |
| `updated_at`           | `TIMESTAMP`           | `NOT NULL`                                        |
|                        |                       | `UNIQUE (user_id, kitchen_equipment_id)`          |

**Active Record Associations:**
```ruby
class UserKitchenEquipment < ApplicationRecord
  belongs_to :user
  belongs_to :kitchen_equipment
end
```

### Recipe and Food Composition Data

### 10. `recipes` table
Stores details for each recipe.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`                                                               |
| `description`              | `TEXT`                |                                                                          |
| `image_url`                | `VARCHAR(255)`        |                                                                          |
| `prep_time_minutes`        | `INTEGER`             |                                                                          |
| `cook_time_minutes`        | `INTEGER`             |                                                                          |
| `total_time_minutes`       | `INTEGER`             | Calculated or stored, useful for filtering                               |
| `instructions`             | `TEXT`                |                                                                          |
| `serving_size_description` | `VARCHAR(100)`        | e.g., "1 bowl", "2 servings" (user-friendly display)                     |
| `number_of_servings`       | `INTEGER`             | `NOT NULL`, Default 1. More structured for calculations.                 |
| `source_name`              | `VARCHAR(255)`        | Optional: e.g., "USDA", "User Submitted"                                 |
| `source_url`               | `VARCHAR(255)`        | Optional                                                                 |
| `difficulty_level`         | `VARCHAR(50)`         | e.g., 'easy', 'medium', 'hard' (Rails enum)                             |
| `creator_id`               | `BIGINT`              | Nullable, `REFERENCES users(id)`. If users can create recipes            |
| `is_public`                | `BOOLEAN`             | `NOT NULL`, Default `false`. If users can create private/public recipes    |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                               |

**Active Record Associations:**
```ruby
class Recipe < ApplicationRecord
  has_many :recipe_ingredients
  has_many :ingredients, through: :recipe_ingredients
  has_many :recipe_nutrition_items
  has_many :nutrients_in_recipe, through: :recipe_nutrition_items, source: :nutrient
  has_many :meal_plan_entries
  belongs_to :creator, class_name: 'User', foreign_key: 'creator_id', optional: true

  # For tagging recipes with dietary restrictions (optional extension)
  # has_many :recipe_dietary_restrictions
  # has_many :dietary_restrictions, through: :recipe_dietary_restrictions
end
```

### 11. `ingredients` table
Stores individual ingredients.

| Column Name     | Data Type             | Constraints & Notes                                                           |
|-----------------|-----------------------|-------------------------------------------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                                                               |
| `name`          | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL`                                                          |
| `category`      | `VARCHAR(100)`        | Optional: e.g., 'Dairy', 'Produce', 'Pantry' (Rails enum)                    |
| `default_unit`  | `VARCHAR(50)`         | Optional, e.g., 'g', 'ml', 'piece' (for shopping list)                     |
| `fdc_id`        | `VARCHAR(50)`         | Nullable. USDA FoodData Central ID for nutrient lookup                        |
| `notes`         | `TEXT`                | Optional, for any additional details like common refuse percentage, etc.      |
| `created_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |
| `updated_at`    | `TIMESTAMP`           | `NOT NULL`                                                                    |

**Active Record Associations:**
```ruby
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients
  has_many :shopping_list_items
  # Optional: has_many :ingredient_nutrition_facts (if storing nutrition per ingredient directly)
end
```

### 12. `recipe_ingredients` table (Join Table)
Links recipes to their ingredients, including quantity and units.

| Column Name       | Data Type             | Constraints & Notes                                                                 |
|-------------------|-----------------------|-------------------------------------------------------------------------------------|
| `id`              | `BIGSERIAL PRIMARY KEY` |                                                                                     |
| `recipe_id`       | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                |
| `ingredient_id`   | `BIGINT`              | `REFERENCES ingredients(id)`, `NOT NULL`                                            |
| `quantity`        | `DECIMAL(10, 4)`      | `NOT NULL`. e.g., 0.5, 2.0.                                                         |
| `unit`            | `VARCHAR(50)`         | `NOT NULL`. e.g., "cup", "tbsp", "g", "oz", "piece"                                 |
| `notes`           | `VARCHAR(255)`        | Optional: e.g., "chopped", "to taste"                                               |
| `order_in_recipe` | `INTEGER`             | Optional, for displaying ingredients in a specific order                              |
| `created_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
| `updated_at`      | `TIMESTAMP`           | `NOT NULL`                                                                          |
|                   |                       | `UNIQUE (recipe_id, ingredient_id, notes)` (Notes might differentiate "1 apple" vs "1 apple, cored") |

**Active Record Associations:**
```ruby
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient
end
```

### Dietary Reference Intake (DRI) Core Data Tables

### 13. `nutrients` table
Stores information about each nutrient or dietary component relevant for DRIs and recipe analysis.

| Column Name                       | Data Type             | Constraints & Notes                                                                                                                               |
|-----------------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                              | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                   |
| `name`                            | `VARCHAR(255)`        | `NOT NULL` (User-facing name, e.g., "Vitamin A", "Added Sugars (% kcal)")                                                                          |
| `dri_identifier`                  | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL`. Stable internal key for DRI lookups (e.g., "VITAMIN_A_RAE", "ADDED_SUGARS_PERCENT_KCAL", "PROTEIN_G"). Indexed.               |
| `nutrient_category`               | `VARCHAR(100)`        | `NOT NULL` (e.g., 'Macronutrient', 'Vitamin', 'Mineral', 'Essential Fatty Acid', 'Dietary Guideline', 'Energy Component' - Rails enum)              |
| `default_dri_unit`                | `VARCHAR(50)`         | `NOT NULL` (Unit used in DRI tables: "g", "mg", "mcg", "mcg RAE", "mg AT", "IU", "% kcal", "L", " g/1000kcal ")                                    |
| `recipe_analysis_unit`            | `VARCHAR(50)`         | `NOT NULL` (Unit typically from food composition DBs for this nutrient: "g", "mg", "mcg", "IU")                                                      |
| `conversion_factor_to_dri_unit`   | `DECIMAL(12, 6)`      | Nullable. Factor to multiply recipe_analysis_unit value to get default_dri_unit value (e.g., for Vit D IU to mcg is 0.025).                          |
| `description`                     | `TEXT`                | Optional: Notes about the nutrient, RAE/DFE/NE definitions, how it's measured.                                                                      |
| `sort_order`                      | `INTEGER`             | Optional. For consistent display order in UI.                                                                                                     |
| `created_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |
| `updated_at`                      | `TIMESTAMP`           | `NOT NULL`                                                                                                                                        |

**Active Record Associations:**
```ruby
class Nutrient < ApplicationRecord
  has_many :recipe_nutrition_items # Links to nutritional content of recipes
  has_many :dri_values             # Links to DRI reference values
end
```

### 14. `recipe_nutrition_items` table (Join Table)
Stores the value of a specific nutrient for a recipe.

| Column Name         | Data Type             | Constraints & Notes                                                                       |
|---------------------|-----------------------|-------------------------------------------------------------------------------------------|
| `id`                | `BIGSERIAL PRIMARY KEY` |                                                                                           |
| `recipe_id`         | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                                      |
| `nutrient_id`       | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                    |
| `value_per_recipe`  | `DECIMAL(12, 5)`      | `NOT NULL`. Total amount of the nutrient in the whole recipe.                             |
| `unit`              | `VARCHAR(50)`         | `NOT NULL`. Unit for value_per_recipe (should match `nutrients.recipe_analysis_unit`).      |
| `value_per_serving` | `DECIMAL(12, 5)`      | `NOT NULL`. Calculated: `value_per_recipe` / `recipe.number_of_servings`.                   |
| `created_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
| `updated_at`        | `TIMESTAMP`           | `NOT NULL`                                                                                |
|                     |                       | `UNIQUE (recipe_id, nutrient_id)`                                                         |

**Active Record Associations:**
```ruby
class RecipeNutritionItem < ApplicationRecord
  belongs_to :recipe
  belongs_to :nutrient
end
```

### 15. `life_stage_groups` table
Defines the specific demographic groups for which DRIs are set.

| Column Name        | Data Type             | Constraints & Notes                                                                                                       |
|--------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                                                                                                                           |
| `name`             | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL` (e.g., "Infants 6-11 months", "Females 19-30 years", "Pregnancy 19-30 years T2", "Males 2-3 years") |
| `min_age_months`   | `INTEGER`             | `NOT NULL` (Lower bound of age in months, inclusive)                                                                      |
| `max_age_months`   | `INTEGER`             | `NOT NULL` (Upper bound of age in months, inclusive, e.g., 11 for 0-11m, 47 for 2-3y, 9999 for 71+y)                      |
| `sex`              | `VARCHAR(20)`         | `NOT NULL` ('male', 'female', 'all' - Rails enum)                                                                      |
| `special_condition`| `VARCHAR(50)`         | Nullable ('pregnancy', 'lactation' - Rails enum)                                                                       |
| `trimester`        | `INTEGER`             | Nullable. 1, 2, or 3 (if `special_condition` is 'pregnancy')                                                             |
| `lactation_period` | `VARCHAR(20)`         | Nullable. '0-6 months', '7-12 months' (if `special_condition` is 'lactation' - Rails enum)                             |
| `notes`            | `TEXT`                | Optional. For specific descriptions from DRI tables (e.g., from "Life Stage Group" column of S-tables).                   |
| `created_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |
| `updated_at`       | `TIMESTAMP`           | `NOT NULL`                                                                                                                |

**Active Record Associations:**
```ruby
class LifeStageGroup < ApplicationRecord
  has_many :dri_values
  has_many :reference_anthropometries
  has_many :eer_profiles # EER profiles can be specific to life stages
  has_many :eer_additive_components # Additive components can be specific to life stages
  has_many :pal_definitions # PAL definitions can be specific to life stages
end
```

### 16. `dri_values` table
Stores the actual DRI values, linking nutrients and life_stage_groups.

| Column Name                 | Data Type             | Constraints & Notes                                                                                                                                                                                                                            |
|-----------------------------|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                        | `BIGSERIAL PRIMARY KEY` |                                                                                                                                                                                                                                                |
| `nutrient_id`               | `BIGINT`              | `REFERENCES nutrients(id)`, `NOT NULL`                                                                                                                                                                                                         |
| `life_stage_group_id`       | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`                                                                                                                                                                                                 |
| `dri_type`                  | `VARCHAR(50)`         | `NOT NULL` ('EAR', 'RDA', 'AI', 'UL', 'AMDR_min_percent', 'AMDR_max_percent', 'CDRR', 'Guideline_absolute', 'Guideline_g_per_1000kcal', 'Guideline_percent_kcal_limit' - Rails enum)                                                 |
| `value_numeric`             | `DECIMAL(12, 5)`      | Nullable. For quantitative DRI values.                                                                                                                                                                                                         |
| `value_string`              | `VARCHAR(50)`         | Nullable. For "n/a", "<10", "ND", or other textual DRI outcomes.                                                                                                                                                                              |
| `unit`                      | `VARCHAR(50)`         | `NOT NULL`. Unit for `value_numeric` (e.g., "g", "mg", "% kcal"). Should match `nutrients.default_dri_unit` or be specific to the guideline type.                                                                                                 |
| `source_of_goal`            | `VARCHAR(100)`        | Nullable. From "Source of Goal" column in DGA tables (e.g., "RDA", "AI", "AMDR", "DGA", "14g/1,000 kcal").                                                                                                                                    |
| `criterion`                 | `TEXT`                | Nullable. Physiological basis for the DRI (from S-tables or other IOM reports).                                                                                                                                                              |
| `footnote_marker`           | `VARCHAR(10)`         | Nullable. e.g., "b", "c", "d" from source tables if a specific footnote applies directly.                                                                                                                                                      |
| `notes`                     | `TEXT`                | Nullable. For detailed footnotes or conditions (e.g., "UL applies to supplemental forms only", "n/a for this age group").                                                                                                                    |
| `source_document_reference` | `VARCHAR(255)`        | Nullable. To track which DRI report/table the value came from (e.g., "DGA App. A1-1", "IOM VitC S-1").                                                                                                                                         |
| `created_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
| `updated_at`                | `TIMESTAMP`           | `NOT NULL`                                                                                                                                                                                                                                     |
|                             |                       | `UNIQUE (nutrient_id, life_stage_group_id, dri_type)` (Needs careful consideration for uniqueness if a nutrient can have multiple values of the same type for the same group under slightly different uncaptured conditions. Postgres handles NULLs as distinct in unique constraints by default.) |

**Active Record Associations:**
```ruby
class DriValue < ApplicationRecord
  belongs_to :nutrient
  belongs_to :life_stage_group
end
```
Meal Planning and Shopping (Tables 17-20)

### 17. `meal_plan_entries` table
Connects users, recipes, dates, and meal types (for calendar).

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `user_id`                  | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                                       |
| `recipe_id`                | `BIGINT`              | `REFERENCES recipes(id)`, `NOT NULL`                                      |
| `date`                     | `DATE`                | `NOT NULL`                                                                 |
| `meal_type`                | `VARCHAR(50)`         | `NOT NULL` (e.g., 'breakfast', 'lunch', 'dinner', 'snack' - Rails enum)      |
| `is_prepped`               | `BOOLEAN`             | `NOT NULL`, Default `false`                                                |
| `notes`                    | `TEXT`                | Optional user notes for this specific meal instance                          |
| `number_of_servings_planned`| `INTEGER`             | Nullable, default to recipe.number_of_servings. Allows user to plan for more/less. |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `UNIQUE (user_id, date, meal_type)` (Consider if multiple same meal_type entries per day are allowed)

**Active Record Associations:**
```ruby
class MealPlanEntry < ApplicationRecord
  belongs_to :user
  belongs_to :recipe
end
```

### 18. `shopping_lists` table
Stores shopping lists, typically one active per user or per "trip".

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `user_id`                  | `BIGINT`              | `REFERENCES users(id)`, `NOT NULL`                                       |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`, Default 'My Shopping List'                                     |
| `status`                   | `VARCHAR(50)`         | `NOT NULL`, Default 'active'. e.g., 'active', 'archived', 'completed' (Rails enum) |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class ShoppingList < ApplicationRecord
  belongs_to :user
  has_many :shopping_list_items, dependent: :destroy # If list deleted, items are too
  has_many :ingredients, through: :shopping_list_items
end
```

### 19. `shopping_list_items` table
Items within a shopping list.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `shopping_list_id`         | `BIGINT`              | `REFERENCES shopping_lists(id)`, `NOT NULL`                                |
| `ingredient_id`            | `BIGINT`              | Nullable, `REFERENCES ingredients(id) (use if linked to a known ingredient) |
| `custom_item_name`         | `VARCHAR(255)`        | If `ingredient_id` is NULL, for one-off items                                |
| `quantity_description`     | `VARCHAR(100)`        | e.g., "2 large", "1 cup", "100g" (User-friendly display)                      |
| `quantity_numeric`         | `DECIMAL(10,4)`       | Nullable. Structured quantity for potential aggregation.                        |
| `unit`                     | `VARCHAR(50)`         | Nullable. Unit for `quantity_numeric`.                                         |
| `is_checked`               | `BOOLEAN`             | `NOT NULL`, Default `false`                                                 |
| `preferred_store_id`       | `BIGINT`              | Optional, `REFERENCES stores(id)                                              |
| `notes`                    | `VARCHAR(255)`        | e.g., "organic if possible"                                                   |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                  |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                  |

**Active Record Associations:**
```ruby
class ShoppingListItem < ApplicationRecord
  belongs_to :shopping_list
  belongs_to :ingredient, optional: true
  belongs_to :store, foreign_key: 'preferred_store_id', optional: true
end
```

### 20. `stores` table
Information about grocery stores.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`                                                                 |
| `address`                  | `VARCHAR(255)`        | Optional                                                                   |
| `zip_code`                 | `VARCHAR(20)`         | Optional, for local search                                                |
| `store_type`               | `VARCHAR(50)`         | e.g., 'grocer', 'supermarket', 'online' (Rails enum)                        |
| `logo_url`                 | `VARCHAR(255)`        |                                                                          |
| `opening_hours`            | `VARCHAR(255)`        | e.g., "Open until 9 PM", "Open 24 Hours"                                      |
| `delivery_info`            | `VARCHAR(255)`        | e.g., "Ships in 2-3 days" (for online)                                         |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class Store < ApplicationRecord
  # has_many :user_preferred_stores
  # has_many :users, through: :user_preferred_stores
  has_many :shopping_list_items, foreign_key: 'preferred_store_id'
end
```

### Dietary Pattern Data (Tables 21-24)

### 21. `dietary_patterns` table
Stores information about different dietary patterns.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `UNIQUE`, `NOT NULL` (e.g., "Healthy U.S.-Style Ages 2+")                     |
| `description`              | `TEXT`                | Optional, further details about the pattern                                |
| `source_document_reference`| `VARCHAR(255)`        | e.g., "Nutrition_data.md - Table A3-2"                                         |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class DietaryPattern < ApplicationRecord
  has_many :dietary_pattern_calorie_levels, dependent: :destroy
end
```

### 22. `food_groups` table
Stores predefined food groups and subgroups relevant to dietary patterns.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(100)`        | `UNIQUE`, `NOT NULL` (e.g., "Vegetables", "Dark-Green Vegetables")            |
| `parent_food_group_id`      | `BIGINT`              | Nullable, `REFERENCES food_groups(id)` (for subgroups)                        |
| `recommended_unit`         | `VARCHAR(50)`         | `NOT NULL` (e.g., "cup eq", "ounce eq", "grams")                                |
| `notes`                    | `TEXT`                | Optional, e.g., "Includes cooked dry beans, peas, lentils"                        |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class FoodGroup < ApplicationRecord
  belongs_to :parent_food_group, class_name: 'FoodGroup', foreign_key: 'parent_food_group_id', optional: true
  has_many :child_food_groups, class_name: 'FoodGroup', foreign_key: 'parent_food_group_id', dependent: :destroy
  has_many :dietary_pattern_food_group_recommendations, dependent: :destroy
end
```

### 23. `dietary_pattern_calorie_levels` table
Defines specific calorie levels within a dietary pattern.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `dietary_pattern_id`      | `BIGINT`              | `REFERENCES dietary_patterns(id)`, `NOT NULL`                                |
| `calorie_level`            | `INTEGER`             | `NOT NULL` (e.g., 700, 1000, 1200, 2000)                                        |
| `limit_on_calories_other_uses_kcal_day`| `INTEGER` | Nullable, from USDA pattern tables                                        |
| `limit_on_calories_other_uses_percent_day`| `DECIMAL(5, 2)` | Nullable, from USDA pattern tables                                        |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `UNIQUE (dietary_pattern_id, calorie_level)`

**Active Record Associations:**
```ruby
class DietaryPatternCalorieLevel < ApplicationRecord
  belongs_to :dietary_pattern
  has_many :dietary_pattern_food_group_recommendations, dependent: :destroy
end
```

### 24. `dietary_pattern_food_group_recommendations` table
Stores recommended amounts of food groups for a specific dietary pattern and calorie level.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `dietary_pattern_calorie_level_id`| `BIGINT` | `REFERENCES dietary_pattern_calorie_levels(id)`, `NOT NULL`                |
| `food_group_id`            | `BIGINT`              | `REFERENCES food_groups(id)`, `NOT NULL`                                    |
| `amount_value`             | `DECIMAL(10, 4)`      | `NOT NULL` (e.g., 0.66, 1.0, 2.5)                                                |
| `amount_frequency`         | `VARCHAR(10)`         | `NOT NULL` (e.g., 'daily', 'weekly' - Rails enum)                                |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `UNIQUE (dietary_pattern_calorie_level_id, food_group_id)`

**Active Record Associations:**
```ruby
class DietaryPatternFoodGroupRecommendation < ApplicationRecord
  belongs_to :dietary_pattern_calorie_level
  belongs_to :food_group
end
```

### Estimated Energy Requirement (EER) Equation & Reference Data (Tables 25-28)

### 25. `eer_profiles` table
Stores coefficients and basic parameters for TEE/EER predictive equations based on distinct population profiles.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `name`                     | `VARCHAR(255)`        | `NOT NULL`, `UNIQUE` (e.g., "Men 19+ Inactive TEE", "Girls 0-2y TEE", "Pregnancy T2 Active") |
| `source_table_reference`   | `VARCHAR(100)`        | e.g., "Nutrition_data.md - Table S-7", "IOM Energy S-4 Preg"                        |
| `life_stage_group_id`      | `BIGINT`              | Nullable, `REFERENCES life_stage_groups(id)`. Links to a broad life stage if applicable. |
| `sex_filter`               | `VARCHAR(20)`         | `NOT NULL` ('male', 'female', 'all' - Rails enum). For which sex this profile applies. |
| `age_min_months_filter`    | `INTEGER`             | Nullable. Lower bound of age applicability for this specific profile.                |
| `age_max_months_filter`    | `INTEGER`             | Nullable. Upper bound of age applicability for this specific profile.                |
| `pal_category_applicable`  | `VARCHAR(50)`         | Nullable. (e.g., 'sedentary', 'low_active', 'active', 'very_active' - Rails enum). PAL Category this equation applies to. |
| `coefficient_intercept`    | `DECIMAL(12, 4)`      |                                                                          |
| `coefficient_age_years`    | `DECIMAL(12, 4)`      | Nullable. Coeff for age if input to formula is in years.                            |
| `coefficient_age_months`   | `DECIMAL(12, 4)`      | Nullable. Coeff for age if input to formula is in months.                           |
| `coefficient_height_cm`    | `DECIMAL(12, 4)`      | Coefficient for height in centimeters.                                             |
| `coefficient_weight_kg`    | `DECIMAL(12, 4)`      | Coefficient for weight in kilograms.                                                |
| `coefficient_pal_value`    | `DECIMAL(12,4)`        | Nullable. If PAL value is a direct multiplier in the equation.                        |
| `coefficient_gestation_weeks`| `DECIMAL(12, 4)` | Nullable. For pregnancy equations.                                                   |
| `equation_basis`           | `VARCHAR(50)`         | `NOT NULL` (e.g., 'TEE', 'EER_direct' - Rails enum). Indicates if this profile calculates TEE (before additions) or a direct EER. |
| `standard_error_of_predicted_value_kcal`| `DECIMAL(10,2)` | Nullable. SEPV (RMSE) from Table 5-8 for prediction intervals.                          |
| `notes`                    | `TEXT`                | e.g., Specific formula structure if not standard; which variables (age, height, weight, pal) are used by this equation. |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class EerProfile < ApplicationRecord
  belongs_to :life_stage_group, optional: true
end
```

### 26. `eer_additive_components` table
Stores values for components added to TEE to calculate final EER (e.g., energy for growth, pregnancy deposition, lactation).

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `component_type`           | `VARCHAR(50)`         | `NOT NULL` ('growth', 'pregnancy_deposition', 'lactation_milk_production', 'lactation_mobilization' - Rails enum) |
| `source_table_reference`   | `VARCHAR(100)`        | e.g., "Nutrition_data.md - Table S-8", "IOM Energy S-4 Preg", "DGA A2-3"                        |
| `life_stage_group_id`      | `BIGINT`              | Nullable, `REFERENCES life_stage_groups(id)`. Filters applicability.                        |
| `sex_filter`               | `VARCHAR(20)`         | Nullable. ('male', 'female', 'all' - Rails enum).                                            |
| `age_min_months_filter`    | `INTEGER`             | Nullable. Lower bound of age for this component.                                            |
| `age_max_months_filter`    | `INTEGER`             | Nullable. Upper bound of age for this component.                                            |
| `condition_pregnancy_trimester_filter`| `INTEGER` | Nullable. (1, 2, 3)                                                                 |
| `condition_pre_pregnancy_bmi_category_filter`| `VARCHAR(20)` | Nullable. ('underweight', 'normal', 'overweight', 'obese' - Rails enum)                        |
| `condition_lactation_period_filter`| `VARCHAR(20)` | Nullable. ('0-6 months', '7-12 months' - Rails enum)                                            |
| `value_kcal_day`            | `INTEGER`             | `NOT NULL`. The energy value in kcal/day for this component.                                    |
| `notes`                    | `TEXT`                | e.g., "For normal weight prepregnancy", "Energy cost for boys 3y"                                |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `UNIQUE (component_type, life_stage_group_id, sex_filter, age_min_months_filter, age_max_months_filter, condition_pregnancy_trimester_filter, condition_pre_pregnancy_bmi_category_filter, condition_lactation_period_filter)` (handle NULLs appropriately for uniqueness)

**Active Record Associations:**
```ruby
class EerAdditiveComponent < ApplicationRecord
  belongs_to :life_stage_group, optional: true
end
```

### 27. `reference_anthropometries` table
Stores reference heights and weights.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `life_stage_group_id`      | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`, `UNIQUE` (One ref set per specific life stage group) |
| `reference_height_cm`      | `DECIMAL(5, 1)`        | Nullable.                                                                  |
| `reference_weight_kg`      | `DECIMAL(5, 2)`        | Nullable.                                                                  |
| `median_bmi`               | `DECIMAL(4, 1)`        | Nullable.                                                                  |
| `source_document_reference`| `VARCHAR(255)`        | e.g., "IOM Elements Table 1-2", "User Doc Table 1-2"                                |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class ReferenceAnthropometry < ApplicationRecord
  belongs_to :life_stage_group
end
```

### 28. `growth_factors` table
Stores growth factors for extrapolating child DRIs.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `life_stage_group_id`      | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`, `UNIQUE` (One growth factor per specific child life stage group) |
| `factor_value`             | `DECIMAL(5, 2)`        | `NOT NULL`                                                                 |
| `source_document_reference`| `VARCHAR(255)`        | e.g., "IOM Elements Table 2-1"                                                |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |

**Active Record Associations:**
```ruby
class GrowthFactor < ApplicationRecord
  belongs_to :life_stage_group
end
```

### 29. `pal_definitions` table
Stores PAL category definitions, ranges, and potentially specific coefficients.

| Column Name                | Data Type             | Constraints & Notes                                                      |
|----------------------------|-----------------------|--------------------------------------------------------------------------|
| `id`                       | `BIGSERIAL PRIMARY KEY` |                                                                          |
| `life_stage_group_id`      | `BIGINT`              | `REFERENCES life_stage_groups(id)`, `NOT NULL`. Links PAL definition to a broad life stage. |
| `pal_category`             | `VARCHAR(50)`         | `NOT NULL` ('sedentary', 'low_active', 'active', 'very_active' - Rails enum). |
| `pal_range_min_value`      | `DECIMAL(4,2)`         | `NOT NULL` (e.g., 1.00 for Sedentary from Table 5-4).                                |
| `pal_range_max_value`      | `DECIMAL(4,2)`         | `NOT NULL` (e.g., 1.30 for PAL <1.31 for Sedentary for 3-8.99y from Table 5-4).            |
| `percentile_value`         | `INTEGER`             | Nullable. If this entry represents a specific percentile from Table 5-3 (e.g., 10, 25, 50). |
| `pal_value_at_percentile`  | `DECIMAL(4,2)`         | Nullable. The PAL value corresponding to percentile_value (from Table 5-3).                |
| `coefficient_for_eer_equation`| `DECIMAL(4,2)` | Nullable. If a PAL category itself has a fixed coefficient used in some EER equations. |
| `source_document_reference`| `VARCHAR(255)`        | e.g., "Nutrition_data.md - Table 5-3", "Nutrition_data.md - Table 5-4"                        |
| `created_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `updated_at`               | `TIMESTAMP`           | `NOT NULL`                                                                 |
| `UNIQUE (life_stage_group_id, pal_category)` (If defining ranges per category) OR `UNIQUE (life_stage_group_id, percentile_value)` (if storing percentile data) - choose primary unique constraint based on main use.

**Active Record Associations:**
```ruby
class PalDefinition < ApplicationRecord
  belongs_to :life_stage_group
end
```

### Considerations / Future Tables:

### tags / recipe_tags: For categorizing recipes (e.g., 'vegan', 'quick-meal', 'high-protein').

### user_favorite_recipes: If users can "favorite" recipes outside of a meal plan.

### user_preferred_stores: Join table if users can mark preferred stores.

### promotions / deals: If integrating local store deals.

### product_prices_at_stores: For more granular price tracking.

### avatars: Predefined avatar options if not just using users.avatar_url.

### household_members: For inviting others to a shared plan.

### dri_source_documents: A table to list source documents (IOM reports, DGA versions) and link dri_values, eer_profiles etc. to them for full traceability (your source_document_reference columns are a good step towards this).