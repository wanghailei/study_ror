# `.new` and the Factory Method pattern

## `.new` uses the Factory Method pattern

`.new` is a class method that creates a new instance of the class, calls the instance's `initialize` method with the provided arguments, and returns the newly created instance. This is precisely what factory methods do - they encapsulate object creation logic and return instances. ==`.new` is the quintessential factory method in Ruby==, and it's a perfect example of the pattern.

```ruby
# When we call:
user = User.new(name: "Alice", email: "alice@example.com")

# The `.new` method is a factory that:
# 1. Allocates a new User object in memory
# 2. Calls initialize on that object with the arguments
# 3. Returns the initialized object
```

### Special properties of `.new`

`.new` is special in a few ways:

- It's automatically defined by the interpreter for every class.
- It has a direct relationship with the instance's `initialize` method.
- It's the conventional, default factory method in Ruby.

## The Factory Methods Pattern

Factory methods are class methods that create and return instances of the class, often with specialized initialization logic beyond what's possible with a simple `new` call. Factory methods represent one of the most valuable and appropriate uses of class methods in Ruby. 

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



## Defining an instance method's name as `new`?

It's technically valid to define an instance method named `new` in Ruby, but it's generally not recommended because this potential confused with class methods. Ruby has strong naming conventions, and deviating from them can make code harder to understand and maintain.

In Ruby, `new` is conventionally a class method used to instantiate objects. Having an instance method with the same name can confuse other developers. When someone sees `object.new()`, they might initially think you're trying to create a new instance, rather than performing whatever action your instance method is designed to do. 

```ruby
# Rails. 
# ActionController::Renderer.new

def new(env = nil)
	self.class.new controller, env, @defaults
end
```

In this specific ActionController::Renderer case, the method is used to create a new renderer instance with modified settings. A better name might be something like `create_with`, `with_env`, or `clone_with_env` to more clearly indicate the purpose.

That said, the Rails codebase sometimes breaks conventions for specific design reasons. This particular case appears to be part of the internal API, where the team made a deliberate choice that made sense in their context.
