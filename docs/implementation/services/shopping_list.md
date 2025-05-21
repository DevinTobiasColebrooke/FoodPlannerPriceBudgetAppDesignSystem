# Shopping List Service Implementation

## Service Definition

```ruby
# lib/services/shopping_list_service.rb
class ShoppingListService
  def initialize(user)
    @user = user
    @meal_plan = user.current_meal_plan
    @budget = user.budget
  end

  def generate_list
    {
      items: generate_items,
      organization: organize_items,
      cost_analysis: analyze_costs,
      recommendations: generate_recommendations
    }
  end

  private

  def generate_items
    ingredients = collect_ingredients
    grouped_items = group_ingredients(ingredients)
    add_price_information(grouped_items)
  end

  def collect_ingredients
    return [] unless @meal_plan
    
    @meal_plan.daily_plans.flat_map do |day|
      day[:meals].flat_map do |meal|
        meal[:recipes].flat_map do |recipe|
          recipe.ingredients.map do |ingredient|
            {
              name: ingredient.name,
              amount: ingredient.amount,
              unit: ingredient.unit,
              recipe: recipe.name,
              meal: meal[:type],
              date: day[:date]
            }
          end
        end
      end
    end
  end

  def group_ingredients(ingredients)
    ingredients.group_by { |i| i[:name] }.map do |name, items|
      {
        name: name,
        total_amount: sum_amounts(items),
        unit: items.first[:unit],
        recipes: items.map { |i| i[:recipe] }.uniq,
        meals: items.map { |i| i[:meal] }.uniq,
        dates: items.map { |i| i[:date] }.uniq
      }
    end
  end

  def add_price_information(items)
    items.map do |item|
      price_info = get_price_information(item[:name])
      item.merge(
        price_per_unit: price_info[:price],
        total_cost: price_info[:price] * item[:total_amount],
        price_source: price_info[:source],
        price_updated_at: price_info[:updated_at]
      )
    end
  end

  def organize_items
    {
      by_store_section: organize_by_store_section,
      by_recipe: organize_by_recipe,
      by_meal: organize_by_meal,
      by_date: organize_by_date
    }
  end

  def organize_by_store_section
    sections = {
      produce: [],
      dairy: [],
      meat: [],
      pantry: [],
      frozen: [],
      other: []
    }
    
    generate_items.each do |item|
      section = determine_store_section(item[:name])
      sections[section] << item
    end
    
    sections
  end

  def organize_by_recipe
    generate_items.group_by { |item| item[:recipes] }
  end

  def organize_by_meal
    generate_items.group_by { |item| item[:meals] }
  end

  def organize_by_date
    generate_items.group_by { |item| item[:dates] }
  end

  def analyze_costs
    items = generate_items
    total_cost = items.sum { |item| item[:total_cost] }
    
    {
      total_cost: total_cost,
      cost_by_section: calculate_cost_by_section,
      cost_by_recipe: calculate_cost_by_recipe,
      cost_by_meal: calculate_cost_by_meal,
      budget_comparison: compare_with_budget(total_cost)
    }
  end

  def calculate_cost_by_section
    sections = organize_by_store_section
    sections.transform_values do |items|
      items.sum { |item| item[:total_cost] }
    end
  end

  def calculate_cost_by_recipe
    recipes = organize_by_recipe
    recipes.transform_values do |items|
      items.sum { |item| item[:total_cost] }
    end
  end

  def calculate_cost_by_meal
    meals = organize_by_meal
    meals.transform_values do |items|
      items.sum { |item| item[:total_cost] }
    end
  end

  def compare_with_budget(total_cost)
    return {} unless @budget
    
    {
      total_cost: total_cost,
      budget_amount: @budget.amount,
      remaining_budget: @budget.amount - total_cost,
      percentage_used: (total_cost / @budget.amount) * 100
    }
  end

  def generate_recommendations
    {
      substitutions: suggest_substitutions,
      bulk_buying: suggest_bulk_buying,
      store_recommendations: suggest_stores,
      timing_recommendations: suggest_timing
    }
  end

  def suggest_substitutions
    generate_items.map do |item|
      {
        item: item[:name],
        alternatives: find_alternatives(item),
        cost_comparison: compare_alternatives(item)
      }
    end
  end

  def suggest_bulk_buying
    generate_items.select { |item| should_bulk_buy?(item) }.map do |item|
      {
        item: item[:name],
        current_price: item[:price_per_unit],
        bulk_price: get_bulk_price(item),
        savings: calculate_bulk_savings(item),
        storage_recommendations: get_storage_recommendations(item)
      }
    end
  end

  def suggest_stores
    items = generate_items
    stores = find_available_stores(items)
    
    stores.map do |store|
      {
        store: store[:name],
        total_cost: calculate_store_cost(store, items),
        distance: store[:distance],
        rating: store[:rating],
        items_available: calculate_items_available(store, items)
      }
    end.sort_by { |store| store[:total_cost] }
  end

  def suggest_timing
    {
      best_shopping_day: determine_best_shopping_day,
      time_of_day: determine_best_time_of_day,
      frequency: determine_shopping_frequency
    }
  end

  # Helper methods
  def sum_amounts(items)
    items.sum { |item| convert_to_base_unit(item[:amount], item[:unit]) }
  end

  def convert_to_base_unit(amount, unit)
    conversion_factors = {
      'g' => 1,
      'kg' => 1000,
      'ml' => 1,
      'l' => 1000,
      'tsp' => 5,
      'tbsp' => 15,
      'cup' => 240,
      'oz' => 28.35,
      'lb' => 453.59
    }
    
    amount * (conversion_factors[unit] || 1)
  end

  def determine_store_section(item_name)
    section_mappings = {
      /apple|banana|orange|lettuce|tomato/i => :produce,
      /milk|cheese|yogurt|butter/i => :dairy,
      /beef|chicken|pork|fish/i => :meat,
      /rice|pasta|flour|sugar/i => :pantry,
      /frozen|ice cream/i => :frozen
    }
    
    section_mappings.find { |pattern, _| pattern.match?(item_name) }&.last || :other
  end

  def get_price_information(item_name)
    ingredient = Ingredient.find_by(name: item_name)
    return default_price_info unless ingredient
    
    {
      price: ingredient.current_price,
      source: ingredient.price_source,
      updated_at: ingredient.price_updated_at
    }
  end

  def default_price_info
    {
      price: 0,
      source: 'unknown',
      updated_at: nil
    }
  end

  def should_bulk_buy?(item)
    return false unless item[:price_per_unit]
    
    bulk_price = get_bulk_price(item)
    return false unless bulk_price
    
    savings = calculate_bulk_savings(item)
    savings > 0 && item[:total_amount] >= get_bulk_quantity_threshold(item)
  end

  def get_bulk_price(item)
    # Implementation would depend on bulk pricing data source
    nil
  end

  def calculate_bulk_savings(item)
    return 0 unless item[:price_per_unit] && get_bulk_price(item)
    
    (item[:price_per_unit] - get_bulk_price(item)) * item[:total_amount]
  end

  def get_bulk_quantity_threshold(item)
    # Implementation would depend on item type and storage considerations
    5
  end

  def find_alternatives(item)
    # Implementation would depend on ingredient substitution database
    []
  end

  def compare_alternatives(item)
    alternatives = find_alternatives(item)
    alternatives.map do |alt|
      {
        name: alt[:name],
        price: alt[:price],
        price_difference: alt[:price] - item[:price_per_unit],
        nutritional_difference: calculate_nutritional_difference(item, alt)
      }
    end
  end

  def calculate_nutritional_difference(item, alternative)
    # Implementation would depend on nutritional database
    {}
  end

  def find_available_stores(items)
    # Implementation would depend on store database
    []
  end

  def calculate_store_cost(store, items)
    # Implementation would depend on store pricing data
    0
  end

  def calculate_items_available(store, items)
    # Implementation would depend on store inventory data
    0
  end

  def determine_best_shopping_day
    # Implementation would depend on store schedules and historical data
    'Wednesday'
  end

  def determine_best_time_of_day
    # Implementation would depend on store traffic data
    'morning'
  end

  def determine_shopping_frequency
    # Implementation would depend on meal plan and storage capacity
    'weekly'
  end
end
```

## Usage

### Basic Shopping List Generation
```ruby
user = User.find(1)
list = ShoppingListService.new(user).generate_list

# Access items
items = list[:items]
items.each do |item|
  puts "#{item[:name]}: #{item[:total_amount]} #{item[:unit]}"
end

# Access organization
organization = list[:organization]
by_section = organization[:by_store_section]
by_recipe = organization[:by_recipe]

# Access cost analysis
costs = list[:cost_analysis]
total_cost = costs[:total_cost]
cost_by_section = costs[:cost_by_section]

# Access recommendations
recommendations = list[:recommendations]
substitutions = recommendations[:substitutions]
bulk_buying = recommendations[:bulk_buying]
```

## Data Models

### ShoppingList Model
```ruby
class ShoppingList < ApplicationRecord
  belongs_to :user
  belongs_to :meal_plan
  
  validates :items, presence: true
  validates :total_cost, presence: true, numericality: { greater_than_or_equal_to: 0 }
  
  def items_by_section
    items.group_by { |item| item[:store_section] }
  end
  
  def items_by_recipe
    items.group_by { |item| item[:recipes] }
  end
end
```

### Store Model
```ruby
class Store < ApplicationRecord
  has_many :store_sections
  has_many :store_items
  
  validates :name, presence: true
  validates :location, presence: true
  
  def distance_from_user(user)
    # Implementation would depend on geolocation service
    0
  end
  
  def items_available(items)
    store_items.where(item: items).count
  end
end
```

## Testing

```ruby
# spec/services/shopping_list_service_spec.rb
RSpec.describe ShoppingListService do
  let(:user) { create(:user) }
  let(:meal_plan) { create(:meal_plan, user: user) }
  
  before do
    allow(user).to receive(:current_meal_plan).and_return(meal_plan)
  end
  
  describe '#generate_list' do
    it 'generates a valid shopping list' do
      list = described_class.new(user).generate_list
      
      expect(list).to include(:items, :organization, :cost_analysis, :recommendations)
    end
    
    it 'groups ingredients correctly' do
      create(:recipe, meal_plan: meal_plan, ingredients: [
        create(:ingredient, name: 'Apple', amount: 2, unit: 'kg'),
        create(:ingredient, name: 'Apple', amount: 1, unit: 'kg')
      ])
      
      list = described_class.new(user).generate_list
      items = list[:items]
      
      apple_item = items.find { |item| item[:name] == 'Apple' }
      expect(apple_item[:total_amount]).to eq(3)
    end
    
    it 'calculates costs correctly' do
      create(:recipe, meal_plan: meal_plan, ingredients: [
        create(:ingredient, name: 'Apple', amount: 2, unit: 'kg', price: 2.0)
      ])
      
      list = described_class.new(user).generate_list
      costs = list[:cost_analysis]
      
      expect(costs[:total_cost]).to eq(4.0)
    end
  end
end
``` 