

# Array



## Set Subtraction

In Ruby, when you use the set subtraction operator (`-`) between two arrays, it returns a new array with elements from the first array that are not in the second array. It's a way to "subtract" one collection from another.

```ruby
[1, 2, 3, 4] - [2, 4]  # => [1, 3]
```

1. **Order Preservation**: The result maintains the order of elements as they appear in the first array.
2. **Element Uniqueness**: The operation automatically removes duplicates from the result.
3. **Non-Destructive**: The original arrays remain unchanged; a new array is returned.**

Ruby uses the `==` method to determine equality between elements.**

### Multiple Array Subtraction

You can chain multiple subtractions:

```
[1, 2, 3, 4, 5] - [2, 3] - [4]  # => [1, 5]
```

### With Objects

When working with objects like modules, Ruby compares object identity:

```
class Module1; end
class Module2; end

m1 = Module1.new
m2 = Module2.new

[m1, m2] - [m2]  # => [m1]
```

### Example in Rails

```ruby
def self.without_modules(*modules)
	modules = modules.map do |m|
		m.is_a?(Symbol) ? ActionController.const_get(m) : m
	end

	MODULES - modules
end
```

In the `without_modules` method, the set subtraction is being used to:

1. Take the complete list of modules (`MODULES`) that would normally be included in `ActionController::Base`
2. Remove any modules that the developer has specified they don't want
3. Return the remaining modules to be included in the custom controller

This allows developers to easily create custom controller base classes by excluding specific modules rather than having to manually include each desired module.