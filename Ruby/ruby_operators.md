# Operators

Ruby enables several elegant return patterns, which reflects Ruby's design philosophy:

1. ==**Expressions Over Statements**==: Everything is an expression with a value.
2. **Declarative Over Imperative**: Describing what we want rather than how to get it.

This is why many Ruby developers find the language so expressive - it lets you write code that reads almost like natural language while still being precise and powerful.

## The `||=` so-fucking-called Memoization pattern



```ruby
def expensive_calculation
	@result ||= calculate_result
end

private

def calculate_result
    # Complex calculation that takes time
    sleep(1) # Simulating expensive work
    42
end
```

The method checks if `@result` is already set (not `nil` or `false`). If set, return its value without performing the calculation. If not set, execute the calculation and store the result in `@result`, and return the newly calculated value.

Thanks to short-circuit evaluation, if `@result` is truthy (not `nil` or `false`), the right side (`calculate_result`) is never executed. 

Behind the Scenes, the expression `@result ||= calculate_result` is syntactic sugar for:

```ruby
@result = @result || calculate_result
```

And this pattern has a bullshit acadamical name callde the Memoization pattern. 

```ruby
# Memoization pattern # Fuck you!
@calculated_value ||= expensive_calculation
```

It's actually just default assignment.

```ruby
# Default value pattern # That's human's words.
options ||= {}
```

There are many examples in the real world.

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

### Why is it called the Memoization pattern or caching mechanism？

To me, it's just a way to assign an object to a variable if it doesn't already exist. ==Technically, at its core, the `||=` pattern is indeed just assigning a value to a variable if it doesn't already exist.== At the end of the day, it's exactly as you describe - a conditional assignment that happens to be extremely useful for avoiding redundant work.

Why It's Called "Memoization"? The term "memoization" comes from computer science and has a specific meaning. The jardon originally comes from "memo" (as in memorandum or reminder). It refers to ==storing the results of expensive function calls and returning the cached result when the same inputs occur again.== The purpose is to avoid redundant calculations and improve performance.

% ==I believe they are just bullshit jargon.== 這就像在日常說話中提出「我這句話是一個定語從句」般地傻逼。 %

The terminology around it, like memoization, can seem unnecessarily complex. Many Ruby patterns have academic-sounding names that can obscure their simplicity.  A straightforward description actually helps demystify the concept.

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

##
