# Recipe Model Implementation

## Model Definition

```ruby
# app/models/recipe.rb
class Recipe < ApplicationRecord
  belongs_to :creator, class_name: 'User', optional: true
  
  # Ingredients
  has_many :recipe_ingredients, dependent: :destroy
  has_many :ingredients, through: :recipe_ingredients
  
  # Nutrition
  has_many :recipe_nutrition_items, dependent: :destroy
  has_many :nutrients, through: :recipe_nutrition_items
  
  # Meal Planning
  has_many :meal_plan_entries, dependent: :destroy
  has_many :planned_users, through: :meal_plan_entries, source: :user

  # Validations
  validates :name, presence: true
  validates :number_of_servings, presence: true, numericality: { greater_than: 0 }
  validates :difficulty_level, inclusion: { in: %w[easy medium hard] }, allow_nil: true

  # Scopes
  scope :public_recipes, -> { where(is_public: true) }
  scope :by_difficulty, ->(level) { where(difficulty_level: level) }
  scope :quick_recipes, -> { where('prep_time_minutes <= ?', 30) }
  scope :recent, -> { order(created_at: :desc) }
end
```

## Related Models

### RecipeIngredient
```ruby
# app/models/recipe_ingredient.rb
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient
  
  validates :quantity, presence: true, numericality: { greater_than: 0 }
  validates :unit, presence: true
  validates :recipe_id, uniqueness: { scope: [:ingredient_id, :notes] }
end
```

### RecipeNutritionItem
```ruby
# app/models/recipe_nutrition_item.rb
class RecipeNutritionItem < ApplicationRecord
  belongs_to :recipe
  belongs_to :nutrient
  
  validates :value_per_recipe, presence: true, numericality: { greater_than_or_equal_to: 0 }
  validates :value_per_serving, presence: true, numericality: { greater_than_or_equal_to: 0 }
  validates :unit, presence: true
  validates :recipe_id, uniqueness: { scope: :nutrient_id }
end
```

### MealPlanEntry
```ruby
# app/models/meal_plan_entry.rb
class MealPlanEntry < ApplicationRecord
  belongs_to :user
  belongs_to :recipe
  
  validates :date, presence: true
  validates :meal_type, presence: true
  validates :user_id, uniqueness: { scope: [:date, :meal_type] }
  
  enum meal_type: {
    breakfast: 'breakfast',
    lunch: 'lunch',
    dinner: 'dinner',
    snack: 'snack'
  }
end
```

## Usage Examples

### Creating a Recipe
```ruby
recipe = Recipe.create!(
  name: 'Chicken Stir Fry',
  description: 'A quick and healthy stir fry',
  prep_time_minutes: 15,
  cook_time_minutes: 20,
  total_time_minutes: 35,
  instructions: '1. Cut chicken\n2. Stir fry vegetables\n3. Add sauce',
  number_of_servings: 4,
  difficulty_level: 'easy',
  is_public: true
)
```

### Adding Ingredients
```ruby
chicken = Ingredient.find_by(name: 'Chicken Breast')
rice = Ingredient.find_by(name: 'Brown Rice')

recipe.recipe_ingredients.create!(
  ingredient: chicken,
  quantity: 500,
  unit: 'g',
  notes: 'cut into strips'
)

recipe.recipe_ingredients.create!(
  ingredient: rice,
  quantity: 2,
  unit: 'cups',
  notes: 'cooked'
)
```

### Adding Nutrition Information
```ruby
protein = Nutrient.find_by(dri_identifier: 'PROCNT')
recipe.recipe_nutrition_items.create!(
  nutrient: protein,
  value_per_recipe: 120,
  unit: 'g',
  value_per_serving: 30
)
```

### Planning a Meal
```ruby
user.meal_plan_entries.create!(
  recipe: recipe,
  date: Date.today,
  meal_type: 'dinner',
  number_of_servings_planned: 2
)
```

### Finding Recipes
```ruby
# Find quick recipes
Recipe.quick_recipes

# Find public recipes by difficulty
Recipe.public_recipes.by_difficulty('easy')

# Find recipes with specific ingredient
Recipe.joins(:ingredients).where(ingredients: { name: 'Chicken' })

# Find recipes for meal planning
Recipe.joins(:meal_plan_entries)
      .where(meal_plan_entries: { user: current_user, date: Date.today })
``` 