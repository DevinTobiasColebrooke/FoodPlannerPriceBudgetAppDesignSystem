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
| `main_goal`                     | `VARCHAR(100)`           | e.g., 'save_money', 'eat_healthier'                  |
| `side_goals`                    | `TEXT[]`                 | Array of strings, e.g., {'reduce_waste', 'new_recipes'} |
| `household_size`                | `VARCHAR(50)`            | e.g., '1-4', '5-10'                                  |
| `cooking_time_preference`       | `VARCHAR(50)`            | e.g., 'quick', 'moderate', 'leisurely'               |
| `meal_difficulty_preference`    | `VARCHAR(50)`            | e.g., 'easy', 'medium', 'involved'                   |
| `shopping_difficulty_preference`| `VARCHAR(50)`            | e.g., 'convenient', 'cheapest', 'balanced'           |
| `weekly_budget`                 | `DECIMAL(10, 2)`         | Optional                                             |
| `flexible_budget`               | `BOOLEAN`                | Default `true`                                       |
| `location_preference_type`      | `VARCHAR(50)`            | e.g., 'auto', 'region', `null`                       |
| `zip_code`                      | `VARCHAR(20)`            | If location_preference_type is 'region'              |
| `age`                           | `INTEGER`                | Required for DRI calculations                        |
| `sex`                           | `VARCHAR(10)`            | e.g., 'male', 'female'. Required for DRI calculations |
| `height_cm`                     | `DECIMAL(5, 2)`          | Store in a consistent unit (cm). Required for DRI calculations |
| `weight_kg`                     | `DECIMAL(5, 2)`          | Store in a consistent unit (kg). Required for DRI calculations |
| `physical_activity_level`       | `VARCHAR(50)`            | e.g., 'sedentary', 'low_active', 'active', 'very_active'. Required for EER |
| `is_pregnant`                   | `BOOLEAN`                | Default false. Required for DRI                      |
| `pregnancy_trimester`           | `INTEGER`                | Nullable. 1, 2, or 3. Required if is_pregnant is true |
| `is_lactating`                  | `BOOLEAN`                | Default false. Required for DRI                      |
| `lactation_months_postpartum`   | `INTEGER`                | Nullable. e.g., 0-6, 7-12. Required if is_lactating is true |
| `created_at`                    | `TIMESTAMP`              | `NOT NULL`                                           |
| `updated_at`                    | `TIMESTAMP`              | `NOT NULL`                                           |

**Active Record Associations:**
```ruby
class User < ApplicationRecord
  has_many :user_allergies
  has_many :allergies, through: :user_allergies
  has_many :user_kitchen_equipments
  has_many :kitchen_equipments, through: :user_kitchen_equipments
  has_many :meal_plan_entries
  has_many :recipes, through: :meal_plan_entries
  has_many :shopping_lists
  has_many :user_dietary_restrictions # New
  has_many :dietary_restrictions, through: :user_dietary_restrictions # New
  # Potentially: has_many :user_preferred_stores
  # Potentially: has_one :user_nutrient_profile (to store their calculated DRIs)
end
```

---

### 2. `allergies` table

Stores predefined common allergies.

| Column Name  | Data Type               | Constraints & Notes |
|--------------|-------------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`          | `UNIQUE`, `NOT NULL`  |
| `created_at` | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class Allergy < ApplicationRecord
  has_many :user_allergies
  has_many :users, through: :user_allergies
end
```

---

### 3. `user_allergies` table (Join Table)

Links users to their allergies.

| Column Name  | Data Type         | Constraints & Notes                              |
|--------------|-------------------|--------------------------------------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                                                  |
| `user_id`    | `BIGINT`          | `REFERENCES users(id)`, `NOT NULL`                 |
| `allergy_id` | `BIGINT`          | `REFERENCES allergies(id)`, `NOT NULL`             |
| `created_at` | `TIMESTAMP`       | `NOT NULL`                                       |
| `updated_at` | `TIMESTAMP`       | `NOT NULL`                                       |
|              |                   | `UNIQUE (user_id, allergy_id)`                   |

**Active Record Associations:**
```ruby
class UserAllergy < ApplicationRecord
  belongs_to :user
  belongs_to :allergy
end
```

### 4. `dietary_restrictions` table

Stores predefined dietary restrictions (e.g., vegan, vegetarian, gluten-free).

| Column Name  | Data Type               | Constraints & Notes |
|--------------|-------------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`          | `UNIQUE`, `NOT NULL`  |
| `description`| `TEXT`                  | Optional, further details about the restriction |
| `created_at` | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class DietaryRestriction < ApplicationRecord
  has_many :user_dietary_restrictions
  has_many :users, through: :user_dietary_restrictions
  # has_many :recipe_dietary_restrictions (if you tag recipes with these)
  # has_many :recipes, through: :recipe_dietary_restrictions
end
```

---

### 5. `user_dietary_restrictions` table (Join Table)

Links users to their dietary restrictions.

| Column Name              | Data Type         | Constraints & Notes                              |
|--------------------------|-------------------|--------------------------------------------------|
| `id`                     | `BIGSERIAL PRIMARY KEY` |                                                  |
| `user_id`                | `BIGINT`          | `REFERENCES users(id)`, `NOT NULL`                 |
| `dietary_restriction_id` | `BIGINT`          | `REFERENCES dietary_restrictions(id)`, `NOT NULL`  |
| `created_at`             | `TIMESTAMP`       | `NOT NULL`                                       |
| `updated_at`             | `TIMESTAMP`       | `NOT NULL`                                       |
|                          |                   | `UNIQUE (user_id, dietary_restriction_id)`        |

**Active Record Associations:**
```ruby
class UserDietaryRestriction < ApplicationRecord
  belongs_to :user
  belongs_to :dietary_restriction
end
```

---

### 6. `kitchen_equipments` table

Stores predefined kitchen equipment.

| Column Name  | Data Type               | Constraints & Notes |
|--------------|-------------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`          | `UNIQUE`, `NOT NULL`  |
| `created_at` | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class KitchenEquipment < ApplicationRecord
  has_many :user_kitchen_equipments
  has_many :users, through: :user_kitchen_equipments
end
```

---

### 7. `user_kitchen_equipments` table (Join Table)

Links users to the kitchen equipment they own.

| Column Name           | Data Type         | Constraints & Notes                                    |
|-----------------------|-------------------|--------------------------------------------------------|
| `id`                  | `BIGSERIAL PRIMARY KEY` |                                                        |
| `user_id`             | `BIGINT`          | `REFERENCES users(id)`, `NOT NULL`                       |
| `kitchen_equipment_id`| `BIGINT`          | `REFERENCES kitchen_equipments(id)`, `NOT NULL`        |
| `created_at`          | `TIMESTAMP`       | `NOT NULL`                                             |
| `updated_at`          | `TIMESTAMP`       | `NOT NULL`                                             |
|                       |                   | `UNIQUE (user_id, kitchen_equipment_id)`                 |


**Active Record Associations:**
```ruby
class UserKitchenEquipment < ApplicationRecord
  belongs_to :user
  belongs_to :kitchen_equipment
end
```

---

### 8. `recipes` table

Stores details for each recipe.

| Column Name         | Data Type               | Constraints & Notes                       |
|---------------------|-------------------------|-------------------------------------------|
| `id`                | `BIGSERIAL PRIMARY KEY` |                                           |
| `name`              | `VARCHAR(255)`          | `NOT NULL`                                |
| `description`       | `TEXT`                  |                                           |
| `image_url`         | `VARCHAR(255)`          |                                           |
| `prep_time_minutes` | `INTEGER`               |                                           |
| `cook_time_minutes` | `INTEGER`               |                                           |
| `total_time_minutes`| `INTEGER`               | Calculated or stored, useful for filtering |
| `instructions`      | `TEXT`                  |                                           |
| `serving_size`      | `VARCHAR(100)`          | e.g., "1 bowl", "2 servings"              |
| `number_of_servings`| `INTEGER`               | More structured for calculations. Default 1 |
| `source_name`       | `VARCHAR(255)`          | Optional: e.g., "USDA", "User Submitted"  |
| `source_url`        | `VARCHAR(255)`          | Optional                                  |
| `difficulty_level`  | `VARCHAR(50)`           | e.g., 'easy', 'medium', 'hard' (for filtering) |
| `creator_id`        | `BIGINT`                | Nullable, `REFERENCES users(id)`. If users can create recipes |
| `is_public`         | `BOOLEAN`               | Default false. If users can create private/public recipes |
| `created_at`        | `TIMESTAMP`             | `NOT NULL`                                |
| `updated_at`        | `TIMESTAMP`             | `NOT NULL`                                |

**Active Record Associations:**
```ruby
class Recipe < ApplicationRecord
  has_many :recipe_ingredients
  has_many :ingredients, through: :recipe_ingredients
  has_many :recipe_nutrition_items
  has_many :nutrition_items, through: :recipe_nutrition_items
  has_many :meal_plan_entries
  belongs_to :creator, class_name: 'User', foreign_key: 'creator_id', optional: true # New
  # has_many :recipe_tags
  # has_many :tags, through: :recipe_tags
  # has_many :recipe_dietary_restrictions # New
  # has_many :dietary_restrictions, through: :recipe_dietary_restrictions # New
end
```

---

### 9. `ingredients` table

Stores individual ingredients.

| Column Name    | Data Type               | Constraints & Notes |
|----------------|-------------------------|---------------------|
| `id`           | `BIGSERIAL PRIMARY KEY` |                     |
| `name`         | `VARCHAR(255)`          | `UNIQUE`, `NOT NULL`  |
| `category`     | `VARCHAR(100)`          | Optional: e.g., 'Dairy', 'Produce', 'Pantry' |
| `default_unit` | `VARCHAR(50)`           | Optional, e.g., 'g', 'ml', 'piece' (for shopping list) |
| `fdc_id`       | `VARCHAR(50)`           | Nullable. USDA FoodData Central ID for nutrient lookup |
| `created_at`   | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at`   | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients
  has_many :shopping_list_items
  # has_many :ingredient_nutrition_items (if storing nutrient data per ingredient)
end
```

---

### 10. `recipe_ingredients` table (Join Table)

Links recipes to their ingredients, including quantity and units.

| Column Name     | Data Type               | Constraints & Notes                         |
|-----------------|-------------------------|---------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                             |
| `recipe_id`     | `BIGINT`                | `REFERENCES recipes(id)`, `NOT NULL`        |
| `ingredient_id` | `BIGINT`                | `REFERENCES ingredients(id)`, `NOT NULL`    |
| `quantity`      | `DECIMAL(10, 4)`        | e.g., 0.5, 2.0. More structured than VARCHAR |
| `unit`          | `VARCHAR(50)`           | e.g., "cup", "tbsp", "g", "oz", "piece"     |
| `notes`         | `VARCHAR(255)`          | Optional: e.g., "chopped", "to taste"       |
| `order_in_recipe`| `INTEGER`               | Optional, for displaying ingredients in a specific order |
| `created_at`    | `TIMESTAMP`             | `NOT NULL`                                  |
| `updated_at`    | `TIMESTAMP`             | `NOT NULL`                                  |

**Active Record Associations:**
```ruby
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient
end
```

---

### 11. `nutrition_items` table

Stores types of nutritional information (e.g., Calories, Protein, Fat).
This is the table that will hold your DRI master list of nutrients.

| Column Name              | Data Type               | Constraints & Notes |
|--------------------------|-------------------------|---------------------|
| `id`                     | `BIGSERIAL PRIMARY KEY` |                     |
| `name`                   | `VARCHAR(100)`          | `UNIQUE`, `NOT NULL`  |
| `unit`                   | `VARCHAR(20)`           | `NOT NULL`          |
| `dri_nutrient_identifier`| `VARCHAR(100)`          | Optional but useful: A consistent key to map to official DRI names, e.g., "VITAMINA_RAE" |
| `nutrient_category`      | `VARCHAR(50)`           | e.g., 'macronutrient', 'vitamin_fat_soluble', 'vitamin_water_soluble', 'mineral_major', 'mineral_trace' |
| `created_at`             | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at`             | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class NutritionItem < ApplicationRecord
  has_many :recipe_nutrition_items
  has_many :recipes, through: :recipe_nutrition_items
  # has_many :dri_values (This is where the actual DRI numbers would be stored per age/sex/etc.)
end
```

---

### 12. `recipe_nutrition_items` table (Join Table)

Stores the value of a specific nutritional item for a recipe.

| Column Name        | Data Type               | Constraints & Notes                               |
|--------------------|-------------------------|---------------------------------------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                                                   |
| `recipe_id`        | `BIGINT`                | `REFERENCES recipes(id)`, `NOT NULL`              |
| `nutrition_item_id`| `BIGINT`                | `REFERENCES nutrition_items(id)`, `NOT NULL`      |
| `value`            | `DECIMAL(10, 4)`        | Store as numeric for calculations and consistency |
| `value_per_serving`| `DECIMAL(10, 4)`        | Calculated or stored. value / recipe.number_of_servings |
| `created_at`       | `TIMESTAMP`             | `NOT NULL`                                        |
| `updated_at`       | `TIMESTAMP`             | `NOT NULL`                                        |
|                    |                         | `UNIQUE (recipe_id, nutrition_item_id)`           |

**Active Record Associations:**
```ruby
class RecipeNutritionItem < ApplicationRecord
  belongs_to :recipe
  belongs_to :nutrition_item
end
```

---

### 13. `meal_plan_entries` table

Connects users, recipes, dates, and meal types (for calendar).

| Column Name              | Data Type               | Constraints & Notes                                |
|--------------------------|-------------------------|----------------------------------------------------|
| `id`                     | `BIGSERIAL PRIMARY KEY` |                                                    |
| `user_id`                | `BIGINT`                | `REFERENCES users(id)`, `NOT NULL`                 |
| `recipe_id`              | `BIGINT`                | `REFERENCES recipes(id)`, `NOT NULL`               |
| `date`                   | `DATE`                  | `NOT NULL`                                         |
| `meal_type`              | `VARCHAR(50)`           | `NOT NULL` (e.g., 'breakfast', 'lunch', 'dinner', 'snack') |
| `is_prepped`             | `BOOLEAN`               | Default `false`                                    |
| `notes`                  | `TEXT`                  | Optional user notes for this specific meal instance |
| `number_of_servings_planned` | `INTEGER`           | Optional, default to recipe.number_of_servings. Allows user to plan for more/less. |
| `created_at`             | `TIMESTAMP`             | `NOT NULL`                                         |
| `updated_at`             | `TIMESTAMP`             | `NOT NULL`                                         |
|                          |                         | `UNIQUE (user_id, date, meal_type)` (Or allow multiple snacks/meals of same type per day? If so, remove this unique constraint or add an order column) |

**Active Record Associations:**
```ruby
class MealPlanEntry < ApplicationRecord
  belongs_to :user
  belongs_to :recipe
end
```

---

### 14. `age_groups` table

Stores DRI-defined age and life stage groups.

| Column Name        | Data Type               | Constraints & Notes |
|--------------------|-------------------------|---------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                     |
| `name`             | `VARCHAR(100)`          | `UNIQUE`, `NOT NULL` (e.g., "Infants 0-6 mo", "Males 19-30 y", "Pregnancy 14-18 y") |
| `min_age_months`   | `INTEGER`               | `NOT NULL` (lower bound of age in months) |
| `max_age_months`   | `INTEGER`               | `NOT NULL` (upper bound of age in months) |
| `life_stage_type`  | `VARCHAR(50)`           | e.g., 'infant', 'child', 'adolescent', 'adult', 'pregnancy', 'lactation' |
| `created_at`       | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at`       | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class AgeGroup < ApplicationRecord
  has_many :dri_values
end
```

---

### 15. `dri_values` table

Stores the actual DRI values for each nutrient, age group, sex, and special life stage.

| Column Name                    | Data Type               | Constraints & Notes |
|--------------------------------|-------------------------|---------------------|
| `id`                           | `BIGSERIAL PRIMARY KEY` |                     |
| `nutrition_item_id`            | `BIGINT`                | `REFERENCES nutrition_items(id)`, `NOT NULL` |
| `age_group_id`                 | `BIGINT`                | `REFERENCES age_groups(id)`, `NOT NULL` |
| `sex`                          | `VARCHAR(10)`           | 'male', 'female', or 'all' (for non-sex-specific) |
| `special_life_stage_modifier`  | `VARCHAR(100)`          | Nullable (e.g., "Trimester 2", "Postpartum 0-6 months", "Smoker") |
| `ear_value`                    | `DECIMAL(10, 4)`        | Nullable (Estimated Average Requirement) |
| `rda_value`                    | `DECIMAL(10, 4)`        | Nullable (Recommended Dietary Allowance) |
| `ai_value`                     | `DECIMAL(10, 4)`        | Nullable (Adequate Intake) |
| `ul_value`                     | `DECIMAL(10, 4)`        | Nullable (Tolerable Upper Intake Level) |
| `amdr_min_percent`             | `DECIMAL(5, 2)`         | Nullable (For macronutrients) |
| `amdr_max_percent`             | `DECIMAL(5, 2)`         | Nullable (For macronutrients) |
| `cdrr_value`                   | `DECIMAL(10,4)`         | Nullable (Chronic Disease Risk Reduction Intake, e.g. for Sodium) |
| `notes`                        | `TEXT`                  | For any specific conditions or footnotes from DRI tables |
| `source_document_reference`    | `VARCHAR(255)`          | Optional, to track which DRI report the value came from |
| `created_at`                   | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at`                   | `TIMESTAMP`             | `NOT NULL`          |
|                                |                         | `UNIQUE (nutrition_item_id, age_group_id, sex, special_life_stage_modifier)` |

**Active Record Associations:**
```ruby
class DriValue < ApplicationRecord
  belongs_to :nutrition_item
  belongs_to :age_group
end
```

---

### 16. `shopping_lists` table

Stores shopping lists, typically one active per user or per "trip".

| Column Name  | Data Type               | Constraints & Notes                     |
|--------------|-------------------------|-----------------------------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                                         |
| `user_id`    | `BIGINT`                | `REFERENCES users(id)`, `NOT NULL`      |
| `name`       | `VARCHAR(255)`          | Default 'My Shopping List', `NOT NULL`  |
| `status`     | `VARCHAR(50)`           | e.g., 'active', 'archived', 'completed' |
| `created_at` | `TIMESTAMP`             | `NOT NULL`                              |
| `updated_at` | `TIMESTAMP`             | `NOT NULL`                              |

**Active Record Associations:**
```ruby
class ShoppingList < ApplicationRecord
  belongs_to :user
  has_many :shopping_list_items
  has_many :ingredients, through: :shopping_list_items
end
```

---

### 17. `shopping_list_items` table

Items within a shopping list.

| Column Name      | Data Type               | Constraints & Notes                                     |
|------------------|-------------------------|---------------------------------------------------------|
| `id`             | `BIGSERIAL PRIMARY KEY` |                                                         |
| `shopping_list_id` | `BIGINT`              | `REFERENCES shopping_lists(id)`, `NOT NULL`             |
| `ingredient_id`  | `BIGINT`                | `REFERENCES ingredients(id)`, `NOT NULL` (or allow custom text item) |
| `custom_item_name`| `VARCHAR(255)`         | If `ingredient_id` is NULL                             |
| `quantity`       | `VARCHAR(100)`          | e.g., "2 large", "1 cup"                                |
| `is_checked`     | `BOOLEAN`               | Default `false`, `NOT NULL`                             |
| `preferred_store_id` | `BIGINT`            | Optional, `REFERENCES stores(id)`                       |
| `notes`          | `VARCHAR(255)`          | e.g., "organic if possible"                             |
| `created_at`     | `TIMESTAMP`             | `NOT NULL`                                              |
| `updated_at`     | `TIMESTAMP`             | `NOT NULL`                                              |

**Active Record Associations:**
```ruby
class ShoppingListItem < ApplicationRecord
  belongs_to :shopping_list
  belongs_to :ingredient, optional: true # If custom_item_name is used
  belongs_to :store, optional: true # For preferred_store_id
end
```

---

### 18. `stores` table

Information about grocery stores.

| Column Name      | Data Type               | Constraints & Notes                        |
|------------------|-------------------------|--------------------------------------------|
| `id`             | `BIGSERIAL PRIMARY KEY` |                                            |
| `name`           | `VARCHAR(255)`          | `NOT NULL`                                 |
| `address`        | `VARCHAR(255)`          | Optional                                   |
| `zip_code`       | `VARCHAR(20)`           | Optional, for local search                 |
| `store_type`     | `VARCHAR(50)`           | e.g., 'grocer', 'supermarket', 'online'    |
| `logo_url`       | `VARCHAR(255)`          |                                            |
| `opening_hours`  | `VARCHAR(255)`          | e.g., "Open until 9 PM", "Open 24 Hours"   |
| `delivery_info`  | `VARCHAR(255)`          | e.g., "Ships in 2-3 days" (for online)     |
| `created_at`     | `TIMESTAMP`             | `NOT NULL`                                 |
| `updated_at`     | `TIMESTAMP`             | `NOT NULL`                                 |

**Active Record Associations:**
```ruby
class Store < ApplicationRecord
  # has_many :user_preferred_stores
  # has_many :users, through: :user_preferred_stores
  has_many :shopping_list_items # If items can be sourced to a specific store on the list
end
```

---
### Considerations / Future Tables:

*   **`tags` / `recipe_tags`**: For categorizing recipes (e.g., 'vegan', 'quick-meal', 'high-protein').
*   **`user_favorite_recipes`**: If users can "favorite" recipes outside of a meal plan.
*   **`user_preferred_stores`**: Join table if users can mark preferred stores.
*   **`promotions` / `deals`**: If integrating local store deals.
*   **`product_prices_at_stores`**: For more granular price tracking.

This structure provides a comprehensive starting point for the application's data needs based on the provided HTML prototypes. 