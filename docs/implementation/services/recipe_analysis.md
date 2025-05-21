# Recipe Analysis Service Implementation

## Service Definition

```ruby
# lib/services/recipe_analysis_service.rb
class RecipeAnalysisService
  def initialize(recipe)
    @recipe = recipe
    @ingredients = recipe.ingredients
    @portions = recipe.portions
  end

  def analyze
    {
      total_nutrients: calculate_total_nutrients,
      per_portion: calculate_per_portion,
      cost_analysis: calculate_cost_analysis,
      nutrient_density: calculate_nutrient_density
    }
  end

  private

  def calculate_total_nutrients
    nutrients = {}
    @ingredients.each do |ingredient|
      ingredient.nutrients.each do |nutrient|
        nutrients[nutrient.name] ||= 0
        nutrients[nutrient.name] += calculate_nutrient_amount(ingredient, nutrient)
      end
    end
    nutrients
  end

  def calculate_per_portion
    total_nutrients = calculate_total_nutrients
    per_portion = {}
    
    total_nutrients.each do |nutrient, amount|
      per_portion[nutrient] = amount / @portions
    end
    
    per_portion
  end

  def calculate_cost_analysis
    total_cost = @ingredients.sum(&:cost)
    cost_per_portion = total_cost / @portions
    
    {
      total_cost: total_cost,
      cost_per_portion: cost_per_portion,
      ingredient_costs: @ingredients.map { |i| [i.name, i.cost] }.to_h
    }
  end

  def calculate_nutrient_density
    total_weight = @ingredients.sum(&:weight)
    nutrients = calculate_total_nutrients
    
    density = {}
    nutrients.each do |nutrient, amount|
      density[nutrient] = amount / total_weight
    end
    
    density
  end

  def calculate_nutrient_amount(ingredient, nutrient)
    base_amount = ingredient.nutrient_amount(nutrient)
    conversion_factor = get_conversion_factor(ingredient, nutrient)
    base_amount * conversion_factor
  end

  def get_conversion_factor(ingredient, nutrient)
    # Get conversion factor based on ingredient preparation method
    # and nutrient bioavailability
    conversion_factors = {
      raw: 1.0,
      cooked: get_cooking_conversion_factor(ingredient, nutrient),
      processed: get_processing_conversion_factor(ingredient, nutrient)
    }
    
    conversion_factors[ingredient.preparation_method] || 1.0
  end

  def get_cooking_conversion_factor(ingredient, nutrient)
    # Lookup cooking conversion factors from database
    CookingConversionFactor.find_by(
      ingredient_type: ingredient.type,
      nutrient: nutrient,
      cooking_method: ingredient.cooking_method
    )&.factor || 1.0
  end

  def get_processing_conversion_factor(ingredient, nutrient)
    # Lookup processing conversion factors from database
    ProcessingConversionFactor.find_by(
      ingredient_type: ingredient.type,
      nutrient: nutrient,
      processing_method: ingredient.processing_method
    )&.factor || 1.0
  end
end
```

## Usage

### Basic Analysis
```ruby
recipe = Recipe.find(1)
analysis = RecipeAnalysisService.new(recipe).analyze

# Access results
total_nutrients = analysis[:total_nutrients]
per_portion = analysis[:per_portion]
cost_analysis = analysis[:cost_analysis]
nutrient_density = analysis[:nutrient_density]
```

### Individual Calculations
```ruby
service = RecipeAnalysisService.new(recipe)

# Calculate just total nutrients
total_nutrients = service.calculate_total_nutrients

# Calculate just per portion values
per_portion = service.calculate_per_portion

# Calculate just cost analysis
cost_analysis = service.calculate_cost_analysis

# Calculate just nutrient density
nutrient_density = service.calculate_nutrient_density
```

## Data Models

### Recipe Model
```ruby
class Recipe < ApplicationRecord
  has_many :ingredients
  has_many :portions
  
  validates :name, presence: true
  validates :portions, presence: true, numericality: { greater_than: 0 }
end
```

### Ingredient Model
```ruby
class Ingredient < ApplicationRecord
  belongs_to :recipe
  has_many :nutrients
  
  validates :name, presence: true
  validates :amount, presence: true, numericality: { greater_than: 0 }
  validates :unit, presence: true
  validates :cost, presence: true, numericality: { greater_than_or_equal_to: 0 }
  validates :weight, presence: true, numericality: { greater_than: 0 }
  
  enum preparation_method: {
    raw: 0,
    cooked: 1,
    processed: 2
  }
  
  def nutrient_amount(nutrient)
    nutrients.find_by(name: nutrient.name)&.amount || 0
  end
end
```

### Nutrient Model
```ruby
class Nutrient < ApplicationRecord
  belongs_to :ingredient
  
  validates :name, presence: true
  validates :amount, presence: true, numericality: { greater_than_or_equal_to: 0 }
  validates :unit, presence: true
end
```

### Conversion Factor Models
```ruby
class CookingConversionFactor < ApplicationRecord
  validates :ingredient_type, presence: true
  validates :nutrient, presence: true
  validates :cooking_method, presence: true
  validates :factor, presence: true, numericality: { greater_than: 0 }
end

class ProcessingConversionFactor < ApplicationRecord
  validates :ingredient_type, presence: true
  validates :nutrient, presence: true
  validates :processing_method, presence: true
  validates :factor, presence: true, numericality: { greater_than: 0 }
end
```

## Error Handling

```ruby
class RecipeAnalysisService
  def analyze
    validate_recipe
    {
      total_nutrients: calculate_total_nutrients,
      per_portion: calculate_per_portion,
      cost_analysis: calculate_cost_analysis,
      nutrient_density: calculate_nutrient_density
    }
  rescue StandardError => e
    Rails.logger.error "Error analyzing recipe: #{e.message}"
    raise RecipeAnalysisError, "Failed to analyze recipe: #{e.message}"
  end

  private

  def validate_recipe
    raise RecipeAnalysisError, "Recipe is required" unless @recipe
    raise RecipeAnalysisError, "Recipe must have ingredients" if @ingredients.empty?
    raise RecipeAnalysisError, "Recipe must have portions" unless @portions&.positive?
  end
end

class RecipeAnalysisError < StandardError; end
```

## Testing

```ruby
# spec/services/recipe_analysis_service_spec.rb
RSpec.describe RecipeAnalysisService do
  let(:recipe) { create(:recipe, portions: 4) }
  let(:ingredient1) { create(:ingredient, recipe: recipe, amount: 100, unit: 'g') }
  let(:ingredient2) { create(:ingredient, recipe: recipe, amount: 200, unit: 'g') }
  
  before do
    create(:nutrient, ingredient: ingredient1, name: 'Protein', amount: 10, unit: 'g')
    create(:nutrient, ingredient: ingredient2, name: 'Protein', amount: 20, unit: 'g')
  end
  
  describe '#analyze' do
    it 'calculates total nutrients correctly' do
      analysis = described_class.new(recipe).analyze
      expect(analysis[:total_nutrients]['Protein']).to eq(30)
    end
    
    it 'calculates per portion correctly' do
      analysis = described_class.new(recipe).analyze
      expect(analysis[:per_portion]['Protein']).to eq(7.5)
    end
    
    it 'calculates cost analysis correctly' do
      analysis = described_class.new(recipe).analyze
      expect(analysis[:cost_analysis][:total_cost]).to eq(ingredient1.cost + ingredient2.cost)
    end
    
    it 'calculates nutrient density correctly' do
      analysis = described_class.new(recipe).analyze
      total_weight = ingredient1.weight + ingredient2.weight
      expect(analysis[:nutrient_density]['Protein']).to eq(30.0 / total_weight)
    end
  end
end
``` 