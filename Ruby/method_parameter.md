# Method Parameters and Arguments

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

## Parameters vs Arguments

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

## Parentheses in Methods

In Ruby, ==parentheses are generally optional in both method definitions and method calls==, with some important exceptions and style considerations. The key is consistency in your codebase.

==When in doubt, using parentheses.==

### In Method Definitions

When defining methods, parentheses around parameters are optional:

```ruby
# Both of these are valid method definitions
def greeting(name)
	puts "Hello, #{name}!"
end
def greeting name
	puts "Hello, #{name}!"
end
```

However, most Ruby style guides recommend using parentheses when defining methods with parameters for clarity.

Omitting parentheses is more commonly accepted when defining methods without parameters.

### In Method Calls

For method calls, parentheses are also generally optional:

```ruby
# These are equivalent
puts("Hello, world!")
puts "Hello, world!"
```

However, there are important cases where parentheses are necessary:

- When chaining methods

```ruby
# Ambiguous what .select applies to
array.map { |x| x * 2 }.select { |x| x > 5 }  
   
# Clearer:
array.map({ |x| x * 2 }).select { |x| x > 5 }
```

- When passing method calls as arguments to other methods

```ruby
puts add 2, 3 	# This is ambiguous.
puts(add(2, 3))	# This is clear.
```

- When there might be ambiguity about method boundaries

```ruby
greeting user world 	# Unclear if "world" is an argument or method call.
greeting(user, world)	# Clear
```