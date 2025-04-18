# Operators

Ruby enables several elegant return patterns, which reflects Ruby's design philosophy:

1. ==**Expressions Over Statements**==: Everything is an expression with a value.
2. **Declarative Over Imperative**: Describing what we want rather than how to get it.

This is why many Ruby developers find the language so expressive - it lets you write code that reads almost like natural language while still being precise and powerful.

% Pattern 應該被翻譯成「方式」或「用法」。其實很簡單的事兒，搞了個不明覺厲的傻逼概念。 %

## The `||=` ~~so-fucking-called Memoization~~ pattern 有就不給了

The conditional assignment operator (`||=`) means "assign only if the left side is nil or false". It is commonly used to provide default values, to ensures the attribute is never `nil`.

```ruby
before_create { self.last_active_at ||= Time.now }
```

This means: "Before creating a new session record, if the `last_active_at` attribute hasn't been set yet, set it to the current time."

```ruby
def expensive_calculation
	@result ||= calculate_result
end
```

==The method checks if `@result` is already set (not `nil` or `false`). If set, return its value without performing the calculation. If not set, execute the calculation and store the result in `@result`, and return the newly calculated value.==

Behind the Scenes, the expression `@result ||= calculate_result` is syntactic sugar for `@result = @result || calculate_result`. ==Technically, at its core, the `||=` pattern is indeed just assigning a value to a variable if it doesn't already exist.==

At the end of the day, to me, it's just a way of assigning an object to a variable if it doesn't already exist, namely **default assignment** or conditional assignment.

### The true value of this pattern

If left side is truthy (not `nil` or `false`), the right side is never executed. 如果杯子裡有水就不用添了。

This is why it's called memoization or caching mechanism？The term "memoization" is a bullshit jardon originally comes from "memo" (as in memorandum or reminder) from computer science. It has a specific meaning - ==storing the results of expensive function calls and returning the cached result when the same inputs occur again==, to avoid redundant calculations and improve performance.

### There are many examples in the real world.

```ruby
# Default value pattern # That's human's words.
options ||= {}
```

```ruby
#  Method Results Caching. # Cache my ass.
def user_privileges
	@user_privileges ||= calculate_user_privileges
end
```

```ruby
# Lazy Initialization # Fuck you!
class ShoppingCart
    def items
		@items ||= []
    end
end
```

```ruby
# Common pattern in Rails views
def current_user_greeting
	@current_user_greeting ||= "Hello, #{current_user.name}!"
end
```

```ruby
# Puma
def self.abstract_unix_socket?
    # Here, ||= is being used to memoize the result of the abstract unix socket check.
    @abstract_unix ||=
    if HAS_UNIX_SOCKET
	    begin
    		::UNIXServer.new("\0puma.temp.unix").close
	    	true
    	rescue ArgumentError  # darwin
    		false
		end
    else
    	false
    end
end
```

```ruby
# Memoization with Arguments Pattern
def calculate(input)
    @calculations ||= {}
	@calculations[input] ||= perform_calculation(input)
end
```

```ruby
#  Class-Level Memoization
def self.system_config
	@@system_config ||= load_system_config
end
```

```ruby
# Rails' dedicated `memoize` helper method.
class User < ApplicationRecord
    def complex_stats
	    expensive_calculation
    end
    memoize :complex_stats
end
```

### Gotchas and Edge Cases

#### Falsey Values

The pattern doesn't work correctly if `false` or `nil` are valid return values:

```ruby
def user_active?
	@user_active ||= check_if_user_active  # Will recalculate if result was false!
end
```

Fix using ==explicit nil checking==:

```ruby
def user_active?
	return @user_active unless @user_active.nil?
	@user_active = check_if_user_active
end
```

#### Thread Safety

==Basic memoization isn't thread-safe.== For thread safety:

```ruby
def expensive_calculation
    @result_mutex ||= Mutex.new
    @result_mutex.synchronize do
	    @result ||= calculate_result
    end
end
```

## The logical OR (`||`) return

The method's purpose is to check if any condition signals a change. This method returns `true` if **either** of these conditions is true. The `||` operator directly expresses this "any of these conditions" logic in the code structure itself. 

```ruby
# ActiveRecord
def target_changed?
	owner.attribute_changed?(reflection.foreign_key) || (!foreign_key_present? && target&.new_record?)
end
```

Without `||` as return value, it takes 7 lines versus 1 line!

```ruby
def target_changed?
    if owner.attribute_changed?(reflection.foreign_key)
    	return true
    end
    if !foreign_key_present? && target&.new_record?
    	return true
    end
    return false
end
```

The use of logical operator `||` for returning results is a concise, expressive and elegant feature of Ruby. What's happening here is a combination of Ruby features that creates a particularly elegant pattern:

1. **Expression-Based Return Values**: Ruby implicitly returns the value of the last expression evaluated in a method.
2. **Short-Circuit Evaluation**: `||` only evaluates the right side if the left side is falsey.
3. **Expressions as Control Flow**: The logical operators themselves control which code executes.

The `||` pattern is a perfect example of ==how Ruby blurs the line between control flow and value calculation==, making code both more concise and more directly aligned with the underlying logic.

## The Ternary Return 

```ruby
def status
	active? ? "Active" : "Inactive"
end
```

所謂「三元回復」，就是一種簡寫而已。

## The `&&` Guard Pattern

```ruby
def process_user(user)
	user && user.active? && user.process_data
end
```

## The safe navigation operator (`&.`)

```ruby
target&.new_record?
```

The `&.` is the safe navigation operator, which only calls `new_record?` if `target` is not ·.

## The assignment operatior (`=`)

Ruby allows significant flexibility with whitespace and line breaks. The `if` statement can be placed on a new line after the assignment operator (`=`). The line break does not affect the legality of the syntax. It’s a common practice in Ruby to format code this way, and it is entirely syntactically correct.

```ruby
# ActiveRecord

association_class =
if reflection.polymorphic?
    owner.public_send(reflection.foreign_type)
else
    reflection.klass
end
```

## The inheritance indicator (`<`)

These are two different uses of the same symbol "`<`" - the implementation as an inheritance indicator is part of Ruby's syntax, while ==the comparison operator is a method==.

The inheritance indicator (`<`) is used in class definitions to specify inheritance. In the following `if` expression, ==the `<` is checking inheritance hierarchy==, and it returns `true` if `model_was` is a subclass of `ActiveRecord::Base`.

```ruby
# ActiveRecord

if foreign_key_was && model_was < ActiveRecord::Base
	update_counters_via_scope(model_was, foreign_key_was, -1)
end
```

Another example: 

```ruby
class Person
end
class User < Person
end

User < Person # => This express returns true.
```



## The `&` operator

`[2, 4, 6].select!( &:even? )` is equivalent to `[2, 4, 6].select! { |i| i.even? }`

The `&:even?` is shorthand for a block `{ |i| i.even? }`.

`:even?` is a method that returns `true` if an integer is even and `false` otherwise. The colon `:` before `even?` indicates that it is a symbol, namely, ==the `:` before `even?` converts the name of the method into a symbol.== So, `:even?` represents the symbol form of method `even?`. The `even?` is an instance method of the `Integer` class.

The ampersand (`&`) is a unary operator that converts the symbol to a block. When used in front of a symbol in a method call like this, it tries to call a method with the same name as the symbol on the objects being iterated over. 

==The `&` operator is used to convert the symbol `:even?` into a Proc object==, which is then treated as a block by the method it's passed to, and which is then passed to the `select!` method. In this case, it's calling `even?` on each element of the array.

Since `!` is used after `select`, it modifies the original array itself.





