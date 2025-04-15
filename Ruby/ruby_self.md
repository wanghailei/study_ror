

# `self`

## What is `self`?

In Ruby, `self` is a keyword that refers to the current object, which is the object "owning" or receiving the currently executing code. Its value depends on the context:

- At the top level, outside any class or method, `self` is the `main` object.
- ==Inside a class definition, `self` refers to the class itself==, allowing definition of class methods.
- ==Inside an instance method, `self` refers to the instance of the class==, used for accessing instance variables.

`self` is a fundamental keyword in Ruby, providing access to the current object, which is ==the object receiving the current method== or in whose context the code is executed. `self` changes based on where the code is, such as being a class instance inside methods or the `main` object at the top level. 

Importantly, ==`self` is read-only and cannot be reassigned==, which would result in a syntax error if attempted.

### What is the Current Object?

The current object is what `self` refers to at any given moment, serving as the receiver for method calls and the context for variable access. This concept is central to Ruby's object-oriented nature, as ==every piece of code executes within an object's context==. The current object can be an instance, a class, or the `main` object, depending on the scope.

==`current object` 就是一個概念，是一种描述，根本不出現在代码里。`self` 就是唯一需要理解的。==

### What is `main`?

==`main` is the name for Ruby's top-level object, an instance of the `Object` class.== It is an instance of the `Object` class, the root of Ruby's class hierarchy. This object serves as the default receiver for methods and variables defined outside any class or module. It's automatically created before program execution begins. 

When evaluated at the top level, `self` returns `"main"`, due to a special `to_s` method assigned to `main` internally. 

## Contextual Behaviors

To illustrate, consider the following contexts and their impact on `self`, the current object, and `main`:

- **Top-level Context**:

  ```ruby
  puts self.inspect # Outputs: "main"
  def hello
  	puts "Hello from main!"
  end
  hello # Calls hello on main
  ```

  Here, `self` is `main`, and methods are defined on the `Object` class.

  

- **Inside a Class Definition**:

  ```ruby
  class Person
  	puts self.inspect # Outputs: "Person"
  	def self.class_method
      	puts self.inspect # Outputs: "Person"
  	end
  end
  ```

  ==Inside the class, `self` is the class== (`Person`), allowing class method definitions.

  

- **Inside an Instance Method**:

	```ruby
	class Person
		def initialize(name)
			@name = name
			puts self.inspect # Outputs: "#<Person:0x...>"
		end
	end
	person = Person.new("Alice")
	```

	Here, `self` is the instance (`person`), enabling access to instance variables.

#### Summary of Contexts for `self`, Current Object, and `main`

| **Context**                       | **Value of `self`**           | **Current Object** | **Notes**                                                    |
| --------------------------------- | ----------------------------- | ------------------ | ------------------------------------------------------------ |
| Top Level                         | `main` (instance of `Object`) | `main`             | Global methods defined here are private methods of `Object`. |
| Inside Class Definition           | The class (e.g., `Person`)    | The class          | Used for defining class methods (e.g., `def self.method`).   |
| Inside Instance Method            | The instance (e.g., `person`) | The instance       | Used for accessing instance variables (e.g., `@name`).       |
| Inside Module Definition          | The module                    | The module         | Similar to class, for module-level methods and constants.    |
| Inside Metaclass (`class << foo`) | The metaclass                 | The metaclass      | For defining singleton methods on an object, as per [Honeybadger](https://www.honeybadger.io/blog/ruby-self-cheat-sheet/). |

No.1 use for `self`, without a doubt, is to define class-level methods.

### `self` vs. `itself`

It's worth noting a distinction between `self` and `itself`. While `self` is a keyword with context-dependent value, `itself` is a method that returns the object it's called on, useful in scenarios like array selections (e.g., `[1,2,3,nil].select(&:itself)`).





## With or without `self.`



```ruby
# ActiveRecord

class BelongsToAssociation < SingularAssociation
    def replace(record)
        if record
			...
        elsif target
            remove_inverse_instance(target)
        end
			...
        self.target = record
    end
end
```

The difference between using `target` (without `self.`) and `self.target` has to do with how Ruby resolves method calls versus assignments.

### Method Call vs. Assignment

When `target` appears alone in the `elsif target` condition, it's a **method call** (Ruby is looking up a method named `target`). Ruby implicitly treats this as `self.target` for method lookup, so writing `self.` is optional for method calls.

#### Method Resolution

Ruby first checks if there's a local variable named `target`. If not, it looks for a method named `target` on the current object (`self`). 

### Why `self.target = record` Needs `self.`

#### Assignment Context

`self.target = record` is an assignment operation. Without `self.`, Ruby would interpret `target = record` as creating or assigning to a local variable named `target`. With `self.`, Ruby recognizes this as a method call to the setter method `target=`.

#### Ruby's Assignment Rule

In Ruby, you **must** use `self.` when calling setter methods, otherwise Ruby assumes you're creating a local variable. This is one of Ruby's syntactic quirks that can be confusing to newcomers.

## Example to Illustrate:

```ruby
class Person
  attr_accessor :name

  def change_name(new_name)
    # Method call - 'self.' is optional
    puts "Current name: #{name}"      # Same as self.name
    
    # Assignment - 'self.' is required for setter method
    name = new_name                   # Creates/sets local variable 'name'
    puts "Local variable: #{name}"
    puts "Name attribute: #{self.name}"  # Still the old value!
    
    # Correct way to set the attribute
    self.name = new_name
    puts "Updated attribute: #{self.name}"
  end
end

person = Person.new
person.name = "Alice"
person.change_name("Bob")
```

This behavior is a deliberate design decision in Ruby to provide predictable variable scoping rules, even though it creates this inconsistency between method calls and assignments.

The `target` referenced in this code is likely inherited from a parent class. The `BelongsToAssociation` class inherits from `SingularAssociation` as shown in the class definition:

```ruby
class BelongsToAssociation < SingularAssociation # :nodoc:
```

The `target` instance variable and accessor methods are probably defined in:
- `SingularAssociation` class, or
- `Association` class (if `SingularAssociation` inherits from it)

## How `target` is Used

Looking at the code, `target` appears to be an instance variable (`@target`) that holds the associated record, and accessed through inherited reader/writer methods.

The usage patterns indicate that:
- `target` (without `self.`) is a reader method call.
- `self.target = record` is a writer method call.

Ruby treats bare method calls (reading) as implicit `self.` calls. Ruby treats assignments without `self.` as local variable assignments.

This is why:
- `target` works for reading - it implicitly calls `self.target`
- `self.target = record` is needed for writing - to call the setter method instead of creating a local variable

The `target` accessor is defined in the base `Association` class:

```ruby
# In activerecord/lib/active_record/associations/association.rb
module ActiveRecord
    module Associations
        class Association
            attr_reader :owner, :target, :reflection

            def target=(target)
	            @target = target
	        end
        end
    end
end
```

This is a classic example of Ruby's inheritance model and method resolution.

