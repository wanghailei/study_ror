# `has_many` Association

The has_many association creates a one-to-many relationship with another model. In database terms, this association says that the other class will have a foreign key that refers to instances of this class.

## Methods Added by has_many

When you declare a has_many association, the declaring class automatically gains 17 methods related to the association:

- collection
- collection<<(object, ...)
- collection.delete(object, ...)
- collection.destroy(object, ...)
- collection=(objects)
- collection_singular_ids
- collection_singular_ids=(ids)
- collection.clear
- collection.empty?
- collection.size
- collection.find(...)
- collection.where(...)
- collection.exists?(...)
- collection.build(attributes = {})
- collection.create(attributes = {})
- collection.create!(attributes = {})
- collection.reload

In all of these methods, collection is replaced with the symbol passed as the first argument to has_many, and collection_singular is replaced with the singularized version of that symbol. For example, given the declaration:

```
class Author < ApplicationRecord
  has_many :books
end
```

Each instance of the Author model will have these methods:

```
books
books<<(object, ...)
books.delete(object, ...)
books.destroy(object, ...)
books=(objects)
book_ids
book_ids=(ids)
books.clear
books.empty?
books.size
books.find(...)
books.where(...)
books.exists?(...)
books.build(attributes = {}, ...)
books.create(attributes = {})
books.create!(attributes = {})
books.reload
```

### 4.3.1.1 collection

The collection method returns a Relation of all of the associated objects. If there are no associated objects, it returns an empty Relation.

```
@books = @author.books
```

### 4.3.1.2 collection<<(object, ...)

The `collection<<` method adds one or more objects to the collection by setting their foreign keys to the primary key of the calling model.

```
@author.books << @book1
```

### collection.delete(object, ...)

The collection.delete method removes one or more objects from the collection by setting their foreign keys to NULL.

```
@author.books.delete(@book1)
```

Additionally, objects will be destroyed if they're associated with dependent: :destroy, and deleted if they're associated with dependent: :delete_all.

### collection.destroy(object, ...)

The collection.destroy method removes one or more objects from the collection by running destroy on each object.

```
@author.books.destroy(@book1)
```

Objects will always be removed from the database, ignoring the :dependent option.

### collection=(objects)

The collection= method makes the collection contain only the supplied objects, by adding and deleting as appropriate. The changes are persisted to the database.

#### 4.3.1.6 collection_singular_ids

The collection_singular_ids method returns an array of the ids of the objects in the collection.

```
@book_ids = @author.book_ids
```

### collection_singular_ids=(ids)

The collection_singular_ids= method makes the collection contain only the objects identified by the supplied primary key values, by adding and deleting as appropriate. The changes are persisted to the database.

### collection.clear

The collection.clear method removes all objects from the collection according to the strategy specified by the dependent option. If no option is given, it follows the default strategy. The default strategy for has_many :through associations is delete_all, and for has_many associations is to set the foreign keys to NULL.

```
@author.books.clear
```

Objects will be deleted if they're associated with dependent: :destroy or dependent: :destroy_async, just like dependent: :delete_all.

#### 4.3.1.9 collection.empty?

The collection.empty? method returns true if the collection does not contain any associated objects.

```
<% if @author.books.empty? %>
  No Books Found
<% end %>
```

#### 4.3.1.10 collection.size

The `collection.size` method returns the number of objects in the collection.

```ruby
@book_count = @author.books.size
```

#### 4.3.1.11 collection.find(...)

The `collection.find` method finds objects within the collection's table.

```
@available_book = @author.books.find(1)
```

#### 4.3.1.12 collection.where(...)

The collection.where method finds objects within the collection based on the conditions supplied but the objects are loaded lazily meaning that the database is queried only when the object(s) are accessed.

```
@available_books = @author.books.where(available: true) # No query yet
@available_book = @available_books.first # Now the database will be queried
```

#### 4.3.1.13 collection.exists?(...)

The collection.exists? method checks whether an object meeting the supplied conditions exists in the collection's table.

### collection.build(attributes = {})

The collection.build method returns a single or array of new objects of the associated type. The object(s) will be instantiated from the passed attributes, and the link through their foreign key will be created, ==but the associated objects will not yet be saved.==

```ruby
@book = @author.books.build( published_at: Time.now, book_number: "A12345" )

@books = @author.books.build([
       { published_at: Time.now, book_number: "A12346" },
	{ published_at: Time.now, book_number: "A12347" }
])
```

### collection.create(attributes = {})

The `collection.create` method returns a single or array of new objects of the associated type. The object(s) will be instantiated from the passed attributes, the link through its foreign key will be created, ==and, once it passes all of the validations specified on the associated model, the associated object will be saved.==

```ruby
@book = @author.books.create( published_at: Time.now, book_number: "A12345" )

@books = @author.books.create([
	{ published_at: Time.now, book_number: "A12346" },
	{ published_at: Time.now, book_number: "A12347" }
])
```

#### 4.3.1.16 collection.create!(attributes = {})

Does the same as collection.create above, but raises ActiveRecord::RecordInvalid if the record is invalid.

#### 4.3.1.17 collection.reload

The collection.reload method returns a Relation of all of the associated objects, forcing a database read. If there are no associated objects, it returns an empty Relation.

```
@books = @author.books.reload
```

## Options for has_many

While Rails uses intelligent defaults that will work well in most situations, there may be times when you want to customize the behavior of the has_many association reference. Such customizations can easily be accomplished by passing options when you create the association. For example, this association uses two such options:

```ruby
class Author < ApplicationRecord
          has_many :books, dependent: :delete_all, validate: false
end
```

The has_many association supports these options:

- :as
- :autosave
- :class_name
- :counter_cache
- :dependent
- :disable_joins
- :ensuring_owner_was
- :extend
- :foreign_key
- :foreign_type
- :inverse_of
- :primary_key
- :query_constraints
- :source
- :source_type
- :strict_loading
- :through
- :validate

### :as

Setting the :as option indicates that this is a polymorphic association, as discussed earlier in this guide.

### :autosave

If you set the :autosave option to true, Rails will save any loaded association members and destroy members that are marked for destruction whenever you save the parent object. Setting :autosave to false is not the same as not setting the :autosave option. If the :autosaveoption is not present, then new associated objects will be saved, but updated associated objects will not be saved.

### :class_name

If the name of the other model cannot be derived from the association name, you can use the :class_name option to supply the model name. For example, if an author has many books, but the actual name of the model containing books is Transaction, you'd set things up this way:

```
class Author < ApplicationRecord
  has_many :books, class_name: "Transaction"
end
```

### :counter_cache

This option can be used to configure a custom named :counter_cache. You only need this option when you customized the name of your :counter_cache on the belongs_to association.

### :dependent

Controls what happens to the associated objects when their owner is destroyed:

- `:destroy` causes all the associated objects to also be destroyed
- :delete_all causes all the associated objects to be deleted directly from the database (so callbacks will not execute)
- :destroy_async: when the object is destroyed, an ActiveRecord::DestroyAssociationAsyncJob job is enqueued which will call destroy on its associated objects. Active Job must be set up for this to work.
- :nullify causes the foreign key to be set to NULL. Polymorphic type column is also nullified on polymorphic associations. Callbacks are not executed.
- `:restrict_with_exception` causes an ActiveRecord::DeleteRestrictionError exception to be raised if there are any associated records
- ==`:restrict_with_error` causes an error to be added to the owner if there are any associated objects.==

#### `:restrict_with_error` vs `:destroy`

For a serious business system, `restrict_with_error` is the superior, "orderly" choice.

- Data Safety (The Anti-Chaos): If you or a manager accidentally deletes a "Croissant" product, `dependent: :destroy` would silently wipe out years of formula history (V1, V2, V3...) instantly. That is chaos.
	`restrict_with_error` enforces order: it says, "You cannot remove this Product because it acts as a parent to existing legal documents (Formulas)."
- Explicit Intent: To delete a Product, you must first consciously decide what to do with its Formulas (delete them manually or archive them). It forces a deliberate action.

| **Option**                        | **Behavior when deleting a Product**                         | **Verdict for BOS**                           |
| --------------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| `dependent: :destroy`             | **Cascading Delete.** Instantly deletes the Product AND all link records in the `formulations` table. | **Dangerous.** Risk of losing recipe history. |
| `dependent: :restrict_with_error` | **Halt & Warn.** The deletion fails. `product.errors` will contain a message saying it cannot be deleted. | **Safe.** Maintains order and integrity.      |

### :disable_joins

Specifies whether joins should be skipped for an association. If set to true, two or more queries will be generated. Note that in some cases, if order or limit is applied, it will be done in-memory due to database limitations. This option is only applicable on has_many :through associations as has_many alone do not perform a join.

### :ensuring_owner_was

Specifies an instance method to be called on the owner. The method must return true in order for the associated records to be deleted in a background job.

### :extend

Specifies a module or array of modules that will be extended into the association object returned. Useful for defining methods on associations, especially when they should be shared between multiple association objects.

### :foreign_key

By convention, Rails assumes that the column used to hold the foreign key on the other model is the name of this model with the suffix _id added. The :foreign_key option lets you set the name of the foreign key directly:

```ruby
class Author < ApplicationRecord
       has_many :books, foreign_key: "cust_id"
end
```

In any case, Rails will not create foreign key columns for you. You need to explicitly define them as part of your migrations.

### :foreign_type

Specify the column used to store the associated object’s type, if this is a polymorphic association. By default this is guessed to be the name of the polymorphic association specified on “as” option with a “_type” suffix. So a class that defines a has_many :tags, as: :taggable association will use “taggable_type” as the default :foreign_type.

### :inverse_of

The :inverse_of option specifies the name of the belongs_to association that is the inverse of this association. See the bi-directional association section for more details.

```
class Author < ApplicationRecord
  has_many :books, inverse_of: :author
end

class Book < ApplicationRecord
  belongs_to :author, inverse_of: :books
end
```

### :primary_key

By convention, Rails assumes that the column used to hold the primary key of the association is id. You can override this and explicitly specify the primary key with the :primary_key option.

Let's say the users table has id as the primary_key but it also has a guid column. The requirement is that the todos table should hold the guid column value as the foreign key and not id value. This can be achieved like this:

```
class User < ApplicationRecord
  has_many :todos, primary_key: :guid
end
```



Now if we execute @todo = @user.todos.create then the @todo record's user_id value will be the guid value of @user.

#### 4.3.2.13 :query_constraints

Serves as a composite foreign key. Defines the list of columns to be used to query the associated object. This is an optional option. By default Rails will attempt to derive the value automatically. When the value is set the Array size must match associated model’s primary key or query_constraintssize.

#### 4.3.2.14 :source

The :source option specifies the source association name for a `has_many :through` association. You only need to use this option if the name of the source association cannot be automatically inferred from the association name.

#### 4.3.2.15 :source_type

The :source_type option specifies the source association type for a has_many :throughassociation that proceeds through a polymorphic association.

```
class Author < ApplicationRecord
  has_many :books
  has_many :paperbacks, through: :books, source: :format, source_type: "Paperback"
end

class Book < ApplicationRecord
  belongs_to :format, polymorphic: true
end

class Hardback < ApplicationRecord; end
class Paperback < ApplicationRecord; end
```

#### 4.3.2.16 :strict_loading

When set to true, enforces strict loading every time the associated record is loaded through this association.

#### 4.3.2.17 :through

==The `:through` option specifies a join model through which to perform the query. `has_many :through` associations provide a way to implement many-to-many relationships==, as discussed earlier in this guide.

### :validate

If you set the :validate option to false, then new associated objects will not be validated whenever you save this object. By default, this is true: new associated objects will be validated when this object is saved.

### Scopes for has_many

There may be times when you wish to customize the query used by `has_many`. Such customizations can be achieved via a scope block. For example:

```ruby
class Author < ApplicationRecord
       has_many :books, -> { where processed: true }
end
```

You can use any of the standard querying methods inside the scope block. The following ones are discussed below:

- where
- extending
- group
- includes
- limit
- offset
- order
- readonly
- select
- distinct

#### 4.3.3.1 where

The where method lets you specify the conditions that the associated object must meet.

```
class Author < ApplicationRecord
  has_many :confirmed_books, -> { where "confirmed = 1" },
    class_name: "Book"
end
```

You can also set conditions via a hash:

```
class Author < ApplicationRecord
  has_many :confirmed_books, -> { where confirmed: true },
    class_name: "Book"
end
```

If you use a hash-style where option, then record creation via this association will be automatically scoped using the hash. In this case, using @author.confirmed_books.create or @author.confirmed_books.build will create books where the confirmed column has the value true.

#### 4.3.3.2 extending

The extending method specifies a named module to extend the association proxy. Association extensions are discussed in detail later in this guide.

#### 4.3.3.3 group

The `group` method supplies an attribute name to group the result set by, using a `GROUP BY` clause in the finder SQL.

```ruby
class Author < ApplicationRecord
  has_many :chapters, -> { group 'books.id' }, through: :books
end
```

#### 4.3.3.4 includes

You can use the includes method to specify second-order associations that should be eager-loaded when this association is used. For example, consider these models:

```
class Author < ApplicationRecord
  has_many :books
end

class Book < ApplicationRecord
  belongs_to :author
  has_many :chapters
end

class Chapter < ApplicationRecord
  belongs_to :book
end
```

If you frequently retrieve chapters directly from authors (@author.books.chapters), then you can make your code somewhat more efficient by including chapters in the association from authors to books:

```
class Author < ApplicationRecord
  has_many :books, -> { includes :chapters }
end

class Book < ApplicationRecord
  belongs_to :author
  has_many :chapters
end

class Chapter < ApplicationRecord
  belongs_to :book
end
```

#### 4.3.3.5 limit

The `limit` method lets you restrict the total number of objects that will be fetched through an association.

```ruby
class Author < ApplicationRecord
	has_many :recent_books, -> { order('published_at desc').limit(100) }, class_name: "Book"
end
```

#### 4.3.3.6 offset

The `offset` method lets you specify the starting offset for fetching objects via an association. For example, `-> { offset(11) }` will skip the first 11 records.

#### 4.3.3.7 order

The `order` method dictates the order in which associated objects will be received (in the syntax used by an SQL `ORDER BY` clause).

```ruby
class Author < ApplicationRecord
       has_many :books, -> { order "date_confirmed DESC" }
end
```

#### 4.3.3.8 readonly

If you use the `readonly` method, then the associated objects will be read-only when retrieved via the association.

#### 4.3.3.9 select

The `select` method lets you override the SQL `SELECT` clause that is used to retrieve data about the associated objects. By default, Rails retrieves all columns.

If you specify your own select, be sure to include the primary key and foreign key columns of the associated model. If you do not, Rails will throw an error.

#### 4.3.3.10 distinct

Use the `distinct` method to keep the collection free of duplicates. This is mostly useful together with the `:through` option.

```ruby
class Person < ApplicationRecord
	has_many :readings
	has_many :articles, through: :readings
end
```

```ruby
irb> person = Person.create(name: 'John')
irb> article = Article.create(name: 'a1')
irb> person.articles << article
irb> person.articles << article
irb> person.articles.to_a
=> [#<Article id: 5, name: "a1">, #<Article id: 5, name: "a1">]
irb> Reading.all.to_a
=> [#<Reading id: 12, person_id: 5, article_id: 5>, #<Reading id: 13, person_id: 5, article_id: 5>]
```

In the above case there are two `readings` and `person.articles` brings out both of them even though these records are pointing to the same article.

Now let's set distinct:

```ruby
class Person
	has_many :readings
	has_many :articles, -> { distinct }, through: :readings
end
```

```ruby
irb> person = Person.create(name: 'Honda')
irb> article = Article.create(name: 'a1')
irb> person.articles << article
irb> person.articles << article
irb> person.articles.to_a
=> [#<Article id: 7, name: "a1">]
irb> Reading.all.to_a
=> [#<Reading id: 16, person_id: 7, article_id: 7>, #<Reading id: 17, person_id: 7, article_id: 7>]
```

In the above case there are still two readings. However `person.articles` shows only one article because the collection loads only unique records.

If you want to make sure that, upon insertion, all of the records in the persisted association are distinct (so that you can be sure that when you inspect the association that you will never find duplicate records), you should add a unique index on the table itself. For example, if you have a table named readings and you want to make sure the articles can only be added to a person once, you could add the following in a migration:

```ruby
add_index :readings, [:person_id, :article_id], unique: true
```

Once you have this unique index, attempting to add the article to a person twice will raise an ActiveRecord::RecordNotUnique error:

```ruby
irb> person = Person.create(name: 'Honda')
irb> article = Article.create(name: 'a1')
irb> person.articles << article
irb> person.articles << article
ActiveRecord::RecordNotUnique
```

Note that checking for uniqueness using something like `include?` is subject to race conditions. Do not attempt to use `include?` to enforce distinctness in an association. For instance, using the article example from above, the following code would be racy because multiple users could be attempting this at the same time:

```ruby
person.articles << article unless person.articles.include?(article)
```

## When are Objects Saved?

==When you assign an object to a `has_many` association, that object is automatically saved (in order to update its foreign key).== If you assign multiple objects in one statement, then they are all saved.

If any of these saves fails due to validation errors, then the assignment statement returns `false` and the assignment itself is cancelled.

If the parent object (the one declaring the `has_many` association) is unsaved (that is, `new_record?` returns `true`) then the child objects are not saved when they are added. ==All unsaved members of the association will automatically be saved when the parent is saved.==

If you want to assign an object to a `has_many` association without saving the object, use the `collection.build` method.