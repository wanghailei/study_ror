# Conditional Operation

## The Trailing `if` Clause

The following line demonstrates Ruby's **conditional modifier** syntax (also called a **trailing conditional**):

```ruby
model_was = owner.class.polymorphic_class_for(model_type_was) if model_type_was
```

This line conditionally sets `model_was` only if `model_type_was` is truthy (not `nil` or `false`). It's equivalent to:

```ruby
if model_type_was
	model_was = owner.class.polymorphic_class_for(model_type_was)
end
```

Or use a ternary operator if you need to ensure `model_was` is always assigned:

```ruby
model_was = model_type_was ? owner.class.polymorphic_class_for(model_type_was) : nil
```

The trailing `if` condition is a Ruby idiom that allows for concise, expressive code. It puts the main action first, then the condition. There are many common ussage of this pattern.

```ruby
# Guard Against Nil/Empty Values
return if array.empty?
process_user(user) if user.active?
```

```ruby
# Conditional Assignment
total += item.price if item.taxable?
user.role = 'admin' if user.admin_requested?
```

```ruby
# Conditional Method Calls
errors.add(:base, "Invalid data") if validation_failed?
logger.debug("Processing item #{item.id}") if debug_mode?
```





## Assignment within a condition



```ruby
# ActiveRecord::Associations::Association
elsif (scope = klass.current_scope) && scope.try(:proxy_association) == self
```

This actually does two things at once:

1. **Assignment within a condition**: `klass.current_scope` is assigned to the local variable `scope`.
2. **Evaluation of the assignment result**: Then that assigned value is immediately used in the condition.

This is equivalent to:
```ruby
scope = klass.current_scope
if scope && scope.try(:proxy_association) == self
	# code here
end
```

The syntax is a shorthand way to:
- Assign a value to a variable;
- Test if that value is truthy (not nil or false);
- Do additional tests on that value if it exists.

So in this case, it assigns `klass.current_scope` to `scope`, and if that value exists (is not nil), it then checks if `scope.try(:proxy_association) == self`.

This pattern is commonly used in Ruby to make code more concise, though some find it less readable than the expanded version.

## Safe Navigation pattern

The following code is safely checking if `scope` has a `proxy_association` method and if its return value equals `self`. If `scope` doesn't respond to `proxy_association`, it would return `nil` instead of raising an error.

```ruby
scope.try(:proxy_association) == self
```

The safe navigation pattern, also called "null-safe navigation", is a programming technique to safely access methods or properties on potentially `nil` objects ==without raising errors==. The safe navigation pattern makes code more concise and resilient. In Ruby, this is implemented through several mechanisms:

### 1. The Safe Navigation Operator (`&.`)

Ruby 2.3+ introduced the `&.` operator, which is the clearest implementation of this pattern:

```ruby
# Without safe navigation
user.profile.address.city if user && user.profile && user.profile.address
# ↓ becomes ↓
user&.profile&.address&.city
```

This operator checks if the object is `nil` before attempting to call the method. If the object is `nil`, it returns `nil` instead of raising a `NoMethodError`.

### 2. The `try` Method (in Rails)

Before the safe navigation operator, Rails introduced the `try` method:

```ruby
# Instead of
user.name if user
# You can write
user.try(:name)
```

The `try` method:
- Calls the specified method if the object is not nil
- Returns nil if the object is nil
- Can also be used with a block: `user.try { |u| u.name.upcase }`

#### The `try!` Variant

Rails also has `try!` which is similar to `try` but only protects against nil, not missing methods:
- `nil.try(:nonexistent_method)` returns nil
- `object.try(:nonexistent_method)` returns nil
- `object.try!(:nonexistent_method)` raises NoMethodError





## Never use elsif because it's ugly.

```ruby
if
    #
else
    #
end

# OR ONLY

if
    #
else
    if
        #
    else
        #
    end
end
```

Another solution is case...when

```ruby
grade = 
case score
when 90..100 then 
    "A"
when 80..89 then 
    "B"
else 
    "F"
end
```

