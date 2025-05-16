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
  has_many :recipes, through: :meal_plan_entries # Or directly if users can save favorite recipes
  has_many :shopping_lists
  # Potentially: has_many :user_preferred_stores
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

---

### 4. `kitchen_equipments` table

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

### 5. `user_kitchen_equipments` table (Join Table)

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

### 6. `recipes` table

Stores details for each recipe.

| Column Name         | Data Type               | Constraints & Notes                       |
|---------------------|-------------------------|-------------------------------------------|
| `id`                | `BIGSERIAL PRIMARY KEY` |                                           |
| `name`              | `VARCHAR(255)`          | `NOT NULL`                                |
| `description`       | `TEXT`                  |                                           |
| `image_url`         | `VARCHAR(255)`          |                                           |
| `prep_time_minutes` | `INTEGER`               |                                           |
| `cook_time_minutes` | `INTEGER`               |                                           |
| `instructions`      | `TEXT`                  |                                           |
| `serving_size`      | `VARCHAR(100)`          | e.g., "1 bowl", "2 servings"              |
| `source_name`       | `VARCHAR(255)`          | Optional: e.g., "USDA", "User Submitted"  |
| `source_url`        | `VARCHAR(255)`          | Optional                                  |
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
  # has_many :users, through: :meal_plan_entries # If tracking who planned which recipe
  # has_many :recipe_tags
  # has_many :tags, through: :recipe_tags
end
```

---

### 7. `ingredients` table

Stores individual ingredients.

| Column Name  | Data Type               | Constraints & Notes |
|--------------|-------------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(255)`          | `UNIQUE`, `NOT NULL`  |
| `category`   | `VARCHAR(100)`          | Optional: e.g., 'Dairy', 'Produce', 'Pantry' |
| `created_at` | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients
  has_many :shopping_list_items
end
```

---

### 8. `recipe_ingredients` table (Join Table)

Links recipes to their ingredients, including quantity and units.

| Column Name     | Data Type               | Constraints & Notes                         |
|-----------------|-------------------------|---------------------------------------------|
| `id`            | `BIGSERIAL PRIMARY KEY` |                                             |
| `recipe_id`     | `BIGINT`                | `REFERENCES recipes(id)`, `NOT NULL`        |
| `ingredient_id` | `BIGINT`                | `REFERENCES ingredients(id)`, `NOT NULL`    |
| `quantity`      | `VARCHAR(50)`           | e.g., "1/2", "2"                            |
| `unit`          | `VARCHAR(50)`           | e.g., "cup", "tbsp", "large", "piece(s)"    |
| `notes`         | `VARCHAR(255)`          | Optional: e.g., "chopped", "to taste"       |
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

### 9. `nutrition_items` table

Stores types of nutritional information (e.g., Calories, Protein, Fat).

| Column Name  | Data Type               | Constraints & Notes |
|--------------|-------------------------|---------------------|
| `id`         | `BIGSERIAL PRIMARY KEY` |                     |
| `name`       | `VARCHAR(100)`          | `UNIQUE`, `NOT NULL`  | <!-- e.g., Calories, Protein, Fat, Carbs, Fiber -->
| `unit`       | `VARCHAR(20)`           | `NOT NULL`          | <!-- e.g., kcal, g, mg -->
| `created_at` | `TIMESTAMP`             | `NOT NULL`          |
| `updated_at` | `TIMESTAMP`             | `NOT NULL`          |

**Active Record Associations:**
```ruby
class NutritionItem < ApplicationRecord
  has_many :recipe_nutrition_items
  has_many :recipes, through: :recipe_nutrition_items
end
```

---

### 10. `recipe_nutrition_items` table (Join Table)

Stores the value of a specific nutritional item for a recipe.

| Column Name        | Data Type               | Constraints & Notes                               |
|--------------------|-------------------------|---------------------------------------------------|
| `id`               | `BIGSERIAL PRIMARY KEY` |                                                   |
| `recipe_id`        | `BIGINT`                | `REFERENCES recipes(id)`, `NOT NULL`              |
| `nutrition_item_id`| `BIGINT`                | `REFERENCES nutrition_items(id)`, `NOT NULL`      |
| `value`            | `VARCHAR(50)`           | Stored as VARCHAR to accommodate "~350", "10-12" etc. Needs parsing in app. |
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

### 11. `meal_plan_entries` table

Connects users, recipes, dates, and meal types (for calendar).

| Column Name   | Data Type               | Constraints & Notes                                |
|---------------|-------------------------|----------------------------------------------------|
| `id`          | `BIGSERIAL PRIMARY KEY` |                                                    |
| `user_id`     | `BIGINT`                | `REFERENCES users(id)`, `NOT NULL`                 |
| `recipe_id`   | `BIGINT`                | `REFERENCES recipes(id)`, `NOT NULL`               |
| `date`        | `DATE`                  | `NOT NULL`                                         |
| `meal_type`   | `VARCHAR(50)`           | `NOT NULL` (e.g., 'breakfast', 'lunch', 'dinner', 'snack') |
| `is_prepped`  | `BOOLEAN`               | Default `false`                                    |
| `notes`       | `TEXT`                  | Optional user notes for this specific meal instance |
| `created_at`  | `TIMESTAMP`             | `NOT NULL`                                         |
| `updated_at`  | `TIMESTAMP`             | `NOT NULL`                                         |
|               |                         | `UNIQUE (user_id, date, meal_type)` (Or allow multiple snacks/meals of same type per day?) |

**Active Record Associations:**
```ruby
class MealPlanEntry < ApplicationRecord
  belongs_to :user
  belongs_to :recipe
end
```

---

### 12. `shopping_lists` table

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

### 13. `shopping_list_items` table

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

### 14. `stores` table

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