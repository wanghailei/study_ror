# Singleton Classe

## Every Ruby Object Has a Singleton Class.

In Ruby, every object—including classes—has a singleton class (also called an eigenclass or metaclass) automatically, whether you use it explicitly or not.

### What Is a Singleton Class?

==A singleton class is an invisible class that sits between any object and its regular class in the inheritance chain.== Its purpose is to store methods that are specific to that single object instance, rather than being shared across all instances of that class.

#### Key Properties of Singleton Classes

1. **Automatically Created**: They exist for every object but are only materialized when needed
2. **Invisible by Default**: They don't show up in normal code inspection
3. **Object-Specific**: Each object has its own unique singleton class
4. **Method Lookup Priority**: Methods in the singleton class take precedence over methods in the regular class

### How Singleton Classes Are Used

#### 1. With Regular Objects

```ruby
str = "hello"

# Adding a method to just this string's singleton class
def str.shout
	self.upcase + "!"
end

str.shout  # => "HELLO!"
"world".shout  # Error - other strings don't have this method
```

#### 2. With Classes (Class Methods)

```ruby
class MyClass
end

# These are three equivalent ways to define class methods:

# 1. Using self prefix inside the class
class MyClass
	def self.class_method
		"I'm a class method"
	end
end

# 2. Using the singleton class directly with class << self
class MyClass
	class << self
		def another_class_method
			"I'm also a class method"
		end
	end
end

# 3. Opening the singleton class from outside
def MyClass.third_method
	"Another class method"
end
```

### The `class << self` Syntax

In your Room model:

```ruby
class Room < ApplicationRecord
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
end
```

This block is explicitly opening the singleton class of the `Room` class to define class methods. It's equivalent to:

```ruby
def self.create_for(attributes, users:)
  # implementation
end

def self.original
  # implementation
end
```

## Why Use Singleton Classes?

1. **Defining Class Methods**: The most common use case, as seen in your `Room` model
2. **Per-Object Behavior**: Adding methods to specific instances without affecting others
3. **Metaprogramming**: Dynamically defining methods for specific objects
4. **Avoiding Monkey Patching**: Adding functionality to specific instances of built-in classes without modifying the class globally

## Singleton Classes vs. Ruby's Singleton Pattern

Although the name is similar, Ruby's singleton classes are different from the Singleton pattern in software design (though Ruby does have a `Singleton` module in its standard library for implementing that pattern).

Singleton classes in Ruby are a language feature that enables per-object method definitions, while the Singleton pattern is a design pattern that ensures a class has only one instance.
