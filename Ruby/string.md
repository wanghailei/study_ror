



# String

## `<<` vs `+` for concatenation



```ruby
# Excerpt from: module ActiveSupport > class CurrentAttributes > def attribute

owner.define_cached_method(name, namespace: :current_attributes) do |batch|
    batch <<
        "def #{name}" <<
            "@attributes[:#{name}]" <<
        "end"
end
```
In this code from `ActiveSupport::CurrentAttributes`, the `<<` operator is being used for string concatenation to build a method definition dynamically.

The `define_cached_method` is receiving a block that uses a "`batch`" parameter. This `batch` appears to be a string builder or buffer object that collects parts of the method definition using string concatenation.

Breaking it down line by line:

```ruby
batch <<
    "def #{name}" <<
    	"@attributes[:#{name}]" <<
    "end"
```

This is concatenating four strings together:
1. Whatever was already in the batch
2. `"def #{name}"` - The method definition header with the dynamic name
3. `"@attributes[:#{name}]"` - The method body that retrieves a value from the @attributes hash
4. `"end"` - The end of the method definition

The `<<` operator here is the string concatenation operator, which appends the right operand to the left operand. The method is being built as a string that will later be evaluated (likely using `class_eval` or similar) to define the actual method.

This is a Ruby metaprogramming technique to dynamically define accessor methods for attributes stored in the @attributes hash.

### Why `<<` instead of `+`

1. **Performance**: The `<<` operator modifies the string in place (mutating the original object), while `+` creates a new string object each time. When concatenating multiple strings, using `+` would create several intermediate string objects, leading to more memory allocation and garbage collection.

2. **Memory efficiency**: With `<<`, the same string object is continuously appended to, resulting in only one string object throughout the entire operation.

3. **Builder pattern**: This approach follows a "builder" pattern where an object is progressively built up through multiple operations. The `<<` operator allows for method chaining in a more readable way.

If the code used `+` instead, it would look like:
```ruby
batch = batch + "def #{name}" + "@attributes[:#{name}]" + "end"
```

This would be less efficient as it creates three new intermediate string objects, whereas the `<<` version modifies the same object three times without creating intermediates.

In Ruby, when building up strings through multiple concatenations, `<<` is almost always preferred for performance reasons, especially in framework code like Rails that might be executed frequently.

### When `+` is valuable:

While `<<` is more efficient for building strings through multiple concatenations, the `+` operator for strings still has important uses in Ruby:

1. **Immutability**: When you need the original string to remain unchanged, `+` creates a new string without modifying the operands.

2. **Expression evaluation**: In expressions where you need the result without modifying existing variables: `full_name = first_name + " " + last_name`

3. **Method chaining**: When working with method return values that you can't mutate: `"Hello, " + user.name + "!"`

4. **One-off concatenations**: For simple, one-time string joining where performance isn't critical

5. **Readability**: Sometimes `+` can be more readable, especially in mathematical-like expressions

6. **Frozen strings**: When strings are frozen (immutable), you can't use `<<` but can use `+`

Each has its purpose: `<<` for building strings efficiently through multiple operations, and `+` for non-destructive concatenation when you need a new string without changing the originals.

The right tool depends on the specific context and requirements of your code.



## `#{}` for string interpolation

The `#{}` syntax in Ruby is string interpolation. It allows you to embed Ruby expressions directly within a string. When Ruby processes a string that contains `#{}`, it evaluates the Ruby code inside the curly braces and converts the result to a string, which then becomes part of the larger string.

For example:

```ruby
# ActionView::Helpers::TagHelper.tag
"<#{name}#{tag_builder.tag_options(options, escape) if options}#{open ? ">" : " />"}"
```

There are three instances of string interpolation:

1. `#{name}` - Inserts the value of the `name` variable
2. `#{tag_builder.tag_options(options, escape) if options}` - Calls the method `tag_options` and inserts its result if `options` is not nil
3. `#{open ? ">" : " />"}` - Inserts ">" if `open` is true, otherwise inserts " />"

This allows you to construct HTML tags dynamically based on the method parameters. For example, if `name` is "div", `options` is `{ class: "container" }`, and `open` is true, this would evaluate to:

```
"<div class=\"container\">"
```

String interpolation is one of Ruby's most useful features for building strings dynamically.