# `belongs_to` Association

==In database terms, the `belongs_to` association says that this model's table contains a column which represents a reference to another table.== This can be used to set up one-to-one or one-to-many relations, depending on the setup. If the table of the other class contains the reference in a one-to-one relation, then you should use `has_one` instead.

## Methods Added by `belongs_to`

==When you declare a `belongs_to` association, the declaring class automatically gains 8 methods related to the association==:

- `association`
- `association=(associate)`
- `build_association(attributes = {})`
- `create_association(attributes = {})`
- `create_association!(attributes = {})`
- `reload_association`
- `reset_association`
- `association_changed?`
- `association_previously_changed?`

In all of these methods, `association` is replaced with the symbol passed as the first argument to `belongs_to`. For example, given the declaration:

```ruby
class Book < ApplicationRecord
	belongs_to :author
end
```

Each instance of the Book model will have these methods:

- `author`
- `author=`
- `build_author`
- `create_author`
- `create_author!`
- `reload_author`
- `reset_author`
- `author_changed?`
- `author_previously_changed?`

When initializing a new `has_one` or `belongs_to` association you must use the `build_` prefix to build the association, rather than the `association.build` method that would be used for `has_many` or `has_and_belongs_to_many` associations. To create one, use the `create_` prefix.

### `association`

The `association` method returns the associated object, if any. If no associated object is found, it returns `nil`.

```ruby
@author = @book.author
```

If the associated object has already been retrieved from the database for this object, the cached version will be returned. To override this behavior (and force a database read), call `#reload_association` on the parent object.

```ruby
@author = @book.reload_author
```

To unload the cached version of the associated object — causing the next access, if any, to query it from the database — call `#reset_association` on the parent object.

```ruby
@book.reset_author
```

##### 4.1.1.2 `association=(associate)`

The `association=` method assigns an associated object to this object. Behind the scenes, this means extracting the primary key from the associated object and setting this object's foreign key to the same value.

```ruby
@book.author = @author
```

##### 4.1.1.3 build_association(attributes = {})

The build_association method returns a new object of the associated type. This object will be instantiated from the passed attributes, and the link through this object's foreign key will be set, but the associated object will not yet be saved.

```ruby
@author = @book.build_author(author_number: 123, author_name: "John Doe")
```

##### 4.1.1.4 create_association(attributes = {})

The create_association method returns a new object of the associated type. This object will be instantiated from the passed attributes, the link through this object's foreign key will be set, and, once it passes all of the validations specified on the associated model, the associated object will be saved.

```ruby
@author = @book.create_author( author_number: 123, author_name: "John Doe" )
```

##### 4.1.1.5 create_association!(attributes = {})

Does the same as create_association above, but raises ActiveRecord::RecordInvalidif the record is invalid.

##### 4.1.1.6 association_changed?

The association_changed? method returns true if a new associated object has been assigned and the foreign key will be updated in the next save.

```ruby
@book.author # => #<Author author_number: 123, author_name: "John Doe">
@book.author_changed? # => false

@book.author = Author.second # => #<Author author_number: 456, author_name: "Jane Smith">
@book.author_changed? # => true

@book.save!
@book.author_changed? # => false
```

##### 4.1.1.7 association_previously_changed?

The association_previously_changed? method returns true if the previous save updated the association to reference a new associate object.

```
@book.author # => #<Author author_number: 123, author_name: "John Doe">
@book.author_previously_changed? # => false

@book.author = Author.second # => #<Author author_number: 456, author_name: "Jane Smith">
@book.save!
@book.author_previously_changed? # => true
```

## Options for belongs_to

While Rails uses intelligent defaults that will work well in most situations, there may be times when you want to customize the behavior of the `belongs_to` association reference. Such customizations can easily be accomplished by passing options and scope blocks when you create the association. For example, this association uses two such options:

```ruby
class Book < ApplicationRecord
	belongs_to :author, touch: :books_updated_at, counter_cache: true
end
```

The `belongs_to` association supports these options:

- :autosave
- :class_name
- :counter_cache
- :default
- :dependent
- :ensuring_owner_was
- :foreign_key
- :foreign_type
- :primary_key
- :inverse_of
- :optional
- :polymorphic
- :required
- :strict_loading
- :touch
- :validate

### :autosave

==If you set the `:autosave` option to `true`, Rails will save any loaded association members and destroy members that are marked for destruction whenever you save the parent object.== Setting `:autosave` to `false` is not the same as not setting the `:autosave` option. If the `:autosave` option is not present, then new associated objects will be saved, but updated associated objects will not be saved.

### :class_name

If the name of the other model cannot be derived from the association name, you can use the `:class_name` option to supply the model name. For example, if a book belongs to an author, but the actual name of the model containing authors is Patron, you'd set things up this way:

```ruby
class Book < ApplicationRecord
	belongs_to :author, class_name: "Patron"
end
```

##### 4.1.2.3 :counter_cache

The `:counter_cache` option can be used to make finding the number of belonging objects more efficient. Consider these models:

```ruby
class Book < ApplicationRecord
	belongs_to :author
end
class Author < ApplicationRecord
	has_many :books
end
```

With these declarations, asking for the value of `@author.books.size` requires making a call to the database to perform a `COUNT(*)` query. To avoid this call, you can add a counter cache to the belonging model:

```ruby
class Book < ApplicationRecord
	belongs_to :author, counter_cache: true
end
class Author < ApplicationRecord
	has_many :books
end
```

With this declaration, Rails will keep the cache value up to date, and then return that value in response to the size method.

Although the `:counter_cache` option is specified on the model that includes the `belongs_to` declaration, the actual column must be added to the associated (`has_many`) model. In the case above, you would need to add a column named `books_count` to the `Author` model.

You can override the default column name by specifying a custom column name in the `counter_cache` declaration instead of `true`. For example, to use count_of_books instead of books_count:

```ruby
class Book < ApplicationRecord
	belongs_to :author, counter_cache: :count_of_books
end
class Author < ApplicationRecord
	has_many :books
end
```

You only need to specify the `:counter_cache` option on the `belongs_to` side of the association.

Counter cache columns are added to the owner model's list of read-only attributes through `attr_readonly`.

If for some reason you change the value of an owner model's primary key, and do not also update the foreign keys of the counted models, then the counter cache may have stale data. In other words, any orphaned models will still count towards the counter. To fix a stale counter cache, use `reset_counters`.

##### 4.1.2.4 :default

When set to `true`, the association will not have its presence validated.

##### 4.1.2.5 `:dependent`

If you set the `:dependent` option to:

- ==`:destroy`, when the object is destroyed, `destroy` will be called on its associated objects.==
- ==`:delete`, when the object is destroyed, all its associated objects will be deleted directly from the database without calling their `destroy` method.==
- `:destroy_async`: when the object is destroyed, an `ActiveRecord::DestroyAssociationAsyncJob` job is enqueued which will call destroy on its associated objects. Active Job must be set up for this to work. Do not use this option if the association is backed by foreign key constraints in your database. The foreign key constraint actions will occur inside the same transaction that deletes its owner.

You should not specify this option on a `belongs_to` association that is connected with a `has_many` association on the other class. Doing so can lead to orphaned records in your database.

##### 4.1.2.6 :ensuring_owner_was

Specifies an instance method to be called on the owner. The method must return true in order for the associated records to be deleted in a background job.

##### 4.1.2.7 :foreign_key

By convention, Rails assumes that the column used to hold the foreign key on this model is the name of the association with the suffix `_id` added. The `:foreign_key` option lets you set the name of the foreign key directly:

```ruby
class Book < ApplicationRecord
	belongs_to :author, class_name: "Patron", foreign_key: "patron_id"
end
```

In any case, Rails will not create foreign key columns for you. You need to explicitly define them as part of your migrations.

##### 4.1.2.8 :foreign_type

Specify the column used to store the associated object’s type, if this is a polymorphic association. By default this is guessed to be the name of the association with a “`_type`” suffix. So a class that defines a belongs_to :taggable, polymorphic: true association will use “taggable_type” as the default :foreign_type.

##### 4.1.2.9 :primary_key

==By convention, Rails assumes that the `id` column is used to hold the primary key of its tables. The `:primary_key` option allows you to specify a different column.==

For example, given we have a users table with guid as the primary key. If we want a separate todos table to hold the foreign key `user_id` in the `guid` column, then we can use `primary_key` to achieve this like so:

```ruby
class User < ApplicationRecord
	self.primary_key = 'guid' # primary key is guid and not id
end

class Todo < ApplicationRecord
	belongs_to :user, primary_key: 'guid'
end
```

When we execute `@user.todos.create` then the `@todo` record will have its `user_id` value as the `guid` value of `@user`.

##### 4.1.2.10 :inverse_of

The :inverse_of option specifies the name of the has_many or has_one association that is the inverse of this association. See the bi-directional association section for more details.

```ruby
class Author < ApplicationRecord
  has_many :books, inverse_of: :author
end

class Book < ApplicationRecord
  belongs_to :author, inverse_of: :books
end
```

##### 4.1.2.11 :optional

==If you set the `:optional` option to true, then the presence of the associated object won't be validated.== By default, this option is set to `false`.

### :polymorphic

Passing `true` to the `:polymorphic` option indicates that this is a polymorphic association.

### :required

When set to `true`, the association will also have its presence validated. This will validate the association itself, not the `id`. You can use :inverse_of to avoid an extra query during validation.

`required` is set to true by default and is deprecated. If you don’t want to have association presence validated, use optional: true.

### :strict_loading

Enforces strict loading every time the associated record is loaded through this association.

### :touch

If you set the `:touch` option to `true`, then the `updated_at` or `updated_on` timestamp on the associated object will be set to the current time whenever this object is saved or destroyed:

```ruby
class Book < ApplicationRecord
    belongs_to :author, touch: true
end

class Author < ApplicationRecord
    has_many :books
end
```

In this case, saving or destroying a `book` will update the timestamp on the associated `author`. You can also specify a particular timestamp attribute to update:

```ruby
class Book < ApplicationRecord
    belongs_to :author, touch: :books_updated_at
end
```

In Rails, touch: true on a belongs_to association tells Active Record to **update the parent record’s updated_at timestamp whenever the child record changes**.

```
belongs_to :board, touch: true
```

#### What exactly happens

Think of `touch: true` as: “This record represents activity **inside** the parent.” With this in place, **any of these actions on the child model** will automatically call `touch` on the associated board: create, update, destroy, and touch (explicitly).

So if you have:

```ruby
class Card < ApplicationRecord
	belongs_to :board, touch: true
end
```

then this:

```ruby
card.update!(title: "New title")
```

will internally do:

```ruby
board.touch
```

which updates:

```ruby
board.updated_at = Time.current
```

and saves it **without running validations**.

#### Why this exists

`touch` is mainly for **cache invalidation and freshness tracking**. Common use cases:

1. Cache keys: Rails cache keys include updated_at. If a card changes but the board’s updated_at does **not** change, the cache stays stale. `touch: true` fixes that.

2. “Last modified” semantics: For things like kanban boards, threads with comments, projects with tasks, you usually want: “The board was updated when *anything inside it* changed.” `touch: true` encodes exactly that idea.

#### `touch: true` vs `touch: :custom_column`

You can be explicit:

```ruby
belongs_to :board, touch: :last_activity_at
```

Then Rails will update **that column instead of `updated_at`**. This is often cleaner for business systems:

```ruby
# Board has both:
# updated_at → schema changes
# last_activity_at → user activity

belongs_to :board, touch: :last_activity_at
```

### :validate

If you set the `:validate` option to `true`, then new associated objects will be validated whenever you save this object. By default, this is `false`: new associated objects will not be validated when this object is saved.

## Scopes for belongs_to

There may be times when you wish ==to customise the query used by belongs_to==. Such customizations can be achieved via a scope block. For example:

```ruby
class Book < ApplicationRecord
	belongs_to :author, -> { where active: true }
end
```

You can use any of the standard querying methods inside the scope block. The following ones are discussed below:

- where
- includes
- readonly
- select

### where

The `where` method lets you specify the conditions that the associated object must meet.

```ruby
class Book < ApplicationRecord
	belongs_to :author, -> { where active: true }
end
```

##### 4.1.3.2 includes

You can use the `includes` method to specify second-order associations that should be eager-loaded when this association is used. For example, consider these models:

```ruby
class Chapter < ApplicationRecord
	belongs_to :book
end

class Book < ApplicationRecord
	belongs_to :author
	has_many :chapters
end

class Author < ApplicationRecord
	has_many :books
end
```

If you frequently retrieve authors directly from chapters (`@chapter.book.author`), then you can make your code somewhat more efficient by including authors in the association from chapters to books:

```ruby
class Chapter < ApplicationRecord
	belongs_to :book, -> { includes :author }
end

class Book < ApplicationRecord
	belongs_to :author
	has_many :chapters
end

class Author < ApplicationRecord
	has_many :books
end
```

There's no need to use `includes` for immediate associations - that is, if you have Book `belongs_to :author`, then the author is eager-loaded automatically when it's needed.

##### 4.1.3.3 readonly

If you use readonly, then the associated object will be read-only when retrieved via the association.

##### 4.1.3.4 select

The `select` method lets you override the SQL SELECT clause that is used to retrieve data about the associated object. ==By default, Rails retrieves all columns.==

If you use the select method on a belongs_to association, you should also set the :foreign_key option to guarantee the correct results.

## Do Any Associated Objects Exist?

You can see if any associated objects exist by using the `association.nil?` method:

```ruby
if @book.author.nil?
	@msg = "No author found for this book"
end
```

## When are Objects Saved?

Assigning an object to a `belongs_to` association does not automatically save the object. It does not save the associated object either.