# Phase 1: Core Data Model & Migrations

## Create Master Migration

Generate a new migration:
```bash
bin/rails g migration CreateAppSchema
```

Copy the content from `design-system/docs/database-schema/migration_file.md` into the generated migration file.

## Run Migrations

```bash
bin/rails db:migrate
```

Verify the `db/schema.rb` file is updated correctly.

## Create ActiveRecord Models

Generate models for each table:

```bash
bin/rails g model Goal name:string description:text
bin/rails g model Allergy name:string
# ... and so on for all tables
```

## Model Associations

Add associations to each model file. See detailed model implementations in the `models/` directory.

## Seed Initial Data

Create seed data in `db/seeds.rb` for lookup tables:

```ruby
# Clear existing data (optional, for development)
puts "Destroying existing data..."
UserGoal.destroy_all
UserAllergy.destroy_all
# ... destroy other join tables first, then master tables
Goal.destroy_all
Allergy.destroy_all
# ...

puts "Seeding Goals..."
Goal.create!([
  { name: "Weight Loss", description: "Focus on calorie deficit and nutrient-dense foods." },
  { name: "Eat Healthier", description: "Incorporate more whole foods and balanced meals." },
  { name: "Save Money", description: "Budget-friendly recipes and smart shopping." },
  { name: "Learn to Cook", description: "Beginner-friendly recipes and cooking tips." },
  { name: "Meal Prep Efficiency", description: "Recipes suitable for batch cooking and quick assembly." }
])

puts "Seeding Allergies..."
Allergy.create!([
  { name: "Dairy" }, { name: "Eggs" }, { name: "Tree Nuts" }, { name: "Peanuts" },
  { name: "Shellfish" }, { name: "Soy" }, { name: "Gluten" }, { name: "Fish" }, { name: "Sesame" }
])

puts "Seeding Kitchen Equipment..."
KitchenEquipment.create!([
  { name: "Instapot / Pressure Cooker" },
  { name: "Blender" },
  { name: "Food Processor" },
  { name: "Strainer / Colander" },
  { name: "Juicer" },
  { name: "Oven" },
  { name: "Microwave" },
  { name: "Air Fryer" }
])

# ... Seed other lookup tables ...
```

Run the seeds:
```bash
bin/rails db:seed
```

For detailed model implementations, see:
- `models/user.md`
- `models/recipe.md`
- `models/nutrition.md` 