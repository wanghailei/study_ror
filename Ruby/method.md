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



## Arguments





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

The `*names` parameter syntax in the method definition bellow represents what's called a "splat" operator in Ruby.

```ruby
def attribute(*names, default: NOT_SET)
end
```

This allows the `attribute` method to accept any number of arguments (including zero), which will be collected into an array called `names`. For example, you can call this method with:

```ruby
attribute :user                         # names will be [:user]
attribute :account, :request            # names will be [:account, :request]
attribute :user, :account, :request     # names will be [:user, :account, :request]
```

This is a common pattern in Rails and Ruby in general when you want to define multiple similar items in a single method call. In this specific case, it allows declaring multiple current attributes at once rather than requiring separate calls to the `attribute` method for each one.

==When using the splat operator (`*`) in a method parameter, Ruby always converts the arguments into an array==, even if only one or zero argument is passed.

For example:

```ruby
def example(*names)
	puts "names is a #{names.class}: #{names.inspect}"
end

example(:user)                 # names is an Array: [:user]
example(:account, :request)    # names is an Array: [:account, :request]
example()                      # names is an Array: []
```

This behavior is consistent and reliable - the splat always collects arguments into an array, which makes it easy to iterate through the arguments or use array methods on them within your method body.

In the `attribute` method from your example, this allows for processing each attribute name with the same logic by simply iterating through the `names` array.

The `default: NOT_SET` part is a keyword parameter that specifies a default value for attributes if none is provided.

### Named Parameters

The `default:` or any name you choose, is specifically using Ruby's **keyword arguments** (also called named parameters) feature. The syntax `default: NOT_SET` defines a keyword parameter named "default" with a default value of `NOT_SET` , which appears to be a constant defined elsewhere in the code.

Ruby introduced keyword arguments in version 2.0 and enhanced them in later versions. These allow you to:

1. Name your parameters explicitly when calling a method
2. Make parameters optional with default values
3. Distinguish between positional arguments and named arguments

When calling this method, you could use it like:

```ruby
# Using the default value for 'default'
attribute :user

# Explicitly providing a value for the 'default' parameter
attribute :user, default: nil
attribute :account, default: -> { Company.default_account }
```

Keyword arguments improve code readability by making it clear what each argument represents, especially when a method takes multiple optional parameters. They're widely used throughout the Ruby and Rails ecosystems.

### Parameters vs Arguments

Both terms refer to values in method calls, but they're used at different points in the process:

#### Parameters

- Defined in the **method declaration**
- The variables listed in the method definition
- Act as placeholders that will receive values
- Exist at design time

```ruby
def welcome(name, age, location: "Unknown")
	# name, age, and location: are parameters
end
```

#### Arguments

- Used when **calling a method**
- The actual values passed to a method
- Provide concrete data to the parameters
- Exist at runtime

```ruby
welcome("Alice", 30, location: "New York")
# "Alice", 30, and "New York" are arguments
```

Maintaining the distinction is helpful for precision in technical discussions. 

- "The method takes three parameters."
- "I'm passing these arguments to the method."

In Conversation, this distinction helps communicate clearly whether you're talking about the method's design or how it's being used.

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

## Setter methods

==The `=` at the end of the method name `user=` defines what's called a "setter method" in Ruby.== This is a special syntax for methods that are designed to set the value of an attribute or property.

When you define a method with `=` at the end of its name:

1. It allows the method to be called using assignment syntax: `object.user = some_value` instead of `object.user(some_value)`

2. Ruby automatically passes the value on the right side of the assignment as an argument to the method

```ruby
def user=(user)
    super
    self.account = user.account
    Time.zone = user.time_zone
end
```

This method:
- Allows code to write `some_object.user = a_user`
- First calls the parent class's implementation of the same method with `super`
- Then sets the object's account to match the user's account
- Finally sets the application's time zone to match the user's time zone

Setter methods are a core part of Ruby's object-oriented design, providing a clean way to control how attributes are updated while maintaining a simple assignment-like syntax for the caller.

### Relationship Between Setter Methods and `attr_writer`

Both provide a way to assign values to instance variables with proper encapsulation. Both allow the assignment syntax: `object.name = "John"`. 

#### Setter Method (Manual Definition)

```ruby
def name=(value)
	@name = value
end
```

`attr_writer` is a convenient shorthand that automatically generates setter methods for you. `attr_writer :name` generates exactly the same setter method shown above. 

#### `attr_writer` Equivalent

```ruby
attr_writer :name
```

Manual setter methods let you add custom logic (validation, transformation, side effects). `attr_writer` provides simple assignment with no additional processing. Use `attr_writer` when you just need basic assignment, and write custom setter methods when you need to control how values are processed. A custom setter method like the `user=` example is necessary when do more than simple assignment is needed, such as updating related attributes or triggering side effects.

### Related Macros

`attr_accessor` creates both getter and setter methods.

`attr_reader` creates only getter methods.

`attr_writer` creates only setter methods.

## Argument forwarding with `(...)`

```ruby
def #{method}(...)
```

The `...` (triple dot) syntax is Ruby's argument forwarding feature. It captures all arguments passed to the method (positional, keyword, and block arguments) and forwards them intact to another method

When `super` is called inside the method, all the original arguments are passed to the parent class's implementation of the same method

This is more concise than writing:
```ruby
def some_method(*args, **kwargs, &block)
    # code here
    super(*args, **kwargs, &block)
end
```

It's particularly useful in this metaprogramming context where the method signature might be complex or could change in future Rails versions.
