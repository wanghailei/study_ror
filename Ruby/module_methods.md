# Module Methods

Ruby modules can have both module methods and instance methods, just like classes. This dual nature is what makes modules versatile in Ruby.

我認為，無論是 module methods 還是 class methods，都可以成為「母屬功能」；而 instance methods，可以被稱為「子屬功能」——屬於「子專用的功能」。

## Module Methods

Also called "class methods" of a module, these are defined on the module itself。這裏叫 class methods 是很不妥當的。

```ruby
module MyUtilities
    # Method 1: Using self
    def self.utility_method
    	puts "This is a module method called directly on the module"
    end

    # Method 2: Using module_function
    def formatted_time
    	Time.now.strftime("%H:%M:%S")
    end
    module_function :formatted_time
end

# Called directly on the module
MyUtilities.utility_method
MyUtilities.formatted_time
```

## Instance Methods

These become available to any class, % as its instance methods %, that includes the module:

```ruby
module Loggable
    def log(message)
    	puts "#{Time.now}: #{message}"
    end
end

class Product
	include Loggable
end

product = Product.new
product.log("Created a new product")  # Instance method from the module
```

## Mixed Approach with `module_function`

The `module_function` approach creates both types - module methods and instance methods, simultaneously:

```ruby
module Calculator
    def add(a, b)
	    a + b
    end
    module_function :add
end

# Can be used as a module method
puts Calculator.add(3, 4)  # => 7

# And also as an instance method
class MathClass
	include Calculator
end
math = MathClass.new
puts math.add(5, 6)  # => 11
```

This flexibility allows modules to serve both as namespaces for utility functions and as mixins for sharing behavior across multiple classes.

## Mixed Approach with `extend self`

When you see `extend self` in a Ruby module, it's implementing a specific pattern that makes all the module's instance methods accessible as module methods.

```ruby
module ActiveJob
    module QueueAdapters
        module ResqueAdapter
            # Resque-specific implementation
            extend self

            def enqueue(job) # defined as an instance method
            end
        end
    end
end
```

After using `extend self`, the instance methods can also be used as module methods.

```ruby
ActiveJob::QueueAdapters::ResqueAdapter.enqueue(some_job)
```

In the `ActiveJob` example, this pattern allows the adapter module to act as a singleton service object. The `ResqueAdapter` module becomes a namespace with callable methods, creating a clean interface without requiring instantiation of any objects.

### How It Works

`extend` adds a module's instance methods as class methods to the receiver. ==When the receiver is `self` inside a module definition, the module extends itself.== Therefore all instance methods defined in the module are also available as module methods.

### Comparison

**`extend self`**:

- Makes methods available both as module methods and as instance methods
- Preserves method visibility

**`module_function`**

- Similar to `extend self` but makes included methods private in the target class

**`self.method_name`**:

- Only creates module methods
- Methods aren't available when the module is included in a class.

