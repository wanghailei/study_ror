# Methods

Method，翻譯成中文，應該是什麼呢？方法、功能？我想「功能」是準確的。

## A method is an object

In Ruby, you can accurately say that ==a method is an object==. This is because Ruby is a language with a very consistent object model and everything in Ruby is an object, including methods.

[`Method`](https://ruby-doc.org/3.4.1/Method.html) objects are created by [`Object#method`](https://ruby-doc.org/3.4.1/Object.html#method-i-method), and are associated with a particular object (not just with a class). They may be used to invoke the method within the object, and as a block associated with an iterator. They may also be unbound from one object (creating an [`UnboundMethod`](https://ruby-doc.org/3.4.1/UnboundMethod.html)) and bound to another.

When you reference a method in Ruby using the `method` method or the `&` operator, what you get is ==an object of the class `Method`.== This object encapsulates the method and allows you to interact with it as an object, including the ability to call the method, inspect its parameters, and so on.

For example:

```ruby
class MyClass
	def say
		"Hello, world!"
	end
end

obj = MyClass.new
method_object = obj.method :say  # Retrieves the 'say' as a Method object
p method_object.class  # => Method
p method_object.call # => "Hello, world!"
```

In this code, `method_object` is an instance of the `Method` class. You can then call this method using `method_object.call`, ==pass it around as an object==, and use other `Method` class functionalities. 

==The `Method` class is particularly useful for metaprogramming==, where methods might be manipulated programmatically, or for scenarios where you need to store references to methods for later execution.

## Defining a Method

In Ruby, parentheses in method definitions are optional unless you need to clarify the precedence of parameters. You can define a method with or without them. Both of the following definitions are valid and functionally equivalent:

```ruby
def a_method(p)
	# Method body
end

def a_method p
	# Method body
end

```

### One-line method

#### the endless method

```ruby
def a_method_name(arg) = puts arg
```



```ruby
def a_method_name; end
```





## Kinds of Methods

Methods defined inside a class and intended for use by all instances of the class, are called instance methods.

Methods that you define for one particular object are called singleton methods.

A class method belongs to the class itself.

==%. 有三種 methods：== 

- ==Class methods：這是 class 給自己（工廠）用的；== 
- ==Instance methods：這是 class 安裝進生產出來的 instance（object）上的； ==
- ==Singleton methods：這是某一個 instance（object）在被生成後，自己搞來用的。== 
	==20231223 %==

### What Are Class Methods For?

Why create a class method rather than the more usual instance method? 
There are two main reasons: First, a class method can ==be used as a “ready-to-run function” without having to go to the bother of creating an object just to use it==, and second, ==it can be used on those occasions when you need to run a method before an object has been created==.

In Ruby, method names can contain characters like ?, !, and =. The convention is to end method names with a ? if they return either true or false, with ! to indicate the method will have some sort of side effect like changing the object it's called on, and with = when defining setter methods.

### The splat `*` operator

In Ruby, the `*` operator, when placed before the parameter in a method definition, is called the 'splat' operator. It's primarily used to specify that the method can accept one or more arguments. This makes it useful in scenarios where you don't know in advance how many arguments will be passed into the method. So essentially, ==the `*args` parameter allows this method to take any number of arguments.==

```ruby
def my_method( *args )
	args.each { |arg| puts arg }
end
my_method("Hello", "World", "!")
# => Hello
# => World
# => !
```

## `super`

Inside the body of a method definition, you can use the `super` keyword to jump up to the next-highest definition in the method-lookup path of the method you’re currently executing. The `super` keyword tells an object to search for and execute ==a method of identical name== defined in the object’s namespace.

When `super()` is used in an instance method, ==a method of the same name but in the superclass== of the current class is called. This is often used in method overrides, where you want to augment or modify the behavior of a method inherited from the superclass, not replace it entirely.

* Called with no argument list (empty or otherwise), `super` automatically forwards the arguments that were passed to the child method.
* Be noted, if `super()` is used, namely `super` with parentheses but no arguments, it calls the superclass method without any arguments, regardless of what arguments were passed to the child method.

For example:

```ruby
class Animal
    def speak
        "Some generic sound"
    end
end
class Dog < Animal
    def speak
        "#{super()} and then barks"
    end
end
dog = Dog.new
puts dog.speak # => "Some generic sound and then barks"
```

`super()` invokes the `speak` method of `Animal`. 

## The method name started with an underscore

`def _prepare_context`&#x20;

In Ruby, a method name beginning with an underscore (`_`) is typically and just a convention used by developers as an additional visual cue to indicate that a method is intended for internal use within the class or module. It's only a naming convention and not a language feature. The underscore does not change the visibility of the method.

For example, in Rails, methods that are not intended to be actions in a controller might be prefixed with an underscore to differentiate them from action methods.



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







**Let's talk more about class methods in Ruby. What is its essential purpose for existing? What if there is no class method in Ruby? 



Can you define a class inside a method/action?

# Defining a Class Inside a Method in Ruby

## The Short Answer

Yes, you can define a class inside a method/action in Ruby. This creates a class dynamically when the method runs, with the class typically scoped to where the method makes it available.

## Example of a Class in a Method

```ruby
def create_custom_class(name)
  # Define a class inside this method
  Class.new do
    attr_accessor :custom_name
    
    def initialize(custom_name = nil)
      @custom_name = custom_name || name
    end
    
    def greet
      puts "Hello from #{@custom_name}!"
    end
  end
end

# Create different classes dynamically
PersonClass = create_custom_class("Person")
RobotClass = create_custom_class("Robot")

# Use the dynamically created classes
person = PersonClass.new
robot = RobotClass.new("Mechanical Friend")

person.greet  # Outputs: Hello from Person!
robot.greet   # Outputs: Hello from Mechanical Friend!
```

## Alternative Syntax with Named Class

You can also use the `class` keyword within a method:

```ruby
def create_named_class
  # Using the class keyword inside a method
  class TemporaryClass
    def perform_action
      puts "Action performed!"
    end
  end
  
  # Return the class reference
  return TemporaryClass
end

# The class only exists after method execution
MyClass = create_named_class
instance = MyClass.new
instance.perform_action  # Outputs: Action performed!

# This would raise an error if called directly
# TemporaryClass.new  # Error: NameError: uninitialized constant TemporaryClass
```

## Technical Details

When defining a class in a method:

1. The class is created when the method executes
2. Each call to the method creates a new class (even with the same name)
3. The class is defined in the current lexical scope (not available outside unless returned or stored)
4. Classes created with the same name in different method calls are different classes

## Practical Applications

This technique enables:

1. **Factory patterns**: Generating specialized classes based on parameters
2. **Dynamic composition**: Creating classes with varying elements and actions
3. **Configuration-based classes**: Building classes that adapt to runtime settings
4. **Domain-specific objects**: Creating customized classes for specific contexts

## Example: Dynamic Element and Action Generation

```ruby
def build_entity_class(type, elements, actions)
  Class.new do
    # Set up the elements
    elements.each do |element_name|
      attr_accessor element_name
    end
    
    # Initialize with the elements
    define_method :initialize do |values = {}|
      @type = type
      elements.each do |element_name|
        instance_variable_set("@#{element_name}", values[element_name])
      end
    end
    
    # Add the actions
    actions.each do |action_name, implementation|
      define_method action_name, &implementation
    end
    
    # Add a helper method
    define_method :describe do
      puts "I am a #{@type} with these elements:"
      elements.each do |element_name|
        value = instance_variable_get("@#{element_name}")
        puts "  #{element_name}: #{value}"
      end
    end
  end
end

# Use the method to create dynamic classes
hero_elements = [:name, :strength, :health]
hero_actions = {
  attack: ->(target) { puts "#{@name} attacks #{target} with #{@strength} power!" },
  heal: -> { @health += 10; puts "#{@name} healed to #{@health} health!" }
}

HeroClass = build_entity_class("Hero", hero_elements, hero_actions)

# Create an instance
hero = HeroClass.new(name: "Aragorn", strength: 95, health: 100)
hero.describe
hero.attack("Orc")
hero.heal
```

This demonstrates how Ruby's metaprogramming capabilities allow you to dynamically create classes with custom elements and actions within method scope.

## Method Chaining pattern

Method chaining, also called "**fluent interface**", is when multiple method calls are chained together in a single expression by:

1. Having each method return an object;
2. Calling another method on the returned object;
3. Continuing this pattern as needed.

Each method in the chain (except the last) must return a value that responds to the next method in the chain.

```ruby
target_scope.merge!(association_scope).merge!(scope) 
```

This demonstrate method chaining, where each method returns an object that the next method is called on. 

This pattern is very common in Ruby, especially in contexts like `ActiveRecord` queries:

```ruby
User.where(active: true).order(created_at: :desc).limit(10)
```

The following code also uses the pattern but with class method followed by an instance method.

```ruby
DisableJoinsAssociationScope.create.scope(self)
```

1. First calls the `create` method on the `DisableJoinsAssociationScope` class, which likely returns a new instance of that class

2. `.scope(self)` - Then calls the `scope` method on the object returned by `create`, passing `self` as an argument

For this to work properly, the `create` method must return an object that has a `scope` method.

### Benefits of Method Chaining

- Creates more concise, readable code
- Reduces the need for temporary variables
- Can make code flow more naturally when properly designed





## Recursion

==Recursion is a programming technique where a method calls itself to solve a problem.== It's particularly useful for tasks that can be broken down into smaller, similar sub-problems.

==A recursive method needs a base case (terminating condition), and a recursive case (where the method calls itself).==

#### Factorial Calculation

```ruby
def factorial(n)
    # Base case
    return 1 if n <= 1

    # Recursive case
    n * factorial(n-1)
end

puts factorial(5)  # Output: 120
```

#### Fibonacci Sequence

```ruby
def fibonacci(n)
    # Base cases
    return 0 if n == 0
    return 1 if n == 1

    # Recursive case
    fibonacci(n-1) + fibonacci(n-2)
end

puts fibonacci(10)  # Output: 55
```

#### Directory Traversal

```ruby
def list_files(directory)
  Dir.foreach(directory) do |entry|
    next if entry == '.' || entry == '..'
    
    full_path = File.join(directory, entry)
    if File.directory?(full_path)
      list_files(full_path)  # Recursive call for subdirectories
    else
      puts full_path
    end
  end
end

list_files('/some/directory')
```

#### Tree Data Structure Traversal

```ruby
class TreeNode
  attr_accessor :value, :children
  
  def initialize(value)
    @value = value
    @children = []
  end
  
  def traverse
    # Process this node
    puts @value
    
    # Recursively process all children
    @children.each do |child|
      child.traverse
    end
  end
end
```

#### ActiveRecord uses recursion to create records

```ruby
# ActiveRecord::Relation
def create(attributes = nil, &block)
    if attributes.is_a?(Array)
		attributes.collect { |attr| create(attr, &block) }
    else
        block = current_scope_restoring_block(&block)
		scoping { _create(attributes, &block) }
    end
end
```

### Advanced: Tail Recursion

Ruby doesn't optimize tail recursion by default, but you can write tail-recursive methods:

```ruby
def factorial_tail(n, accumulator = 1)
	return accumulator if n <= 1
	factorial_tail(n-1, n * accumulator)
end
```

### Avoiding Stack Overflow

Recursive methods can cause stack overflow for deep recursion. Solutions include:
- Tail recursion (though Ruby doesn't optimize it)
- Iteration instead of recursion
- Memoization to avoid redundant calculations

```ruby
# Memoized Fibonacci
def fibonacci_memoized(n, memo = {})
	return memo[n] if memo.has_key?(n)
  
	result =
	if n <= 1
		n
	else
		fibonacci_memoized(n-1, memo) + fibonacci_memoized(n-2, memo)
	end
    
	memo[n] = result
	result
end
```

### Converting Recursion to Iteration

==Many recursive algorithms can be rewritten using iteration==:

```ruby
def factorial_iterative(n)
    result = 1
    (1..n).each do |i|
	    result *= i
    end
    result
end
```

Recursion is powerful but should be used carefully in Ruby, considering memory usage and performance implications for deep recursive calls.



