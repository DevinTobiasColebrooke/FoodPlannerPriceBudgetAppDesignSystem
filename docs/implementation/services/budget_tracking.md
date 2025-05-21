# Budget Tracking Service Implementation

## Service Definition

```ruby
# lib/services/budget_tracking_service.rb
class BudgetTrackingService
  def initialize(user)
    @user = user
    @budget = user.budget
    @transactions = user.transactions
  end

  def track_spending
    {
      current_period: calculate_current_period,
      historical: calculate_historical_data,
      projections: calculate_projections,
      recommendations: generate_recommendations
    }
  end

  private

  def calculate_current_period
    {
      total_spent: calculate_total_spent,
      remaining_budget: calculate_remaining_budget,
      category_breakdown: calculate_category_breakdown,
      daily_average: calculate_daily_average
    }
  end

  def calculate_total_spent
    @transactions
      .where(date: current_period_range)
      .sum(:amount)
  end

  def calculate_remaining_budget
    @budget.amount - calculate_total_spent
  end

  def calculate_category_breakdown
    @transactions
      .where(date: current_period_range)
      .group(:category)
      .sum(:amount)
  end

  def calculate_daily_average
    days_elapsed = (Date.today - current_period_start).to_i + 1
    calculate_total_spent / days_elapsed
  end

  def calculate_historical_data
    {
      monthly_trends: calculate_monthly_trends,
      category_trends: calculate_category_trends,
      spending_patterns: analyze_spending_patterns
    }
  end

  def calculate_monthly_trends
    last_6_months = 6.times.map { |i| i.months.ago.beginning_of_month }
    
    last_6_months.map do |month|
      {
        month: month,
        total_spent: calculate_monthly_spending(month),
        budget_utilization: calculate_budget_utilization(month)
      }
    end
  end

  def calculate_category_trends
    categories = @transactions.distinct.pluck(:category)
    
    categories.map do |category|
      {
        category: category,
        monthly_average: calculate_category_monthly_average(category),
        trend: calculate_category_trend(category)
      }
    end
  end

  def analyze_spending_patterns
    {
      weekday_vs_weekend: analyze_weekday_weekend_spending,
      time_of_day: analyze_time_of_day_spending,
      location_based: analyze_location_based_spending
    }
  end

  def calculate_projections
    {
      end_of_period: project_end_of_period,
      category_projections: project_category_spending,
      budget_adjustments: suggest_budget_adjustments
    }
  end

  def project_end_of_period
    daily_average = calculate_daily_average
    days_remaining = (current_period_end - Date.today).to_i
    
    {
      projected_spending: daily_average * days_remaining,
      projected_savings: calculate_remaining_budget - (daily_average * days_remaining)
    }
  end

  def project_category_spending
    categories = @transactions.distinct.pluck(:category)
    
    categories.map do |category|
      {
        category: category,
        current_spending: calculate_category_spending(category),
        projected_spending: calculate_category_projection(category)
      }
    end
  end

  def suggest_budget_adjustments
    {
      category_adjustments: suggest_category_adjustments,
      overall_adjustment: suggest_overall_adjustment,
      savings_opportunities: identify_savings_opportunities
    }
  end

  def generate_recommendations
    {
      spending_recommendations: generate_spending_recommendations,
      budget_recommendations: generate_budget_recommendations,
      savings_recommendations: generate_savings_recommendations
    }
  end

  def generate_spending_recommendations
    overspent_categories = identify_overspent_categories
    high_spending_areas = identify_high_spending_areas
    
    {
      reduce_spending: overspent_categories,
      optimize_spending: high_spending_areas,
      alternative_suggestions: generate_alternative_suggestions
    }
  end

  def generate_budget_recommendations
    {
      category_adjustments: recommend_category_adjustments,
      overall_adjustment: recommend_overall_adjustment,
      reallocation_suggestions: suggest_reallocations
    }
  end

  def generate_savings_recommendations
    {
      immediate_savings: identify_immediate_savings,
      long_term_savings: identify_long_term_savings,
      investment_opportunities: suggest_investments
    }
  end

  # Helper methods
  def current_period_range
    current_period_start..current_period_end
  end

  def current_period_start
    @budget.period_start
  end

  def current_period_end
    @budget.period_end
  end

  def calculate_monthly_spending(month)
    @transactions
      .where(date: month.beginning_of_month..month.end_of_month)
      .sum(:amount)
  end

  def calculate_budget_utilization(month)
    spent = calculate_monthly_spending(month)
    budget = @budget.monthly_amount
    (spent / budget) * 100
  end

  def calculate_category_monthly_average(category)
    @transactions
      .where(category: category)
      .where('date >= ?', 6.months.ago)
      .average(:amount)
  end

  def calculate_category_trend(category)
    monthly_spending = 6.times.map do |i|
      month = i.months.ago.beginning_of_month
      calculate_category_spending_for_month(category, month)
    end
    
    calculate_trend_line(monthly_spending)
  end

  def calculate_category_spending_for_month(category, month)
    @transactions
      .where(category: category)
      .where(date: month.beginning_of_month..month.end_of_month)
      .sum(:amount)
  end

  def calculate_trend_line(values)
    # Simple linear regression
    n = values.length
    x_mean = (0...n).sum / n.to_f
    y_mean = values.sum / n.to_f
    
    numerator = (0...n).sum { |i| (i - x_mean) * (values[i] - y_mean) }
    denominator = (0...n).sum { |i| (i - x_mean) ** 2 }
    
    slope = denominator.zero? ? 0 : numerator / denominator
    intercept = y_mean - slope * x_mean
    
    { slope: slope, intercept: intercept }
  end
end
```

## Usage

### Basic Budget Tracking
```ruby
user = User.find(1)
tracking = BudgetTrackingService.new(user).track_spending

# Access current period data
current_period = tracking[:current_period]
total_spent = current_period[:total_spent]
remaining_budget = current_period[:remaining_budget]
category_breakdown = current_period[:category_breakdown]

# Access historical data
historical = tracking[:historical]
monthly_trends = historical[:monthly_trends]
category_trends = historical[:category_trends]

# Access projections
projections = tracking[:projections]
end_of_period = projections[:end_of_period]
category_projections = projections[:category_projections]

# Access recommendations
recommendations = tracking[:recommendations]
spending_recommendations = recommendations[:spending_recommendations]
budget_recommendations = recommendations[:budget_recommendations]
```

## Data Models

### Budget Model
```ruby
class Budget < ApplicationRecord
  belongs_to :user
  has_many :transactions
  
  validates :amount, presence: true, numericality: { greater_than: 0 }
  validates :period_start, presence: true
  validates :period_end, presence: true
  validate :end_date_after_start_date
  
  def monthly_amount
    amount / ((period_end - period_start) / 30.0)
  end
  
  private
  
  def end_date_after_start_date
    return unless period_start && period_end
    if period_end <= period_start
      errors.add(:period_end, "must be after period start")
    end
  end
end
```

### Transaction Model
```ruby
class Transaction < ApplicationRecord
  belongs_to :user
  belongs_to :budget
  
  validates :amount, presence: true, numericality: { greater_than: 0 }
  validates :date, presence: true
  validates :category, presence: true
  validates :description, presence: true
  
  enum category: {
    groceries: 0,
    dining_out: 1,
    household: 2,
    transportation: 3,
    entertainment: 4,
    utilities: 5,
    other: 6
  }
end
```

## Testing

```ruby
# spec/services/budget_tracking_service_spec.rb
RSpec.describe BudgetTrackingService do
  let(:user) { create(:user) }
  let(:budget) { create(:budget, user: user, amount: 1000) }
  
  before do
    create_list(:transaction, 5, user: user, budget: budget, amount: 100)
  end
  
  describe '#track_spending' do
    it 'calculates current period spending correctly' do
      tracking = described_class.new(user).track_spending
      
      expect(tracking[:current_period][:total_spent]).to eq(500)
      expect(tracking[:current_period][:remaining_budget]).to eq(500)
    end
    
    it 'calculates category breakdown correctly' do
      create(:transaction, user: user, budget: budget, category: :groceries, amount: 200)
      
      tracking = described_class.new(user).track_spending
      breakdown = tracking[:current_period][:category_breakdown]
      
      expect(breakdown[:groceries]).to eq(200)
    end
    
    it 'generates appropriate recommendations' do
      create(:transaction, user: user, budget: budget, amount: 800)
      
      tracking = described_class.new(user).track_spending
      recommendations = tracking[:recommendations]
      
      expect(recommendations[:spending_recommendations][:reduce_spending]).not_to be_empty
    end
  end
end
``` 