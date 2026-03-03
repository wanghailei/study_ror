# Schema

## Schema

### What are Schema Files for?

==Database remains the source of truth.== The `schema.rb` file serves as a snapshot of the database structure at a given point in time, reflecting all tables, columns, indexes, and foreign keys in a format that can be used to recreate the database. By default, ==Rails generates `db/schema.rb` which attempts to capture the current state of your database schema.==

Schema files are also useful if you want a quick look at what attributes an Active Record object has. This information is not in the model's code and is frequently spread across several migrations, but the information is nicely summed up in the schema file.

Key characteristics of `schema.rb`:

- **Explicit and Low-Level**: It describes the schema using low-level column definitions (e.g., `t.integer`, `t.string`) to precisely match the actual database structure.
- **Database-Agnostic**: It aims to be portable across different database systems (e.g., PostgreSQL, MySQL, SQLite), so it avoids Rails-specific abstractions that might not translate directly to SQL.
- **Generated, Not Edited**: `schema.rb` is not meant to be manually edited; it’s a reflection of the database state after applying migrations.

### Create database with schema.rb

The `bin/rails db:setup` command will create the database, load the schema, and initialize it with the seed data.

### Preparing the Database

The `bin/rails db:prepare` command is similar to `bin/rails db:setup`, but it operates idempotently.

- If the database has not been created yet, the command will run as the `bin/rails db:setup` does.
- If the database exists but the tables have not been created, the command will load the schema, run any pending migrations, dump the updated schema, and finally load the seed data.
- If both the database and tables exist but the seed data has not been loaded, the command will only load the seed data.
- If the database, tables, and seed data are all in place, the command will do nothing.

Once the database, tables, and seed data are all established, the command will not try to reload the seed data, even if the previously loaded seed data or the existing seed file have been altered or deleted. To reload the seed data, you can manually run `bin/rails db:seed`.

### Reset the Database

==The `bin/rails db:reset` command will drop the database and set it up again. This is functionally equivalent to `bin/rails db:drop db:setup`.==

This is not the same as running all the migrations. It will only use the contents of the current db/schema.rb or db/structure.sql file. If a migration can't be rolled back, bin/rails db:reset may not help you.

### Drop and recreate database with discrete steps

Recreating a database from a clean `schema.rb` file provides greater control and transparency. 

Isolate this process to non-production environments by leveraging Rails environments. For testing, consider using `rails db:environment:set` if switching databases.

#### Drop the Existing Database

Execute `rails db:drop`. This removes the current database entirely, including all tables and data.  

**Best Practice**: Verify the environment with `Rails.env` to confirm you are not in production. If necessary, specify the environment using `RAILS_ENV=development rails db:drop`. Always back up critical data beforehand, as this operation is irreversible.

#### Create a New Database

Run `rails db:create`. This initializes a fresh database based on your configuration in `config/database.yml`.  

**Best Practice**: Ensure your database adapter (e.g., PostgreSQL, MySQL) is properly installed and configured. Check for any custom database options, such as encoding or collation, to avoid inconsistencies.

#### Load the Schema

==Use `rails db:schema:load` to populate the new database with the structure defined in `db/schema.rb`.== This is preferable to `db:migrate` for a clean start, as it directly applies the schema without running individual migration files, reducing potential errors from historical migrations.

==Confirm that `schema.rb` is up-to-date and reflects the desired state.==

Commit `schema.rb` and `seeds.rb` to your repository before proceeding. This ensures reproducibility across team members or deployments.

#### Seed the Database

Execute `rails db:seed` to insert predefined data from `db/seeds.rb`. This step is essential if your application relies on initial records, such as default users or configurations.  

**Best Practice**: Keep `seeds.rb` idempotent (e.g., using `find_or_create_by`) to prevent duplicate entries on repeated runs. Test seeds in isolation to ensure they do not introduce validation errors or dependencies on non-existent data.

#### Inspecting

After each step, verify success with tools like Rails console (`rails c`) to inspect the database state. Monitor logs for errors related to permissions, connections, or schema incompatibilities.

### Schema dump

The `schema.rb` file is created by running `rails db:schema:dump`. ==It tends to be faster and less error prone to create a new instance of your application's database by loading the schema file via `bin/rails db:schema:load` than it is to replay the entire migration history.== Migrations, mighty as they may be, are not the authoritative source for your database schema. 

#### Types of Schema Dumps

The format of the schema dump generated by Rails is controlled by the `config.active_record.schema_format` setting defined in config/application.rb. By default, the format is `:ruby`, or alternatively can be set to `:sql`.

#### Using the default :ruby schema

==When `:ruby` is selected, then the schema is stored in `db/schema.rb`.== If you look at this file you'll find that it looks an awful lot like one very big migration:

```ruby
ActiveRecord::Schema[8.1].define(version: 2025_09_06_171750) do
	create_table "authors", force: true do |t|
		t.string   "name"
		t.datetime "created_at"
		t.datetime "updated_at"
	end
	create_table "products", force: true do |t|
		t.string   "name"
		t.text     "description"
		t.datetime "created_at"
		t.datetime "updated_at"
		t.string   "part_number"
	end
end
```

In many ways this is exactly what it is. ==This file is created by inspecting the database and expressing its structure using `create_table`, `add_index`, and so on.==

#### 6.2.2 Using the `:sql` schema dumper

==However, `db/schema.rb` cannot express everything your database may support such as triggers, sequences, stored procedures, etc.==

While migrations may use `execute` to create database constructs that are not supported by the Ruby migration DSL, these constructs may not be able to be reconstituted by the schema dumper.

If you are using features like these, you should set the schema format to `:sql` in order to get an accurate schema file that is useful to create new database instances.

When the schema format is set to `:sql`, the database structure will be dumped using a tool specific to the database into `db/structure.sql`. For example, for PostgreSQL, the `pg_dump` utility is used. For MySQL and MariaDB, this file will contain the output of `SHOW CREATE TABLE` for the various tables.

To load the schema from `db/structure.sql`, run `bin/rails db:schema:load`. Loading this file is done by executing the SQL statements it contains. By definition, this will create a perfect copy of the database's structure.

## Schema vs Migration

### `schema.rb` and a migration share the core DSL.

Merely from a code perspective, they use the same core language. ==Both files use the exact same **Active Record Domain-Specific Language (DSL)** to describe database tables and columns. That's why they look so similar.== Methods like `create_table`, `t.string`, `t.decimal`, `t.references`, and `add_index` are the common vocabulary used in both.

From The main difference is the "wrapper" around the core DSL code. Migrations are wrapped in a Class. `schema.rb` is wrapped in a `define` block.

A migration is a full Ruby class designed to execute a change. It has a `change` method that tells Rails what to do. The schema is not a class to be executed but a single block that **defines** the entire database structure at once. It's a machine-generated script.

```ruby
# A migration file
class CreateProducts < ActiveRecord::Migration[8.1]
    def change
        create_table :products do |t|
            t.string :name
            t.string :codename
            t.timestamps
        end
    end
end

# The generated schema.rb file
ActiveRecord::Schema[8.1].define(version: 20251007145000) do
    create_table "products", force: :cascade do |t|
        t.string "name"
        t.string "codename"
        t.datetime "created_at", null: false
        t.datetime "updated_at", null: false
    end
end
```

They have totally diferent purposes. The `schema.rb` is like a clean, final export, while migrations are the raw, editable source files. A **migration tells a story of change** (`class DoSomething`), while the **schema states the final fact** (`define what exists`). 

### schema.rb vs migration - foreign key / `t.references`



> In schema.rb, why use `t.product_id` instead of `t.reference product`, like in migration?

In a Ruby on Rails `schema.rb` file, you may notice that tables are defined with explicit column types like `t.integer "product_id"` instead of using `t.references :product` as seen in migration files. This difference arises due to the distinct purposes and conventions of `schema.rb` versus migration files. 

#### Why `t.references :product` in Migrations?

In contrast, migration files (e.g., `db/migrate/20251008120000_create_formulas.rb`) are written by developers to define *how* to modify the database schema incrementally. The `t.references` method is a Rails helper designed to simplify the creation of foreign key columns and associated metadata in migrations.

Key characteristics of `t.references`:

- **Higher-Level Abstraction**: `t.references :product` is a shorthand that automatically creates an `integer` (or `bigint`) column named `product_id` and optionally adds an index or foreign key constraint, depending on options like `index: true` or `foreign_key: true`. 

- **Developer Convenience**: `t.references` reduces boilerplate in migrations by combining the column creation and relationship setup into one line. It also aligns with Rails’ convention of associating models (e.g., `belongs_to :product` in the `Formula` model).

- **Not Needed in `schema.rb`**: Since `schema.rb` is the *output* of migrations, it doesn’t need the abstraction of `t.references`. Instead, it directly represents the resulting database structure (e.g., the `product_id` column and any foreign key constraints).

#### Why `t.integer "product_id"` in `schema.rb`?

==In `schema.rb`, foreign key relationships are represented explicitly as columns (e.g., `t.integer "product_id"`) rather than using higher-level Rails abstractions like `t.references`.== 

The use of `t.integer "product_id"` in `schema.rb` reflects the actual database column for a foreign key, ensuring precision and portability, while ==`t.references :product` in migrations is a Rails helper== for convenient schema creation. The distinction arises because `schema.rb` is a low-level, database-accurate snapshot, whereas migrations use high-level abstractions for developer ease. 

#### Why not use `t.references` in `schema.rb`?

Using `t.references` in `schema.rb` would introduce several issues:

- **Loss of Precision**: ==`t.references` is a Rails abstraction that assumes conventions (e.g., column named `product_id`, integer type). If the database uses a different column name or type (e.g., `product_uuid` as a UUID), `t.references` wouldn’t accurately reflect it.==
- **Dependency on Rails**: `schema.rb` is meant to be a standalone description of the database schema, usable even without Rails’ full context. Using `t.references` would tie it to Rails’ migration logic.
- **Redundancy**: ==Since `schema.rb` already includes `add_foreign_key` for relationships, `t.references` would add unnecessary complexity.==

#### Examples

```ruby
class CreateFormulas < ActiveRecord::Migration[7.0]
    def change
        create_table :formulas do |t|
            t.references :product, foreign_key: true
            t.timestamps
        end
    end
end
```

This creates a `formulas` table with a `product_id` integer column, an index on `product_id`, and a foreign key constraint to `products`. Resulting `schema.rb`:

```ruby
create_table "formulas", force: :cascade do |t|
    t.integer "product_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["product_id"], name: "index_formulas_on_product_id"
end
add_foreign_key "formulas", "products"
```

The `schema.rb` explicitly lists the `product_id` column as `t.integer` and separates the foreign key constraint, reflecting the database’s actual structure.

#### Key Differences Summarized

| Aspect                | Migration (`t.references :product`)          | `schema.rb` (`t.integer "product_id"`)               |
| --------------------- | -------------------------------------------- | ---------------------------------------------------- |
| **Purpose**           | Defines how to create or modify the schema   | Represents the current schema state                  |
| **Abstraction Level** | High-level Rails helper                      | Low-level, database-accurate                         |
| **Column Definition** | Implicitly creates `product_id` as integer   | Explicitly defines `product_id` as integer           |
| **Foreign Key**       | Can add foreign key with `foreign_key: true` | Foreign key defined separately via `add_foreign_key` |
| **Editable**          | Written and edited by developers             | Auto-generated, not meant to be edited               |
| **Portability**       | Rails-specific syntax                        | Database-agnostic, reflects actual SQL               |

#### Practical Implications

- **For Developers**: ==When writing migrations, use `t.references :product` for convenience and to align with Rails conventions== (e.g., `belongs_to :product` in models). The `schema.rb` will automatically reflect this as `t.integer "product_id"` after running `rails db:migrate`.
- **For Database Setup**: Use `rails db:schema:load` to recreate the database from `schema.rb`. The explicit `t.integer "product_id"` ensures the column is created exactly as it exists in the database.
- **For Debugging**: ==If you see `t.integer "product_id"` in `schema.rb` without a corresponding `add_foreign_key`, it might indicate a missing foreign key constraint in your migration== (e.g., you used `t.integer :product_id` instead of `t.references :product, foreign_key: true`).

#### Additional Notes

- **Custom Column Names**: If you used a non-standard column name in your migration (e.g., `t.references :product, foreign_key: { to_table: :products, column: :prod_id }`), `schema.rb` would show `t.integer "prod_id"`, reflecting the actual column name.
- **Database-Specific Types**: In some cases (e.g., PostgreSQL with `id` as `bigint`), `schema.rb` might use `t.bigint "product_id"` to match the database’s column type.
- **Best Practice**: Always include `foreign_key: true` in `t.references` in migrations to enforce referential integrity, which will appear as `add_foreign_key` in `schema.rb`.

### schema.rb vs migration - adding index

Both syntaxes achieve the exact same result, but they are used in different contexts:

**`add_index "materials"`** (migration syntax): This is the syntax you typically write in a migration file. It's a command that explicitly says, "Go find the 'materials' table and add an index to it."

**`t.index ["manufacturer_id"]`** (schema.rb syntax): When Rails generates the `schema.rb` file, for efficiency and clarity, it defines a table and all of its associated indexes together in one consolidated block. So, `t.index` is a shorthand that means, "Add an index to *this table that we are currently creating*."

### `force: :cascade` in schema.rb (not in migrations)

> ```ruby
> ActiveRecord::Schema[8.1].define(version: 2025_10_08_220000) do
> 	create_table "units", force: :cascade do |t|
> ```
>
> Why each table has `force: :cascade`?

That `force: :cascade` you see in every `create_table` block inside schema.rb isn’t something you wrote — it’s automatically generated by Rails.

`force: :cascade` tells Rails what to do if the table already exists when you reload or rebuild the schema (for example, during db:schema:load).

`:cascade` means “drop this table first, cascading any dependent foreign-key constraints, before recreating it.”In plain English: *If the table already exists, drop it completely (even if other tables reference it), then recreate it exactly as defined below.*

#### Purpose

When you run `rails db:schema:load`, Rails doesn’t use migrations — it just rebuilds the database directly from schema.rb. To make sure the schema loads cleanly even if some tables already exist (or have foreign key dependencies), Rails writes:

```ruby
create_table "units", force: :cascade do |t|
```

This ensures the table will be dropped safely before being re-created, rather than causing “table already exists” or “foreign key constraint” errors. So it’s purely a safety mechanism for clean schema regeneration — not used in your migrations themselves.

==`force: :cascade` only matters when *loading* a schema (like db:schema:load or db:setup). It doesn’t affect runtime, and migrations ignore it.== In production, you usually rely on migrations, not schema loading, so it never drops your data unexpectedly.



## Keep Schema and Kill Migrations

> What if I directly modify the schema.rb and delete all the migrations?

Gemini: Yes, you can. In fact, given your philosophy of "Order" and simplicity, this is often the superior approach during the early stages of architecture design.

This practice is essentially "Schema Squashing." It discards the messy historical narrative of *how* the database evolved (the "becoming") and retains only the pure definition of what the database *is* (the "being"). A directory with 50 iterative migration files is often just historical noise.

**The Prerequisite**

You must be willing to reset (wipe) your local database.

==Since `schema.rb` is the blueprint, modifying it doesn't change the actual `.sqlite3` file on your disk. You must regenerate the database file to match your edited blueprint.==

**The Constraints**

If you have a live Production database that contains critical business data, this is dangerous. Without migration files, Rails loses the incremental instructions needed to upgrade the production schema safely. You would likely have to manually patch the production database structure to match your new schema.

**The Procedure for Order**

1. **Delete** all files in `db/migrate/`.
2. **Edit** `db/schema.rb` directly: remove the redundant index, align column names, and ensure the structure is aesthetically perfect.
3. **Run** `rails db:reset` (or `rails db:schema:load`). This drops your existing database and builds a pristine new one strictly based on your edited `schema.rb`.

You now have a clean slate with zero historical baggage.

