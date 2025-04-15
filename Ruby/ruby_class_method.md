# Class Methods in Ruby

Class methods in Ruby serve several essential purposes.

首先，Class methods 存在的首要目的，我不覺得是 untility methods or ready-to-run method，而是做它最本源的事情 —— 創造和管理 objects。作為「工具功能」存在時，它跟在 module 中沒什麼區別，還不如作為 module methods 適合。

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

## Factory Methods Pattern

Factory methods represent one of the most valuable and appropriate uses of class methods in Ruby. 

### Core Concepts of Factory Methods

Factory methods are class methods that create and return instances of the class, often with specialized initialization logic beyond what's possible with a simple `new` call.

```ruby
class Document
  def self.from_file(path)
    content = File.read(path)
    new(File.basename(path), content)
  end
  
  def initialize(name, content)
    @name = name
    @content = content
  end
end

doc = Document.from_file("report.txt")
```

### Key Benefits of Factory Methods

1. **Semantic Clarity**
	- Factory methods can have descriptive names that explain *how* an object is being created
	- `User.from_oauth(auth_data)` is more expressive than `User.new(params_extracted_from_auth_data)`

2. **Encapsulation of Creation Logic**
	- Complex object construction details stay within the class
	- External code doesn't need to know how to properly assemble an object

3. **Multiple Creation Patterns**
	- Different factory methods can implement different object creation strategies
	- Creates a clear API for various initialization scenarios

4. **Default and Derived Values**
	- Can compute default values or derive parameters that would be cumbersome to calculate at the call site
	- `Transaction.for_current_month` vs `Transaction.new(start_date: Date.today.beginning_of_month, end_date: Date.today.end_of_month)`

### Advanced Factory Method Patterns

#### Memoization Factories

```ruby
class Configuration
  def self.instance
    @instance ||= new.load_from_disk
  end
  
  private :initialize  # Makes new unusable from outside
end
```

#### Conditional Factories

```ruby
class Payment
  def self.create(attributes)
    case attributes[:method]
    when "credit_card" then CreditCardPayment.new(attributes)
    when "paypal" then PayPalPayment.new(attributes)
    when "bank_transfer" then BankTransferPayment.new(attributes)
    else raise ArgumentError, "Unknown payment method"
    end
  end
end
```

#### Builder Pattern Integration

```ruby
class Report
  def self.builder
    ReportBuilder.new
  end
end

# Usage:
report = Report.builder
           .with_title("Q3 Results")
           .including_section("Revenue")
           .including_section("Expenses")
           .build
```

### When to Use Factory Methods

- When object creation requires complex logic
- When you want to provide multiple ways to create an object with different parameters
- When the initialization process involves resources (files, network, database)
- When creating specialized instances of abstract base classes
- When you want to enforce specific object creation patterns

==Factory methods remain one of the strongest use cases for class methods, providing a clean, expressive API for object creation that maintains proper encapsulation of initialization logic.==

## `.new` is indeed also a Factory Method

`.new` is a class method that creates a new instance of the class, calls the instance's `initialize` method with the provided arguments, and returns the newly created instance. This is precisely what factory methods do - they encapsulate object creation logic and return instances. `.new` is the quintessential factory method in Ruby, and it's a perfect example of the pattern.

```ruby
# When we call:
user = User.new(name: "Alice", email: "alice@example.com")

# The `.new` method is a factory that:
# 1. Allocates a new User object in memory
# 2. Calls initialize on that object with the arguments
# 3. Returns the initialized object
```

### Special Properties of `.new`

`.new` is special in a few ways:

- It's automatically defined by Ruby for every class.
- It has a direct relationship with the instance's `initialize` method.
- It's the conventional, default factory method in Ruby.

## Custom Factory Methods vs. `.new`

When we create custom factory methods, we're essentially creating specialized alternatives to `.new`:

```ruby
class User
  # Standard factory method provided by Ruby
  # User.new(name:, email:)
  def initialize(name:, email:)
    @name = name
    @email = email
  end
  
  # Custom factory method
  def self.from_oauth(auth_data)
    new(
      name: auth_data.info.name,
      email: auth_data.info.email
    )
  end
end
```

## Overriding `.new`

You can even override `.new` itself to create custom behavior for the default factory:

```ruby
class Logger
  def self.new(*args)
    if @instance
      @instance
    else
      @instance = super
    end
  end
end
```

Your observation highlights an important point - the concept of factory methods isn't foreign to Ruby; it's built into the core of the language through `.new`. Our custom factory methods simply extend this fundamental pattern to handle specialized creation scenarios.

What is the relationship between factory methods, like new and the initialize method? Is initialize a class method or instance method?

## The Relationship between Factory Methods and `initialize`

You've touched on a crucial distinction in Ruby's object creation mechanism.

## `initialize` is an Instance Method

`initialize` is definitively an **instance method**, not a class method. This is a fundamental aspect of Ruby's object model.

```ruby
class Person
  def initialize(name)  # Instance method
    @name = name
  end
end
```

## The Factory Method and Initialize Relationship

The relationship between factory methods (like `.new`) and `initialize` is one of cooperation across the class/instance boundary:

1. **Sequence of Events:**
	- First, the factory method (e.g., `Person.new("Alice")`) is called on the class
	- The factory method allocates memory for a new, blank instance
	- Then, the factory method calls `initialize` on that new instance
	- Finally, the factory method returns the initialized instance

2. **Division of Responsibilities:**
	- Factory methods (class methods) handle the creation of the object
	- `initialize` (instance method) handles the setup of the object's internal state

## Visual Representation:

```
Person.new("Alice")  -->  instance = allocate_memory()
                           instance.initialize("Alice")
                           return instance
```

## Code Example Showing the Distinction:

```ruby
class Product
  # Class method (factory)
  def self.from_database(id)
    data = Database.find(id)
    instance = new  # Calls the built-in factory method
    # At this point, initialize has already been called
    instance.created_at = Time.now  # Additional setup
    instance
  end
  
  # Instance method
  def initialize(name = nil, price = nil)
    @name = name
    @price = price
  end
  
  attr_accessor :created_at
end
```

## Why This Separation Matters

This separation creates a clean division of responsibilities:

- Class methods handle class-level concerns like "how do I create an instance?"
- Instance methods handle instance-level concerns like "how do I set up my internal state?"

This design allows for powerful patterns:

- You can have multiple factory methods but a single initialization method
- Factory methods can choose what parameters to pass to initialize
- Factory methods can perform additional setup beyond what initialize does

The relationship between factory methods and `initialize` exemplifies Ruby's elegant object-oriented design, creating a clear separation between the creation process (class responsibility) and the initialization process (instance responsibility).

So the initialize method is triggered internally and automatically inside an object?







So the initialize method is triggered internally and automatically inside an object?



So the initialize method is triggered internally and automatically inside an object?



So the initialize method is triggered internally and automatically inside an object?

# Yes, `initialize` is Triggered Automatically

You're absolutely correct. The `initialize` method is triggered automatically during object creation.

## How `initialize` Gets Called

When you call a factory method like `new`:

1. Ruby allocates memory for the new object
2. Ruby automatically calls the `initialize` method on that new object
3. The arguments passed to `new` are forwarded to `initialize`

This happens internally within Ruby's object creation process. You never need to explicitly call `initialize` yourself.

## Example of Automatic Triggering

```ruby
class Person
  def initialize(name)
    puts "Initialize being called automatically with: #{name}"
    @name = name
  end
end

# When you run this:
person = Person.new("Ruby")

# You'll see:
# "Initialize being called automatically with: Ruby"
```

The key points:

- `initialize` is called automatically by Ruby's internals
- You don't manually call `initialize` in your code
- It happens behind the scenes during object creation
- It's part of Ruby's object instantiation mechanism

This automatic triggering of `initialize` is fundamental to Ruby's object model and is what makes the initialization process so seamless. The method exists specifically to be called automatically during object creation, allowing you to set up the initial state of your objects.


When you say "Ruby automatically calls the `initialize` method on that new object", what does Ruby mean? the interpreter?









# Class Methods and ActiveRecord

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
