# Active Record

Smart models by design

* &#x20;﻿﻿"Fat model, skinny controller"
* Move most business logic to models
* Define instance methods for things an instance of a class should "know"
* Models can leverage attributes and associations

Model performs two key roles: one for representing the state of the application or business, two it communicates with the database.

State Representation: It represents the data and the rules (business logic) that govern access to and updates of this data. In other words, it maintains the state of the application.

Database Communication: The Model is responsible for all interactions with the database, namely CRUD. The Model uses an Object-Relational Mapping (ORM) system to abstract the low-level database commands into higher-level programming language methods or functions.

==The purpose of separating these responsibilities into a Model component is to isolate the business logic and data manipulation logic of an application.== Changes in the graphical user interface (handled by the View) or changes in the way user inputs are processed (handled by the Controller) do not affect the business rules or the database communication logic, and vice versa.

## Active Record

Active Record was described by Martin Fowler in his book _Patterns of Enterprise Application Architecture_: An object that wraps a row in a database table or view, encapsulates the database access, and adds domain logic on that data. An object carries both data and behaviour. Much of this data is persistent and needs to be stored in a database. Active Record uses the most obvious approach, putting data access logic in the domain object.

In Active Record, objects carry both persistent data and behaviour which operates on that data.

==Active Record automatically defines model attributes for every table column, so you don't have to declare those attributes in your model file.==

Active Record gives us several mechanisms, the most important being the ability to:

\- Represent models and their data.

\- Represent associations between these models.

\- Represent inheritance hierarchies through related models.

\- Validate models before they get persisted to the database.

\- Perform database operations in an object-oriented fashion.

### ActiveModel

ActiveRecord is an Object-Relational Mapper so that each of your objects represents a database row. ActiveModel is the interface that Rails uses to all of storage including non-relational stores like Cassandra or MongoDB, to fit particular object types into Rails.

## Associations

In Rails, an *association* is a connection between two Active Record models. Why do we need associations between models? Because they make common operations simpler and easier in your code.

Rails supports six types of associations, each with a particular use-case in mind.

- `belongs_to`
- `has_one`
- `has_many`
- `has_many :through`
- `has_one :through`
- `has_and_belongs_to_many`



### `belongs_to`

A `belongs_to` association sets up a connection with another model, such that each instance of the declaring model "belongs to" one instance of the other model. 

The `belongs_to` associations *must* use the singular term. If you used the pluralized form in the above example for the `author` association in the `Book` model and tried to create the instance by `Book.create(authors: @author)`, you would be told that there was an "uninitialized constant Book::Authors". This is because Rails automatically infers the class name from the association name. If the association name is wrongly pluralized, then the inferred class will be wrongly pluralized too.

In general, ==the `belongs_to` association should be defined on the table that holds the foreign key, while the `has_one` association should be defined on the table that holds the primary key==. This is because the foreign key is a reference to the primary key in the other table, and the `belongs_to` association is used to define this relationship.

For example, if you have a `User` model and an `Address` model, where each user has one address, you would define the `belongs_to` association on the `Address` model.

```ruby
class User < ApplicationRecord
     has_one :address
end

Class Address < ApplicationRecord
	belongs_to :user
end
```

There are 4 kinds of relationships: `belongs_to`, `has_one`, `has_many`, `has_many :through`.

###

In Rails, belongs_to is used to set up associations on the model that contains the foreign key column. belongs_to is not just for one-to-one relationships. 

In a one-to-one relationship, you will often use belongs_to and has_one together. The model that belongs_to another contains the foreign key.

```ruby
class User < ApplicationRecord
	has_one :profile
end

class Profile < ApplicationRecord
	belongs_to :user
end
```

In this example, a User has one Profile, and Profile belongs to User. The profiles table would have a user_id foreign key column.

In a one-to-many relationship, you will often use belongs_to and has_many together. The model that belongs_to another contains the foreign key.

```ruby
class Author < ApplicationRecord
       has_many :books
end

class Book < ApplicationRecord
       belongs_to :author
end
```

In this example, an Author can have many Books, and each Book belongs to an Author. The books table would have an author_id foreign key column.

So, belongs_to can be used to set up both one-to-one and one-to-many relationships, depending on whether the associated model uses has_one or has_many.

### `has_many`



```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end

class Book < ApplicationRecord
  belongs_to :author
end
```

### Relationships need be defined in Models

An example with Employee, Position, Department and Appointment as models.

% Appointment is a join table for Employee and Position and it actually could be named as EmployeePosition. 我認為應該創建這種 ==meaningful join-tabel==，而不是硬生生把兩個 object 連起來。 20240119 %

```ruby
# app/models/department.rb
class Department < ApplicationRecord
	has_many :employees
     has_many :positions
end
```

In general, when you're setting up a `has_many :through` association, first you create a direct one-to-many association, and then you "go through" that association to set up the many-to-many relation. So both lines are essential for establish gving a many-to-many relationship through a join model in Rails.

```ruby
# app/models/employee.rb
class Employee < ApplicationRecord
	belongs_to :department, optional: true
	has_many :appointments
	has_many :positions, through: :appointments
end
```

`has_many :appointments`: This line sets up a one-to-many relationship between `Employee` and `Appointment`. It `means` an employee can have many appointments. ==This is important because it allows access to `appointment` model's properties, and without it, the `through:` option cannot be used.==

==`has_many :positions, through: :appointments`: This line establishes the many-to-many relationship between `Employee` and `Position`.== The `through: :appointments` part tells Rails to use the `appointments` table as a join-table to facilitate this relationship.

```ruby
# app/models/position.rb
class Position < ApplicationRecord
	belongs_to :department, optional: true
	has_many :appointments
	has_many :employees, through: :appointments
end
```

The `optional: true` parameter is added to indicate that these associations are not mandatory, meaning an `employee` can exist without being related to a `position` or a `department`. If you want to enforce the presence of these associations, simply remove the `optional: true` parameter.

The `Appointment` model helps facilitate the many-to-many relationship between `Employee` and `Position`. ==% Join-tables 都是 belongs_to, belongs_to 的。 20240119 %==

```ruby
# app/models/appointment.rb
class Appointment < ApplicationRecord
	belongs_to :employee
	belongs_to :position
end
```

### Relationships also need be defined in Migration

Besides defining the relationship in ActiveRecord, you also need to define the relationships in migration files when setting up your database schema in Rails. 

```ruby
# Migration for the Employee model
class CreateEmployees < ActiveRecord::Migration[7.1]
	def change
		create_table :employees do |t|
			t.string :name
			t.references :department, null: false, foreign_key: true
			t.references :position, null: false, foreign_key: true
			t.timestamps
		end
	end
end
```

In the `Employee` migration, `t.references` adds a `department_id` and `position_id` to the `employees` table that will be used as foreign keys. The `foreign_key: true` makes these references to actually act as foreign keys.

Please note that `null: false` option in the references makes the association mandatory. If you want `department_id` and `position_id` to be optional, you would remove `null: false`.

```ruby
# migration creating the Appointment join-table
class CreateAppointments < ActiveRecord::Migration[7.1]
	def change
		create_table :appointments do |t|
			t.references :employee, null: false, foreign_key: true
			t.references :position, null: false, foreign_key: true
			t.timestamps
		end
	end
end
```

### Add references (foreign key relationships)

% 經常需要後續添加 reference（foreign key relationships）。單獨創建 migration 來做這件事。 20240120 %

```ruby
class EmployeesReferencesPositionAppointmentDepartmentAddition < ActiveRecord::Migration[7.1]
	def change
		add_reference :employees, :position, null: false, foreign_key: true
		add_reference :employees, :appointment, null: false, foreign_key: true
		add_reference :employees, :department, null: false, foreign_key: true
	end
end
```

Foreign keys additionally add an index to that column for DB performance. 

#### While it's not required, you might want to add foreign key constraints to guarantee referential integrity.

==In Rails, foreign key constraints are not strictly required because Rails provides its own mechanisms to maintain referential integrity at the application level.== 

==ActiveRecord Associations allow Rails to infer the foreign key relationships from the model definitions, without having to explicitly enforce them at the database level.==

Not all databases support foreign key constraints, therefore Rails doesn't require them as part of its standards.

However, adding foreign key constraints at the database level provides an added level of data protection and can prevent race conditions or inadvertent errors from corrupting data relationships, such as running SQL commands directly on the database outside of Rails or concurrent writes to the database.

#### `foreign_key: true`

A Foreign Key, on the other hand, is a column or a set of columns in a table that is used to establish a link between the data in two tables. ==It's a field in one table, that uniquely identifies a row of another table. The foreign key in one table is the primary key of another table.==

For example, consider a posts table in a blog database. Each post might be associated with a user. The posts table might have a `user_id` column as a foreign key. This `user_id` is used to associate each post with a user. The `user_id` in `posts` table refers to the id `in` the `users` table.

The main purpose of foreign keys is to enforce a certain level of data integrity in the database. If a foreign key constraint is set up, it prevents actions that would destroy links between tables. For example, ==a database wouldn't allow a row containing a foreign key to be deleted if there is a row in the linked table that references it.==

==In Rails Migration, when `foreign_key: true` is specified, it means that you're adding a foreign key constraint at the database level.==

```ruby
def change
	add_reference :articles, :author, foreign_key: true
end
```

In this migration, you're adding a new column `author_id` to the `articles` table. The `author_id` will be used to link each article to a specific author. `foreign_key: true` adds a foreign key constraint to this `author_id` column.

==The foreign key constraint ensures referential integrity, which means the database will only allow you to enter an `author_id` in the `articles` table if it refers to an existing `author` record in the `authors` table.== If you try to enter an `author_id` that doesn't exist in the `authors` table, or if you try to delete an `author` record that's associated with an `article` record, the database will raise an error. ==In this way, `foreign_key: true` helps to preserve the integrity of your data.==

In Rails 5 and onward, `foreign_key: true` by default also implies an index on this column, which helps speed up queries involving this foreign key. ==Note that it's not enough just to have the foreign key column in your model - you should add the foreign key at the database level with a migration to ensure referential integrity.==

#### Potential issues that might occur:

**Nonexistent Tables**: If `employees`, `position`, `appointment`, or `department` tables do not exist, then this migration will fail during runtime. % 所以，==在運行添加 reference 的 migration 時，首先要確保相關的表都已經創建了。== 20240120 %

**Existing Columns**: If `employees` already has `position_id`, `appointment_id` or `department_id` column, the migration will fail. % 是不是在創建 migration（尤其用 rails g migration）時，分兩步走比較好？首先創建獨立的基本表，然後另起一個 migration 創建 reference。 20240120 %

**Nonexistent Values**: If `null: false` was set, ensure that you have the necessary data for all existing records. If not, existing records without these foreign key values would not be valid.

**Locking**: A very large number of indexes may negatively impact write performance. If you have a significant number of records, adding an index can lock your table for a time proportional to the number of records. 

### Setting null: false

==Setting `null: false` implies that these fields must always have a value==. By default, Rails sets `null: true` if you do not specify the `null` option.  

If you define a column with `null: true` or without specifying `null: false` in a migration in Rails, it means that the column is allowed to have `NULL` values. Therefore, in this case, the `position_id`, `appointment_id`, and `department_id` columns in the `employees` table could have `NULL` values. In other words, it would be legal for an `employee` record to not have associated `position`, `appointment`, or `department`.

#### Is setting `null: true` a good thing?

Whether to set null: true or null: false largely depends on the business requirements and the domain model of your application.

Setting null: false enforces that the column must always contain a value, i.e., it cannot be null. This is typically used when you know that a column should always have a value for every record in the table.
On the other hand, null: true is the default setting which allows NULL values in the column. This means that it's not mandatory for every record to have a value for this column.

Here are a few things to consider when choosing between null: true and null: false:
Data Integrity and Consistency: Setting null: false promotes data integrity and consistency since it enforces every record to provide a value for the column.

Business Rules: If, according to the business rules, a field must always contain data, then setting null: false is the right choice. For instance, an employees table should probably have a name field that is always filled, thus null: false would be appropriate.

Flexibility: Setting null: true can provide flexibility. There might be circumstances where you do not have all the data at the time of creating a record. In such cases, allowing NULL values in certain columns could be beneficial.

Performance: Allowing NULL values (null: true) can slightly increase the performance of the database when querying data because SQL databases can optimize NULL storage and searches.

Weigh these considerations in terms of your specific requirements. For instance, if your employees can exist without associated position, appointment, or department, then setting null: true for position_id, appointment_id, and department_id would seem appropriate. However, if these associations are mandatory according to your business rules, then you should enforce null: false to ensure data integrity.

### Example 2

For three models, Position, Responsibilty and Capability. A Position shall have mulitiple responsibilities and capabilities. A responsibility might be shared by many positions, such as all directors should be able to lead team members well. A common capability might be required by different positions, such as critical thinking. How to define the relationships in ActiveRecord?

Based on the need, there are many-to-many relationships between Position and Responsibility, as well as between Position and Capability. `has_many :through` is used to set up many-to-many relationships. You'll also need *join models* (for example, PositionResponsibilities and PositionCapabilities).

```ruby
# model: position.rb
class Position < ApplicationRecord
     has_many :responsibilities, through: :position_responsibilities
	has_many :capabilities, through: :position_capabilities
     has_many :position_responsibilities
     has_many :position_capabilities
end

# model: responsibility.rb
class Responsibility < ApplicationRecord
	has_many :position_responsibilities
	has_many :positions, through: :position_responsibilities
end

# model: capability.rb
class Capability < ApplicationRecord
	has_many :position_capabilities
	has_many :positions, through: :position_capabilities
end

# model: position_responsibilities.rb
class PositionResponsibility < ApplicationRecord
	belongs_to :position
	belongs_to :responsibility
end

# model: position_capabilities.rb
class PositionCapability < ApplicationRecord
	belongs_to :position
	belongs_to :capability
end
```

Now for the migrations, you will have to create the join models `PositionResponsibility` and `PositionCapability`:

```ruby
# migration creating the PositionResponsibility join-table
class CreatePositionResponsibilities < ActiveRecord::Migration[7.1]
	def change
		create_table :position_responsibilities do |t|
			t.references :position, null: false, foreign_key: true
			t.references :responsibility, null: false, foreign_key: true
			t.timestamps
		end
	end
end

# migration creating the PositionCapability join-table
class CreatePositionCapabilities < ActiveRecord::Migration[7.1]
	def change
		create_table :position_capabilities do |t|
			t.references :position, null: false, foreign_key: true
			t.references :capability, null: false, foreign_key: true
			t.timestamps
		end
	end
end
```

These models and migrations will establish the relationships, where a position can have multiple responsibilities and capabilities, and a responsibility or capability can be associated with multiple positions.

## Scopes

% 我發現 scope 確實很好用。 %

```ruby
# For a three levels self-referencial model, viewed as a dropdown and leveled list.
scope :level_one, -> { where(parent_id: nil) }
scope :level_two, -> { joins(:parent).where(parent: { parent_id: nil }) }
scope :level_three, -> { joins(parent: :parent).where(parent: { parent: { parent_id: nil } }) }
```

For level two categories, you need to find categories whose `parent_id` is not nil (so they have a parent) but whose parent's `parent_id` is nil (so their parent is a level one category).

- Uses `joins(:parent)` to join with the parent category.
- Checks that the parent's `parent_id` is nil (meaning the parent is level one)
- This automatically excludes level one categories (since they don't have a parent to join with)

The `level_three` scope joins through two levels of parents to ensure the grandparent is a level one category.

### Why Use Lambdas in Scopes?

In Rails scopes, lambdas are used because they are **evaluated lazily** (when called), not when defined:

```ruby
# BAD - evaluated once when class loads
scope :recent, where("created_at > ?", 1.week.ago)

# GOOD - evaluated each time the scope is called
scope :recent, -> { where("created_at > ?", 1.week.ago) }
```

This matters for time-dependent queries. Without the lambda, `1.week.ago` would be calculated once when the Rails app starts, not each time you call the scope.

```ruby
scope :level_one, -> { where(parent_id: nil) }
```

This creates a scope named `level_one` that returns a lambda. When you call `Category.level_one`, Rails executes the lambda and returns the query `where(parent_id: nil)`.

## Other things

### Getting Column Statistics

==Rails has the ability to perform statistics on the values in a column.==

### Deleting Associated Objects

If you delete an article, its associated comments will also need to be deleted. Rails allows you to use the _dependent_ option of an association to achieve this.

## How is Model communicate with the Database specifically?

The Model interacts with the database through ActiveRecord.

==When a Controller needs to interact with data, it asks the Model to do so. The Model uses ActiveRecord methods to create SQL queries.==

* ActiveRecord then sends the SQL query to the database for execution.
* The database returns the data, and ActiveRecord converts this raw data into Ruby objects that the application can work with. These objects are instances of the Model class. For example, each row in the users table is represented as an instance of the User model.
* If the data needs to be modified, the Controller makes changes to the Model's objects, and then it calls a method (like save or update) to write these changes back to the database.
* ActiveRecord generates the appropriate SQL queries for these operations as well.

ActiveRecord also handles relationships between different models (like has\_many, belongs\_to, etc.). This lets you manage related data across different tables in a more intuitive and Ruby-like way, rather than writing complex SQL queries.

Before writing data back to the database, ActiveRecord performs any validations defined in the Model. If the data is invalid (for example, if a required field is missing), the save or update operation is cancelled, and the invalid data is not written to the database.

==So while it seems like the Model is communicating directly with the database, it's actually ActiveRecord that does most of the work behind the scenes.==

**WHL: Are the states of the business represented by the Model stored in the RAM of the server?**

ChatGPT: The Model in an MVC architecture doesn't typically store business state in server RAM in the way that you might be imagining. Instead, the Model represents the structure of the data and the rules for interacting with that data. ==The actual data, or "state", is usually stored in a database.==

However, when your application is running and an instance of a Model is created, that instance and its data are temporarily stored in the server's RAM. This data remains in RAM only for the duration of the request. Once the request is completed and the response has been sent, the data for that request is discarded to free up memory for other requests. Persistent storage is managed by the database.

It's important to note that some applications do store certain data in server memory for quick access, such as session data or cache data.





