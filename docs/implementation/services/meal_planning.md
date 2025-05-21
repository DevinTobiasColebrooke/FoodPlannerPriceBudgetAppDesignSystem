# Meal Planning Service Implementation

## Service Definition

```ruby
# lib/services/meal_planning_service.rb
class MealPlanningService
  def initialize(user, start_date, end_date)
    @user = user
    @start_date = start_date
    @end_date = end_date
    @nutrition_profile = user.nutrition_profile
  end

  def generate_plan
    validate_dates
    validate_user_profile
    
    plan = {
      daily_plans: generate_daily_plans,
      shopping_list: generate_shopping_list,
      nutrition_summary: calculate_nutrition_summary
    }
    
    save_plan(plan)
    plan
  end

  private

  def generate_daily_plans
    (@start_date..@end_date).map do |date|
      {
        date: date,
        meals: generate_meals_for_date(date),
        nutrition_targets: calculate_daily_targets,
        adjustments: calculate_adjustments(date)
      }
    end
  end

  def generate_meals_for_date(date)
    meal_types = %w[breakfast lunch dinner snacks]
    meal_types.map do |meal_type|
      {
        type: meal_type,
        recipes: select_recipes_for_meal(meal_type),
        timing: calculate_meal_timing(meal_type, date),
        nutrition: calculate_meal_nutrition(meal_type)
      }
    end
  end

  def select_recipes_for_meal(meal_type)
    recipes = Recipe.where(meal_type: meal_type)
      .where('portions >= ?', 1)
      .where('cost <= ?', calculate_meal_budget(meal_type))
    
    recipes = filter_by_preferences(recipes)
    recipes = filter_by_nutrition_needs(recipes, meal_type)
    
    select_optimal_recipes(recipes, meal_type)
  end

  def filter_by_preferences(recipes)
    recipes.reject do |recipe|
      recipe.ingredients.any? { |i| @user.dietary_restrictions.include?(i.allergen) }
    end
  end

  def filter_by_nutrition_needs(recipes, meal_type)
    target_nutrients = calculate_meal_nutrient_targets(meal_type)
    
    recipes.select do |recipe|
      recipe_nutrients = RecipeAnalysisService.new(recipe).analyze[:total_nutrients]
      meets_nutrient_targets?(recipe_nutrients, target_nutrients)
    end
  end

  def select_optimal_recipes(recipes, meal_type)
    recipes.sort_by do |recipe|
      score = calculate_recipe_score(recipe, meal_type)
      -score # Sort in descending order
    end.first(2) # Select top 2 recipes
  end

  def calculate_recipe_score(recipe, meal_type)
    nutrition_score = calculate_nutrition_score(recipe)
    cost_score = calculate_cost_score(recipe)
    preference_score = calculate_preference_score(recipe)
    variety_score = calculate_variety_score(recipe, meal_type)
    
    weights = {
      nutrition: 0.4,
      cost: 0.3,
      preference: 0.2,
      variety: 0.1
    }
    
    (nutrition_score * weights[:nutrition]) +
    (cost_score * weights[:cost]) +
    (preference_score * weights[:preference]) +
    (variety_score * weights[:variety])
  end

  def calculate_meal_timing(meal_type, date)
    base_times = {
      breakfast: '08:00',
      lunch: '12:00',
      dinner: '18:00',
      snacks: ['10:00', '15:00']
    }
    
    times = base_times[meal_type.to_sym]
    times.is_a?(Array) ? times : [times]
  end

  def calculate_meal_nutrition(meal_type)
    target_nutrients = calculate_meal_nutrient_targets(meal_type)
    {
      targets: target_nutrients,
      actual: calculate_actual_nutrition(meal_type)
    }
  end

  def calculate_meal_nutrient_targets(meal_type)
    daily_targets = calculate_daily_targets
    meal_distribution = {
      breakfast: 0.25,
      lunch: 0.35,
      dinner: 0.35,
      snacks: 0.05
    }
    
    daily_targets.transform_values do |value|
      value * meal_distribution[meal_type.to_sym]
    end
  end

  def calculate_daily_targets
    @nutrition_profile.daily_targets
  end

  def calculate_adjustments(date)
    previous_day = date - 1.day
    previous_nutrition = get_previous_day_nutrition(previous_day)
    
    {
      carryover: calculate_carryover(previous_nutrition),
      adjustments: calculate_nutrition_adjustments(previous_nutrition)
    }
  end

  def generate_shopping_list
    ingredients = collect_all_ingredients
    grouped_ingredients = group_ingredients(ingredients)
    
    {
      items: grouped_ingredients,
      total_cost: calculate_total_cost(grouped_ingredients),
      store_sections: organize_by_store_section(grouped_ingredients)
    }
  end

  def collect_all_ingredients
    all_recipes = get_all_selected_recipes
    all_recipes.flat_map do |recipe|
      recipe.ingredients.map do |ingredient|
        {
          name: ingredient.name,
          amount: ingredient.amount,
          unit: ingredient.unit,
          cost: ingredient.cost
        }
      end
    end
  end

  def group_ingredients(ingredients)
    ingredients.group_by { |i| i[:name] }.map do |name, items|
      {
        name: name,
        total_amount: sum_amounts(items),
        unit: items.first[:unit],
        total_cost: items.sum { |i| i[:cost] }
      }
    end
  end

  def calculate_nutrition_summary
    daily_plans = generate_daily_plans
    {
      daily_averages: calculate_daily_averages(daily_plans),
      weekly_totals: calculate_weekly_totals(daily_plans),
      target_compliance: calculate_target_compliance(daily_plans)
    }
  end

  def save_plan(plan)
    MealPlan.create!(
      user: @user,
      start_date: @start_date,
      end_date: @end_date,
      plan_data: plan,
      status: 'active'
    )
  end

  def validate_dates
    raise MealPlanningError, "Start date must be before end date" if @start_date > @end_date
    raise MealPlanningError, "Plan cannot exceed 7 days" if (@end_date - @start_date).to_i > 7
  end

  def validate_user_profile
    raise MealPlanningError, "User must have a nutrition profile" unless @nutrition_profile
    raise MealPlanningError, "User must have dietary preferences set" if @user.dietary_preferences.empty?
  end
end

class MealPlanningError < StandardError; end
```

## Usage

### Basic Meal Planning
```ruby
user = User.find(1)
start_date = Date.today
end_date = start_date + 6.days

plan = MealPlanningService.new(user, start_date, end_date).generate_plan

# Access plan components
daily_plans = plan[:daily_plans]
shopping_list = plan[:shopping_list]
nutrition_summary = plan[:nutrition_summary]
```

### Accessing Daily Plans
```ruby
daily_plans.each do |day|
  puts "Plan for #{day[:date]}"
  
  day[:meals].each do |meal|
    puts "#{meal[:type].capitalize}:"
    meal[:recipes].each do |recipe|
      puts "  - #{recipe.name}"
    end
  end
end
```

### Accessing Shopping List
```ruby
shopping_list[:items].each do |item|
  puts "#{item[:name]}: #{item[:total_amount]} #{item[:unit]}"
end

puts "Total cost: $#{shopping_list[:total_cost]}"
```

## Data Models

### MealPlan Model
```ruby
class MealPlan < ApplicationRecord
  belongs_to :user
  
  validates :start_date, presence: true
  validates :end_date, presence: true
  validates :plan_data, presence: true
  validates :status, presence: true
  
  enum status: {
    active: 0,
    completed: 1,
    cancelled: 2
  }
  
  def daily_plans
    plan_data[:daily_plans]
  end
  
  def shopping_list
    plan_data[:shopping_list]
  end
  
  def nutrition_summary
    plan_data[:nutrition_summary]
  end
end
```

### User Model Extensions
```ruby
class User < ApplicationRecord
  has_one :nutrition_profile
  has_many :meal_plans
  has_many :dietary_preferences
  has_many :dietary_restrictions
  
  def current_meal_plan
    meal_plans.active.first
  end
end
```

## Testing

```ruby
# spec/services/meal_planning_service_spec.rb
RSpec.describe MealPlanningService do
  let(:user) { create(:user) }
  let(:start_date) { Date.today }
  let(:end_date) { start_date + 6.days }
  
  before do
    create(:nutrition_profile, user: user)
    create(:dietary_preference, user: user)
  end
  
  describe '#generate_plan' do
    it 'generates a valid meal plan' do
      plan = described_class.new(user, start_date, end_date).generate_plan
      
      expect(plan).to include(:daily_plans, :shopping_list, :nutrition_summary)
      expect(plan[:daily_plans].length).to eq(7)
    end
    
    it 'respects dietary restrictions' do
      create(:dietary_restriction, user: user, allergen: 'peanuts')
      
      plan = described_class.new(user, start_date, end_date).generate_plan
      
      plan[:daily_plans].each do |day|
        day[:meals].each do |meal|
          meal[:recipes].each do |recipe|
            expect(recipe.ingredients).not_to include(peanuts)
          end
        end
      end
    end
    
    it 'stays within budget' do
      plan = described_class.new(user, start_date, end_date).generate_plan
      
      expect(plan[:shopping_list][:total_cost]).to be <= user.weekly_budget
    end
  end
end
``` 