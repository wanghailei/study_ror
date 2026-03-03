# Principles and Professional way of data migrations

## God Migration - Initialise



To intialise or re-intialise a db, run:

```
bin/rails db:migrate:reset
```

**Why this works:** `db:migrate:reset` performs `db:drop` + `db:create` + **`db:migrate`**. It ignores the `schema.rb` (the "ghost") and rebuilds the database using your new migration code (the "truth").

db:seed







































Here is the **real, professional** way Rails teams at places like **Basecamp, Shopify, GitHub, Stripe, Zendesk** structure data migrations. This is distilled from years of production experience and is considered the *gold standard*.

### Schema changes and data changes must NEVER mix.

Each migration must do **only one thing**, cleanly. One migration = one purpose

Examples:

- migration 1 → create table
- migration 2 → copy data
- migration 3 → remove old column

## Never use app models in migrations

Because app models change in the future, but the database at the migration point does not.

So they use:

```ruby
class MigrationUser < ApplicationRecord
	self.table_name = "users"
end
```

This model is “frozen” and safe.

## Migrations must remain runnable forever.

Meaning:

- `rails db:setup` must always work
- `rails db:migrate` must work even with an empty DB
- Migrations must not depend on future code.
- Migrations must not assume the current schema.

This is why using current models is dangerous.

## Avoid irreversible destructive operations

But if data migration is destructive, mark down as:

```ruby
raise ActiveRecord::IrreversibleMigration
```



# Folder Structure in Big Rails Companies



Big companies often keep:

```
db/migrate/
db/post_migrate/
lib/data_migrations/
```

Where:

- db/migrate/ = pure schema changes
- db/post_migrate/ = data fixes after deployment (GitLab style)
- lib/data_migrations/ = reusable tasks

But baseline Rails apps keep everything in db/migrate using carefully separated migrations.

# Migration Flow Used by Pros

Example: “Move product/time fields from formulas to formulations”

### **Migration A — Create New Structure**

Contains only:

- create_table :formulations
- indexes

No touching data.

### **Migration B — Migrate Data**





Contains only:



- anonymous AR classes
- find_each
- insert into formulations





No schema changes.





### **Migration C — Remove Old Columns**





Contains only:



- remove_column
- dropping constraints
- dropping old indexes





------





# **4. How They Write the Data Migration**





A real industry-standard migration:

```
class BackfillUserProfiles < ActiveRecord::Migration[7.1]
  class MigrationUser < ApplicationRecord
    self.table_name = "users"
  end
  
  class MigrationProfile < ApplicationRecord
    self.table_name = "profiles"
  end

  def up
    MigrationUser.find_each do |u|
      MigrationProfile.create!(
        user_id: u.id,
        full_name: "#{u.first_name} #{u.last_name}"
      )
    end
  end

  def down
    raise ActiveRecord::IrreversibleMigration
  end
end
```



### **Important:**





This migration:



- doesn’t use User (your evolving app model)
- avoids model validations
- avoids callbacks
- avoids scopes
- only sees the schema as it exists during this migration





This makes it **forever safe**.



------





# **5. What They NEVER Do**







### **A. Never use real models**





Because:



- validations may reject old data
- callbacks expect columns that don’t exist yet
- enums rely on fields that aren’t migrated yet
- scopes refer to other tables
- associations break if tables don’t exist yet





This is the #1 cause of broken historical migrations.





### **B. Never load unnecessary Rails code**





Migrations should not run business logic.





### **C. Never write complex logic inside change**





Use:

```
def up
def down
```

for data migrations.





### **D. Never rely on future code changes**





Your 2025 migration must run in 2035.





### **E. Never make migrations depend on environment**





They must run the same in:



- dev
- test
- staging
- production





------





# **6. Additional Best Practices From Rails Experts**







### **A. “Migrations are snapshots”**





Meaning: each migration must assume *only what the DB looks like at that time*.





### **B. “Long-running tasks need batching”**





Production databases may have millions of rows.



Use:

```
find_in_batches
```



### **C. “No destructive change without data copy first”**





e.g., always copy data → verify → remove old column





### **D. “Migrations should be boring”**





Simple, predictable, stable.





### **E. “Every migration should be reversible unless impossible”**





------





# **7. Example: How your BOS formulation refactor should look (final picture)**







### **Migration A: create_formulations.rb**



```
create_table :formulations do |t|
  t.references :product, null: false
  t.references :formula, null: false
  t.date :valid_since, null: false
  t.date :valid_until
  t.timestamps
end
```



### **Migration B: migrate_formulas_to_formulations.rb**



```
class MigrationFormula < ApplicationRecord
  self.table_name = "formulas"
end

class MigrationFormulation < ApplicationRecord
  self.table_name = "formulations"
end

def up
  MigrationFormula.find_each do |f|
    next if f.product_id.nil?
    MigrationFormulation.create!(
      product_id: f.product_id,
      formula_id: f.id,
      valid_since: f.effective_from || f.created_at.to_date,
      valid_until: f.effective_to
    )
  end
end
```



### **Migration C: cleanup_formulas.rb**



```
remove_column :formulas, :product_id
remove_column :formulas, :effective_from
remove_column :formulas, :effective_to
```

**This is exactly what Shopify and GitHub would do.**



------



If you want, I can show you the **exact before/after schema.rb**, or the **final three migrations** ready for your repo, with zero risk of future failure.





In migrations, using app models is dangerous. Because migrations run **at different times**, and your *app models change over time*. Migrations are meant to run safely **years later**, but app models always reflect the **current** codebase. This mismatch easily breaks migrations.

e.g., Suppose you write a migration, and a year later, you rename a column, rename a model, change validations, add a required association, or change default scopes. Suddenly, your old migration **no longer works**.And the next time you run db:setup or db:migrate, Rails will crash because the migration tries to use the **new** Formula model against the **old** schema. This is the most common cause of “broken historical migrations” in real apps.

#### 3. How dedicated migration classes solve this

If you write:

```erb
class MigrationFormula < ApplicationRecord
	self.table_name = "formulas"
end
```

This object has **no validations, no callbacks, no scopes, no associations** unless you explicitly add them. It reflects only the **database table itself** at the moment the migration runs.

This means:

- It will successfully operate on the table even if your future app code changes
- It will not load current validations (which may block saving old data)
- It will not run callbacks (which might expect missing columns)
- It will not trigger associations or default scopes

It is **stable and safe for data migrations**.



------





# **4. Real example that often breaks migrations**



Your current Formula model has:

- required associations (belongs_to :output_unit)
- validations (presence: true)
- enum status
- maybe callbacks

But at the time the migration runs, the table formulas might **not yet have** the status column, or the output_unit_id column, or the validations might reject old data.

Then this will *fail*:

```
Formula.find_each do |formula|
  ...
end
```

Because:

- validations block saving
- callbacks expect missing columns
- associations expect missing tables
- the enum status refers to a column that may not exist yet

This is why using app models is fragile.



------





##### 5. The dedicated class avoids ALL of that. 

It is a “dumb” ActiveRecord class tied only to the table name:

```
class MigrationFormula < ApplicationRecord
  self.table_name = "formulas"
end
```

- No validations
- No callbacks
- No associations
- No enums
- No scopes
- No business logic

It only knows the table schema.

This is precisely what you want during a data migration.



------





# **6. Rails Core Team recommendation**

Rails core maintainers recommend:

> **Never use application models in migrations. Use anonymous classes instead.**

This is the standard practice in Basecamp, GitHub, Shopify, Zendesk, Stripe Rails codebases.

Every Rails pro follows this rule.



------





# **7. Summary — The real reason**



App models evolve; migrations don’t. Therefore, app models in migrations will eventually break.

Dedicated migration classes:

- always match the table structure at that migration snapshot
- never break because of future code changes
- are safe to use even 10 years later
- avoid validations, callbacks, default scopes, enums
- make migrations deterministic and stable





Using dedicated migration classes is the **correct** and **future-proof** Rails practice for any serious system.