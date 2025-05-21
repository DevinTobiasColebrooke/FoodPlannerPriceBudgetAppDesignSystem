# User Preferences Service Implementation

## Service Definition

```ruby
# lib/services/user_preferences_service.rb
class UserPreferencesService
  def initialize(user)
    @user = user
    @preferences = user.preferences
  end

  def manage_preferences
    {
      dietary: manage_dietary_preferences,
      meal_planning: manage_meal_planning_preferences,
      shopping: manage_shopping_preferences,
      notifications: manage_notification_preferences
    }
  end

  private

  def manage_dietary_preferences
    {
      restrictions: manage_dietary_restrictions,
      preferences: manage_dietary_preferences,
      allergies: manage_allergies,
      nutritional_goals: manage_nutritional_goals
    }
  end

  def manage_dietary_restrictions
    {
      current: get_current_restrictions,
      add: add_dietary_restriction,
      remove: remove_dietary_restriction,
      update: update_dietary_restriction
    }
  end

  def get_current_restrictions
    @preferences.dietary_restrictions.map do |restriction|
      {
        type: restriction.type,
        description: restriction.description,
        severity: restriction.severity,
        notes: restriction.notes
      }
    end
  end

  def add_dietary_restriction(restriction_data)
    restriction = @preferences.dietary_restrictions.build(restriction_data)
    if restriction.save
      {
        success: true,
        restriction: restriction,
        message: 'Dietary restriction added successfully'
      }
    else
      {
        success: false,
        errors: restriction.errors.full_messages
      }
    end
  end

  def remove_dietary_restriction(restriction_id)
    restriction = @preferences.dietary_restrictions.find(restriction_id)
    if restriction.destroy
      {
        success: true,
        message: 'Dietary restriction removed successfully'
      }
    else
      {
        success: false,
        errors: restriction.errors.full_messages
      }
    end
  end

  def update_dietary_restriction(restriction_id, update_data)
    restriction = @preferences.dietary_restrictions.find(restriction_id)
    if restriction.update(update_data)
      {
        success: true,
        restriction: restriction,
        message: 'Dietary restriction updated successfully'
      }
    else
      {
        success: false,
        errors: restriction.errors.full_messages
      }
    end
  end

  def manage_dietary_preferences
    {
      current: get_current_preferences,
      add: add_dietary_preference,
      remove: remove_dietary_preference,
      update: update_dietary_preference
    }
  end

  def get_current_preferences
    @preferences.dietary_preferences.map do |preference|
      {
        type: preference.type,
        description: preference.description,
        priority: preference.priority,
        notes: preference.notes
      }
    end
  end

  def add_dietary_preference(preference_data)
    preference = @preferences.dietary_preferences.build(preference_data)
    if preference.save
      {
        success: true,
        preference: preference,
        message: 'Dietary preference added successfully'
      }
    else
      {
        success: false,
        errors: preference.errors.full_messages
      }
    end
  end

  def remove_dietary_preference(preference_id)
    preference = @preferences.dietary_preferences.find(preference_id)
    if preference.destroy
      {
        success: true,
        message: 'Dietary preference removed successfully'
      }
    else
      {
        success: false,
        errors: preference.errors.full_messages
      }
    end
  end

  def update_dietary_preference(preference_id, update_data)
    preference = @preferences.dietary_preferences.find(preference_id)
    if preference.update(update_data)
      {
        success: true,
        preference: preference,
        message: 'Dietary preference updated successfully'
      }
    else
      {
        success: false,
        errors: preference.errors.full_messages
      }
    end
  end

  def manage_allergies
    {
      current: get_current_allergies,
      add: add_allergy,
      remove: remove_allergy,
      update: update_allergy
    }
  end

  def get_current_allergies
    @preferences.allergies.map do |allergy|
      {
        allergen: allergy.allergen,
        severity: allergy.severity,
        symptoms: allergy.symptoms,
        notes: allergy.notes
      }
    end
  end

  def add_allergy(allergy_data)
    allergy = @preferences.allergies.build(allergy_data)
    if allergy.save
      {
        success: true,
        allergy: allergy,
        message: 'Allergy added successfully'
      }
    else
      {
        success: false,
        errors: allergy.errors.full_messages
      }
    end
  end

  def remove_allergy(allergy_id)
    allergy = @preferences.allergies.find(allergy_id)
    if allergy.destroy
      {
        success: true,
        message: 'Allergy removed successfully'
      }
    else
      {
        success: false,
        errors: allergy.errors.full_messages
      }
    end
  end

  def update_allergy(allergy_id, update_data)
    allergy = @preferences.allergies.find(allergy_id)
    if allergy.update(update_data)
      {
        success: true,
        allergy: allergy,
        message: 'Allergy updated successfully'
      }
    else
      {
        success: false,
        errors: allergy.errors.full_messages
      }
    end
  end

  def manage_nutritional_goals
    {
      current: get_current_goals,
      add: add_nutritional_goal,
      remove: remove_nutritional_goal,
      update: update_nutritional_goal
    }
  end

  def get_current_goals
    @preferences.nutritional_goals.map do |goal|
      {
        nutrient: goal.nutrient,
        target: goal.target,
        unit: goal.unit,
        priority: goal.priority,
        notes: goal.notes
      }
    end
  end

  def add_nutritional_goal(goal_data)
    goal = @preferences.nutritional_goals.build(goal_data)
    if goal.save
      {
        success: true,
        goal: goal,
        message: 'Nutritional goal added successfully'
      }
    else
      {
        success: false,
        errors: goal.errors.full_messages
      }
    end
  end

  def remove_nutritional_goal(goal_id)
    goal = @preferences.nutritional_goals.find(goal_id)
    if goal.destroy
      {
        success: true,
        message: 'Nutritional goal removed successfully'
      }
    else
      {
        success: false,
        errors: goal.errors.full_messages
      }
    end
  end

  def update_nutritional_goal(goal_id, update_data)
    goal = @preferences.nutritional_goals.find(goal_id)
    if goal.update(update_data)
      {
        success: true,
        goal: goal,
        message: 'Nutritional goal updated successfully'
      }
    else
      {
        success: false,
        errors: goal.errors.full_messages
      }
    end
  end

  def manage_meal_planning_preferences
    {
      meal_times: manage_meal_times,
      portion_sizes: manage_portion_sizes,
      cooking_preferences: manage_cooking_preferences,
      meal_variety: manage_meal_variety
    }
  end

  def manage_meal_times
    {
      current: get_current_meal_times,
      update: update_meal_times
    }
  end

  def get_current_meal_times
    @preferences.meal_times.map do |meal_time|
      {
        meal_type: meal_time.meal_type,
        preferred_time: meal_time.preferred_time,
        flexibility: meal_time.flexibility,
        notes: meal_time.notes
      }
    end
  end

  def update_meal_times(meal_times_data)
    if @preferences.update(meal_times: meal_times_data)
      {
        success: true,
        message: 'Meal times updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_portion_sizes
    {
      current: get_current_portion_sizes,
      update: update_portion_sizes
    }
  end

  def get_current_portion_sizes
    @preferences.portion_sizes.map do |portion|
      {
        meal_type: portion.meal_type,
        size: portion.size,
        unit: portion.unit,
        notes: portion.notes
      }
    end
  end

  def update_portion_sizes(portion_sizes_data)
    if @preferences.update(portion_sizes: portion_sizes_data)
      {
        success: true,
        message: 'Portion sizes updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_cooking_preferences
    {
      current: get_current_cooking_preferences,
      update: update_cooking_preferences
    }
  end

  def get_current_cooking_preferences
    {
      cooking_skill_level: @preferences.cooking_skill_level,
      preferred_cooking_methods: @preferences.preferred_cooking_methods,
      cooking_time_preferences: @preferences.cooking_time_preferences,
      kitchen_equipment: @preferences.kitchen_equipment
    }
  end

  def update_cooking_preferences(cooking_preferences_data)
    if @preferences.update(cooking_preferences_data)
      {
        success: true,
        message: 'Cooking preferences updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_meal_variety
    {
      current: get_current_meal_variety,
      update: update_meal_variety
    }
  end

  def get_current_meal_variety
    {
      cuisine_preferences: @preferences.cuisine_preferences,
      meal_rotation_frequency: @preferences.meal_rotation_frequency,
      new_recipe_frequency: @preferences.new_recipe_frequency
    }
  end

  def update_meal_variety(meal_variety_data)
    if @preferences.update(meal_variety_data)
      {
        success: true,
        message: 'Meal variety preferences updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_shopping_preferences
    {
      store_preferences: manage_store_preferences,
      shopping_schedule: manage_shopping_schedule,
      budget_preferences: manage_budget_preferences
    }
  end

  def manage_store_preferences
    {
      current: get_current_store_preferences,
      update: update_store_preferences
    }
  end

  def get_current_store_preferences
    @preferences.store_preferences.map do |store|
      {
        store_name: store.name,
        priority: store.priority,
        distance: store.distance,
        notes: store.notes
      }
    end
  end

  def update_store_preferences(store_preferences_data)
    if @preferences.update(store_preferences: store_preferences_data)
      {
        success: true,
        message: 'Store preferences updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_shopping_schedule
    {
      current: get_current_shopping_schedule,
      update: update_shopping_schedule
    }
  end

  def get_current_shopping_schedule
    {
      preferred_day: @preferences.preferred_shopping_day,
      preferred_time: @preferences.preferred_shopping_time,
      frequency: @preferences.shopping_frequency
    }
  end

  def update_shopping_schedule(schedule_data)
    if @preferences.update(schedule_data)
      {
        success: true,
        message: 'Shopping schedule updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_budget_preferences
    {
      current: get_current_budget_preferences,
      update: update_budget_preferences
    }
  end

  def get_current_budget_preferences
    {
      weekly_budget: @preferences.weekly_budget,
      budget_priorities: @preferences.budget_priorities,
      savings_goals: @preferences.savings_goals
    }
  end

  def update_budget_preferences(budget_preferences_data)
    if @preferences.update(budget_preferences_data)
      {
        success: true,
        message: 'Budget preferences updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end

  def manage_notification_preferences
    {
      current: get_current_notification_preferences,
      update: update_notification_preferences
    }
  end

  def get_current_notification_preferences
    {
      meal_reminders: @preferences.meal_reminders,
      shopping_reminders: @preferences.shopping_reminders,
      price_alerts: @preferences.price_alerts,
      recipe_suggestions: @preferences.recipe_suggestions
    }
  end

  def update_notification_preferences(notification_preferences_data)
    if @preferences.update(notification_preferences_data)
      {
        success: true,
        message: 'Notification preferences updated successfully'
      }
    else
      {
        success: false,
        errors: @preferences.errors.full_messages
      }
    end
  end
end
```

## Usage

### Basic Preference Management
```ruby
user = User.find(1)
preferences = UserPreferencesService.new(user).manage_preferences

# Access dietary preferences
dietary = preferences[:dietary]
restrictions = dietary[:restrictions]
preferences = dietary[:preferences]
allergies = dietary[:allergies]
goals = dietary[:nutritional_goals]

# Access meal planning preferences
meal_planning = preferences[:meal_planning]
meal_times = meal_planning[:meal_times]
portion_sizes = meal_planning[:portion_sizes]
cooking_preferences = meal_planning[:cooking_preferences]

# Access shopping preferences
shopping = preferences[:shopping]
store_preferences = shopping[:store_preferences]
shopping_schedule = shopping[:shopping_schedule]
budget_preferences = shopping[:budget_preferences]

# Access notification preferences
notifications = preferences[:notifications]
```

### Managing Dietary Restrictions
```ruby
service = UserPreferencesService.new(user)

# Add a new restriction
result = service.manage_dietary_preferences[:restrictions][:add].call(
  type: 'vegetarian',
  description: 'No meat products',
  severity: 'strict'
)

# Update an existing restriction
result = service.manage_dietary_preferences[:restrictions][:update].call(
  restriction_id,
  severity: 'moderate'
)

# Remove a restriction
result = service.manage_dietary_preferences[:restrictions][:remove].call(restriction_id)
```

## Data Models

### UserPreferences Model
```ruby
class UserPreferences < ApplicationRecord
  belongs_to :user
  
  has_many :dietary_restrictions
  has_many :dietary_preferences
  has_many :allergies
  has_many :nutritional_goals
  has_many :meal_times
  has_many :portion_sizes
  has_many :store_preferences
  
  validates :user, presence: true
  
  # Cooking preferences
  enum cooking_skill_level: {
    beginner: 0,
    intermediate: 1,
    advanced: 2
  }
  
  # Shopping preferences
  enum preferred_shopping_day: {
    monday: 0,
    tuesday: 1,
    wednesday: 2,
    thursday: 3,
    friday: 4,
    saturday: 5,
    sunday: 6
  }
  
  # Notification preferences
  serialize :meal_reminders, Array
  serialize :shopping_reminders, Array
  serialize :price_alerts, Array
  serialize :recipe_suggestions, Array
end
```

### DietaryRestriction Model
```ruby
class DietaryRestriction < ApplicationRecord
  belongs_to :user_preferences
  
  validates :type, presence: true
  validates :description, presence: true
  validates :severity, presence: true
  
  enum severity: {
    strict: 0,
    moderate: 1,
    flexible: 2
  }
end
```

## Testing

```ruby
# spec/services/user_preferences_service_spec.rb
RSpec.describe UserPreferencesService do
  let(:user) { create(:user) }
  let(:preferences) { create(:user_preferences, user: user) }
  
  before do
    allow(user).to receive(:preferences).and_return(preferences)
  end
  
  describe '#manage_preferences' do
    it 'manages dietary preferences' do
      service = described_class.new(user)
      result = service.manage_preferences[:dietary]
      
      expect(result).to include(:restrictions, :preferences, :allergies, :nutritional_goals)
    end
    
    it 'adds dietary restrictions' do
      service = described_class.new(user)
      result = service.manage_preferences[:dietary][:restrictions][:add].call(
        type: 'vegetarian',
        description: 'No meat products',
        severity: 'strict'
      )
      
      expect(result[:success]).to be true
      expect(result[:restriction]).to be_present
    end
    
    it 'updates meal planning preferences' do
      service = described_class.new(user)
      result = service.manage_preferences[:meal_planning][:meal_times][:update].call(
        breakfast_time: '08:00',
        lunch_time: '12:00',
        dinner_time: '18:00'
      )
      
      expect(result[:success]).to be true
    end
  end
end
``` 