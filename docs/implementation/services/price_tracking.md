# Price Tracking Service Implementation

## Service Definition

```ruby
# lib/services/price_tracking_service.rb
class PriceTrackingService
  def initialize(ingredient)
    @ingredient = ingredient
    @price_history = ingredient.price_history
  end

  def track_prices
    {
      current_price: get_current_price,
      price_history: analyze_price_history,
      price_trends: calculate_price_trends,
      recommendations: generate_recommendations
    }
  end

  private

  def get_current_price
    {
      amount: @ingredient.current_price,
      unit: @ingredient.price_unit,
      last_updated: @ingredient.price_updated_at,
      source: @ingredient.price_source
    }
  end

  def analyze_price_history
    {
      historical_prices: get_historical_prices,
      price_changes: calculate_price_changes,
      seasonal_patterns: identify_seasonal_patterns
    }
  end

  def get_historical_prices
    @price_history
      .order(date: :desc)
      .limit(30)
      .map do |record|
        {
          date: record.date,
          price: record.price,
          unit: record.unit,
          source: record.source
        }
      end
  end

  def calculate_price_changes
    recent_prices = get_historical_prices
    return {} if recent_prices.empty?

    {
      daily_change: calculate_daily_change(recent_prices),
      weekly_change: calculate_weekly_change(recent_prices),
      monthly_change: calculate_monthly_change(recent_prices)
    }
  end

  def calculate_daily_change(prices)
    return 0 if prices.length < 2
    ((prices[0][:price] - prices[1][:price]) / prices[1][:price]) * 100
  end

  def calculate_weekly_change(prices)
    return 0 if prices.length < 8
    ((prices[0][:price] - prices[7][:price]) / prices[7][:price]) * 100
  end

  def calculate_monthly_change(prices)
    return 0 if prices.length < 31
    ((prices[0][:price] - prices[30][:price]) / prices[30][:price]) * 100
  end

  def identify_seasonal_patterns
    {
      seasonal_trends: calculate_seasonal_trends,
      peak_seasons: identify_peak_seasons,
      low_seasons: identify_low_seasons
    }
  end

  def calculate_seasonal_trends
    monthly_averages = calculate_monthly_averages
    seasonal_index = calculate_seasonal_index(monthly_averages)
    
    {
      monthly_averages: monthly_averages,
      seasonal_index: seasonal_index,
      trend_line: calculate_trend_line(monthly_averages)
    }
  end

  def calculate_monthly_averages
    @price_history
      .group("DATE_TRUNC('month', date)")
      .average(:price)
      .transform_keys { |k| k.to_date }
  end

  def calculate_seasonal_index(monthly_averages)
    overall_average = monthly_averages.values.sum / monthly_averages.length
    
    monthly_averages.transform_values do |avg|
      (avg / overall_average) * 100
    end
  end

  def calculate_price_trends
    {
      short_term: calculate_short_term_trend,
      long_term: calculate_long_term_trend,
      forecast: generate_price_forecast
    }
  end

  def calculate_short_term_trend
    recent_prices = get_historical_prices.first(7)
    return {} if recent_prices.empty?

    {
      direction: determine_trend_direction(recent_prices),
      magnitude: calculate_trend_magnitude(recent_prices),
      confidence: calculate_trend_confidence(recent_prices)
    }
  end

  def calculate_long_term_trend
    all_prices = get_historical_prices
    return {} if all_prices.empty?

    {
      direction: determine_trend_direction(all_prices),
      magnitude: calculate_trend_magnitude(all_prices),
      confidence: calculate_trend_confidence(all_prices)
    }
  end

  def generate_price_forecast
    {
      next_week: forecast_next_week,
      next_month: forecast_next_month,
      next_quarter: forecast_next_quarter
    }
  end

  def forecast_next_week
    recent_trend = calculate_short_term_trend
    current_price = @ingredient.current_price
    
    {
      predicted_price: calculate_predicted_price(current_price, recent_trend, 7),
      confidence_interval: calculate_confidence_interval(recent_trend, 7)
    }
  end

  def forecast_next_month
    recent_trend = calculate_short_term_trend
    current_price = @ingredient.current_price
    
    {
      predicted_price: calculate_predicted_price(current_price, recent_trend, 30),
      confidence_interval: calculate_confidence_interval(recent_trend, 30)
    }
  end

  def forecast_next_quarter
    recent_trend = calculate_short_term_trend
    current_price = @ingredient.current_price
    
    {
      predicted_price: calculate_predicted_price(current_price, recent_trend, 90),
      confidence_interval: calculate_confidence_interval(recent_trend, 90)
    }
  end

  def generate_recommendations
    {
      buying_recommendations: generate_buying_recommendations,
      storage_recommendations: generate_storage_recommendations,
      alternative_suggestions: generate_alternative_suggestions
    }
  end

  def generate_buying_recommendations
    current_price = @ingredient.current_price
    price_trends = calculate_price_trends
    seasonal_patterns = identify_seasonal_patterns
    
    {
      buy_now: should_buy_now?(current_price, price_trends, seasonal_patterns),
      wait_for: calculate_optimal_buying_time(price_trends, seasonal_patterns),
      quantity_recommendation: recommend_quantity(price_trends, seasonal_patterns)
    }
  end

  def generate_storage_recommendations
    {
      storage_duration: calculate_optimal_storage_duration,
      storage_conditions: get_storage_conditions,
      shelf_life: get_shelf_life
    }
  end

  def generate_alternative_suggestions
    {
      similar_ingredients: find_similar_ingredients,
      price_comparisons: compare_with_alternatives,
      substitution_recommendations: recommend_substitutions
    }
  end

  # Helper methods
  def determine_trend_direction(prices)
    return :stable if prices.length < 2
    
    changes = prices.each_cons(2).map { |a, b| b[:price] - a[:price] }
    average_change = changes.sum / changes.length
    
    if average_change > 0.05
      :increasing
    elsif average_change < -0.05
      :decreasing
    else
      :stable
    end
  end

  def calculate_trend_magnitude(prices)
    return 0 if prices.length < 2
    
    changes = prices.each_cons(2).map { |a, b| ((b[:price] - a[:price]) / a[:price]) * 100 }
    changes.sum / changes.length
  end

  def calculate_trend_confidence(prices)
    return 0 if prices.length < 2
    
    changes = prices.each_cons(2).map { |a, b| b[:price] - a[:price] }
    standard_deviation = Math.sqrt(changes.map { |x| x ** 2 }.sum / changes.length)
    
    # Convert to confidence score (0-100)
    (100 - (standard_deviation * 10)).clamp(0, 100)
  end

  def calculate_predicted_price(current_price, trend, days)
    return current_price if trend.empty?
    
    daily_change = trend[:magnitude] / 100
    current_price * (1 + (daily_change * days))
  end

  def calculate_confidence_interval(trend, days)
    return [0, 0] if trend.empty?
    
    confidence = trend[:confidence] / 100.0
    predicted_price = calculate_predicted_price(@ingredient.current_price, trend, days)
    
    margin = predicted_price * (1 - confidence)
    [predicted_price - margin, predicted_price + margin]
  end

  def should_buy_now?(current_price, price_trends, seasonal_patterns)
    return false if price_trends[:short_term][:direction] == :increasing
    return true if price_trends[:short_term][:direction] == :decreasing
    
    current_seasonal_index = get_current_seasonal_index(seasonal_patterns)
    current_seasonal_index < 100
  end

  def calculate_optimal_buying_time(price_trends, seasonal_patterns)
    return 'now' if should_buy_now?(@ingredient.current_price, price_trends, seasonal_patterns)
    
    seasonal_patterns[:low_seasons].min_by do |season|
      days_until_season(season)
    end
  end

  def recommend_quantity(price_trends, seasonal_patterns)
    return 1 if price_trends[:short_term][:direction] == :increasing
    
    optimal_storage = calculate_optimal_storage_duration
    current_seasonal_index = get_current_seasonal_index(seasonal_patterns)
    
    if current_seasonal_index < 90
      optimal_storage
    else
      1
    end
  end
end
```

## Usage

### Basic Price Tracking
```ruby
ingredient = Ingredient.find(1)
tracking = PriceTrackingService.new(ingredient).track_prices

# Access current price
current_price = tracking[:current_price]
price_amount = current_price[:amount]
price_unit = current_price[:unit]

# Access price history
history = tracking[:price_history]
historical_prices = history[:historical_prices]
price_changes = history[:price_changes]

# Access price trends
trends = tracking[:price_trends]
short_term = trends[:short_term]
long_term = trends[:long_term]
forecast = trends[:forecast]

# Access recommendations
recommendations = tracking[:recommendations]
buying_recommendations = recommendations[:buying_recommendations]
storage_recommendations = recommendations[:storage_recommendations]
```

## Data Models

### Ingredient Model
```ruby
class Ingredient < ApplicationRecord
  has_many :price_history
  
  validates :name, presence: true
  validates :current_price, presence: true, numericality: { greater_than: 0 }
  validates :price_unit, presence: true
  validates :price_source, presence: true
  
  def price_updated_at
    price_history.order(date: :desc).first&.date
  end
end
```

### PriceHistory Model
```ruby
class PriceHistory < ApplicationRecord
  belongs_to :ingredient
  
  validates :date, presence: true
  validates :price, presence: true, numericality: { greater_than: 0 }
  validates :unit, presence: true
  validates :source, presence: true
end
```

## Testing

```ruby
# spec/services/price_tracking_service_spec.rb
RSpec.describe PriceTrackingService do
  let(:ingredient) { create(:ingredient, current_price: 10.0) }
  
  before do
    create_list(:price_history, 30, ingredient: ingredient, price: 10.0)
  end
  
  describe '#track_prices' do
    it 'tracks current price correctly' do
      tracking = described_class.new(ingredient).track_prices
      
      expect(tracking[:current_price][:amount]).to eq(10.0)
    end
    
    it 'calculates price changes correctly' do
      create(:price_history, ingredient: ingredient, price: 9.0, date: 1.day.ago)
      
      tracking = described_class.new(ingredient).track_prices
      changes = tracking[:price_history][:price_changes]
      
      expect(changes[:daily_change]).to be_within(0.1).of(11.11)
    end
    
    it 'generates appropriate recommendations' do
      create(:price_history, ingredient: ingredient, price: 11.0, date: 1.day.ago)
      
      tracking = described_class.new(ingredient).track_prices
      recommendations = tracking[:recommendations]
      
      expect(recommendations[:buying_recommendations][:buy_now]).to be false
    end
  end
end
``` 