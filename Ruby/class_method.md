# Class Methods in Ruby

Class methods in Ruby serve several essential purposes.

首先，Class methods 存在的首要目的，我覺得，不是 untility methods or ready-to-run method，而是做它最本源的事情 —— 創造和管理 objects。作為「工具功能」存在時，它跟在 module 中沒什麼區別，還不如作為 module methods 適合。

Class methods are also called a "static method" in some other languages.

## Purpose of using class methods?

Class methods represent behaviors that are relevant to the class itself. Class methods allow operations that conceptually belong to the class as a whole, not to individual instances. This makes them indispensable to the language's object-oriented paradigm.

### Creation Patterns

#### Constructors Beyond `new`

Class methods provide alternative ways to create objects - named constructors - often with specific initialization patterns. Give more meaning than just `new`.

```ruby

class User
    # Look up user and return a properly configured instance
    def self.from_database(id)
    end
    def self.from_omniauth(auth_data) 
    end
end
```



**Factory methods**: Like `Session.start!`  - provides a clean, semantic way to create objects.



**Builder patterns**: Methods that construct complex objects step by step.

### Global Operations

- **Queries**: Methods that find specific objects (e.g., `User.find_by_transfer_id`)
- **Batch operations**: Perform actions on multiple instances at once
- **Caching**: Implement class-level caching strategies

### Namespacing Utility Functions

Class methods enable organising utility and helper methods with the class they serve but not instances.

Class methods help organize code by namespacing under the appropriate class, not polluting the global namespace.

## If Ruby had no class methods

Without class methods, Ruby would face significant limitations:

**Architectural Problems**

- Methods that operate on the class itself would need to be global methods.
- Utility methods would need to be placed in modules or as global methods.
- Poor encapsulation of class-related behaviors

**Creation Pattern Limitations**

- All object creation would be limited to `new` with explicit parameters.
- Factory patterns would be harder to implement.
- Named constructors would be impossible.

**Design Pattern Impact**

- Singleton pattern implementation would be difficult.
- Testing would be more complex without the ability to mock class methods.

### Real Example

In `Session.rb`, the `start!` method demonstrates a perfect use case:

```ruby
def self.start!(user_agent:, ip_address:)
	create! user_agent: user_agent, ip_address: ip_address
end
```

This is better than doing `Session.new` followed by setting attributes because:

1. It's communicates intent and more **concise**.
2. It's ensures sessions are always created properly.
3. It **encapsulates** creation logic - if creation needs to change, it's in one place.

### With ActiveSupport::Concern

The `class_methods` block in `ActiveSupport::Concern` is essentially a cleaner syntax for using `extend`. When you write:

```ruby
module User::Transferable
	extend ActiveSupport::Concern
  
    class_methods do
        def find_by_transfer_id(id)
	        find_signed(id, purpose: :transfer)
        end
    end
end
```

Behind the scenes, Rails is creating a submodule with these methods and extending the class with it when you include the main module. It's equivalent to:

```ruby
module User::Transferable
    module ClassMethods
        def find_by_transfer_id(id)
			find_signed(id, purpose: :transfer)
        end
    end

    def self.included(base)
    	base.extend(ClassMethods)
    end
end
```

This approach makes modules more powerful and expressive while maintaining clean syntax.



## `class << self`

This syntax is used to define **class methods** within the block. It opens up the singleton class of the current class, allowing you to define methods that will be available on the class itself, rather than on instances of the class.

Using the singleton class (`class << self`) to define class methods, groups them together makes the class more organised.

```ruby
class MyClass
    class << self
        def my_class_method
            # method implementation
        end
    end
end
```

A real-world code example from Rails:

```ruby
module ActiveRecord
    class CurrentAttributes
        class << self
            def instance
                current_instances[ current_instances_key ] ||= new
            end
            def attribute( *name )
            end
        end
    end
end

```

Using `self.` prefix, is another way, which is clear and concise, easy to read and understand.

```ruby
class MyClass
    def self.my_class_method
        # method implementation
    end
end
```

#### `class << self` opens a class definition block, where all class methods defined inside.



```ruby
# will become class methods rather than instance methods
# This is equivalent to writing "def self.method_name"
class << self

	def create_for(attributes, users:)
		transaction do
			create!(attributes).tap do |room|
				room.memberships.grant_to users
			end
		end
	end

	def original
		order(:created_at).first
	end
end
```



Let's talk more about class methods in Ruby. What is its essential purpose for existing? What if there is no class method in Ruby? 

Can you define a class inside a method/action?

## Methods in Modules vs. Class Methods

==IMHO, utility methods do should be put inside modules instead of classes.==

One of Ruby's design philosophy is favouring composition over inheritance and keeping responsibilities clearly separated.

Modules create a clearer distinction, or separation of concerns, between behavior and entity. ==Classes can focus on defining what objects *are*, while modules define what they *do*.==

Modules can be mixed into multiple classes, which creates more flexible and composable code than hierarchical class structures.

#### Example of Better Module Usage

Instead of:

```ruby
class StringUtils
    def self.titleize(str)
		# implementation
    end
end

StringUtils.titleize("hello world")
```

Using a module:

```ruby
module StringUtils
    def titleize(str)
		# implementation
    end
end

StringUtils.titleize("hello world")

# Or include it where needed
class SomeClass
	include StringUtils
end 

sc = SomeClass.new
sc.titleize("hello world")
```



## Class Methods and ActiveRecord

> For class methods, I saw in ActiveRecord, they are defined with `self.method_name`, and also called with `self.method_name`. Does it mean calling a class method inside the Class needs a `self` prefix?
>

## Defining Class Methods

In Ruby, there are several ways to define class methods:

```ruby
class MyClass
  # Method 1: Using self prefix
  def self.my_method
    # implementation
  end
  
  # Method 2: Using class << self block
  class << self
    def another_method
      # implementation
    end
  end
end
```

## Calling Class Methods

When it comes to calling class methods, the rules are:

1. **From outside the class**: You call them on the class name
   ```ruby
   MyClass.my_method
   ```

2. **From inside instance methods**: You must use the class name
   ```ruby
   def instance_method
     MyClass.my_method  # Correct
     self.class.my_method  # Also works
   end
   ```

3. **From inside class methods**: 
   ```ruby
   def self.method_a
     method_b  # Works without self prefix
     self.method_b  # Also works, with self prefix
   end
   ```

## ActiveRecord's Pattern

In ActiveRecord, you'll often see the `self` prefix when calling class methods from other class methods for a few reasons:

1. **Clarity and explicitness**: Makes it clear you're calling a class method, not an instance method or local variable
2. **Consistency**: Makes the code style consistent
3. **Avoiding ambiguity**: Distinguishes from local variables with the same name

For example:
```ruby
class User < ApplicationRecord
  def self.find_active
    self.where(active: true)  # self is explicit but not strictly required
  end
end
```

The `self` prefix is not technically required when calling one class method from another, but it's a common style convention in ActiveRecord and helps with readability.



## Class methods have feel of the functions in functional languages

我不應該混合討論不同的編程語言模式，這沒啥意義。我只是想紀錄下一點感受。

Class methods do share some similarities with functions in functional programming. They can be called without instantiating an object - Similar to standalone functions in functional languages. 

```ruby
ActionController::Renderer.normalize_env(env)  # Class method call
PostsController.render :show, assigns: { post: Post.first } 
```

However, there are also some important distinctions:

1. **Class methods are still associated with a class** - They're not truly standalone functions like in functional languages. They're namespaced under their class and have access to class variables/constants.
2. **Class methods can have side effects** - While functional programming emphasizes pure functions without side effects, Ruby's class methods can modify class variables or external state. Although good class methods, often don't depend on changing instance state - stateless.
