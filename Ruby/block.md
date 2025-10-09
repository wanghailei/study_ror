# Block, Proc, Lambda

## Blocks are not objects?

One of Ruby's core principles is that "everything is an object." However, blocks are actually one of the few exceptions to this rule, along with things like keywords and operators. In Ruby, **blocks are not objects themselves**. 

- Some said, "blocks are closures and do not belong to an object." 
- Some describes blocks as "syntax literals for Proc objects," implying that blocks are a way to create Proc objects but are not objects themselves.
- Some suggest that "in Ruby virtually everything is an object, even blocks (in the form of Proc instances)." This statement, however, refers to blocks after conversion, not blocks in their raw form.

Blocks are syntactical constructs rather than objects themselves. They don't have object IDs and cannot be directly manipulated like other Ruby objects.

A Proc object is an encapsulation of a block of code, which can be invoked as if it were a method.

The distinction is important because:

1. Blocks can only be passed directly to methods that expect them using the `yield` keyword or as the last argument with an ampersand (`&`)
2. Procs are full-fledged objects that can be stored in variables, passed around, and called later.

### Why this design?

This design decision in Ruby allows for:

1. A lightweight syntax for passing code blocks (the block syntax is less verbose)
2. Performance optimization (blocks don't need to allocate memory for objects)
3. A clear distinction between throwaway code (blocks) and reusable code (Procs)??

## What are Blocks indeed?

If you have used `each` before, then you have used blocks!

==A code block is a chunk of code you can pass to a method==, as if the code block were another parameter. 

A code block is a set of Ruby statements and expressions inside braces or a `do ... end` pair. `{...}` is a block. `do ... end` is a block

==A code block may appear only immediately after a method invocation, and the start of the block (the brace or the `do`) must be on the same logical source line as the end of the invocation.==

Blocks are ==a way of creating `Proc` objects==. Proc objects are instructions between `{}` or `do...end` which represent code that can be used by other code.

我感覺，一個block就像一個snippet，把一段代碼紀錄到一個小紙片上而已。

A block does not have its own name.

A block cannot be stored directly. A Proc object can be stored in a variable.

## Why do blocks exist?

A block is useful because it allows you to save a bit of logic (code) & use it later.

這麼說的話，感覺 block 像一張小卡片，上面記錄了一些好用的代碼片段或想法，一般是解決某一個小事兒、產生一個結果。所以它被傳來傳去，挺形象的。



You're absolutely correct, and I appreciate the precision. Let me clarify this important distinction:

## A block passed as an argument to a method?

% ==Stricktly speaking, since the `&` operator converted a block to a `Proc` object, what can be used as an argument is a `Proc` object but not a block.== %

So when we write code that appears to pass a block as an argument, we're actually converting a block to a `Proc` object, and pass the `Proc` object as a argument. For example, in this code:

```ruby
def some_method(&lovely_block)  # block is converted to a Proc
    # lovely_block is now a Proc object
    another_method do |x|
    	lovely_block.call(x)  # Using the Proc object
    end
end
```

The lovely_block passed to `some_method` is immediately converted to a `Proc` object by the `&` operator. We're not passing the block itself as an argument but rather a `Proc` object created from that block.

Possibly converting `Proc` objects back to blocks when needed, using `&` again.

```ruby
def pass_along(&received_block)
	another_method(&received_block)  # Convert Proc back to a block
end
```

The `&` operator in the method call converts the `Proc` back to a block syntax that Ruby methods expect.

This distinction between blocks (syntactical constructs) and `Proc` objects (actual Ruby objects) is indeed fundamental to understanding Ruby's block implementation.



## Block's arguments syntax

The block may start with an argument list between vertical bars. 

### `yield 呼叫`

`yield` is a keyword that calls a block when you use it. It’s how methods **USE** blocks! When you use the `yield` keyword, the code inside the block will run to do its job, just like when you call a regular method.

### Block as an Implicit Argument

Notice that there’s no parameter explicitly passed to the ActionView helper method `capture` because Ruby allows you to pass a block as an implicit argument. 

```ruby
<% @greeting = capture do %>
	<p>Welcome! The date and time is <%= Time.current %></p>
<% end %>
```

In Ruby, when you write a method call followed by `do...end` or `{...}`, that block is automatically ==passed to the method as an implicit parameter==. There’s no need for an explicit parameter between the method name and the block, which is a common Ruby idiom for methods that work with blocks.

### Blocks can be “explicit” or “implicit”. 

Explicit means that you give it a name in the parameter list. You can pass an explicit block to another method or save it into a variable to use later.

```ruby
def broadcast( &work )
     work.call # same as yield
end
broadcast { puts "Explicit block called" }
```

Notice the `&work` paramete. That’s how you ==define the block’s name==!

## The Ampersand `&` Operator

`[2, 4, 6].select!( &:even? )` is equivalent to `[2, 4, 6].select! { |i| i.even? }`

The `&:even?` is shorthand for a block `{ |i| i.even? }`.

`:even?` is a method that returns `true` if an integer is even and `false` otherwise. The colon `:` before `even?` indicates that it is a symbol, namely, ==the `:` before `even?` converts the name of the method into a symbol.== So, `:even?` represents the symbol form of method `even?`.

==The `&` operator is used to convert the symbol `:even?` into a Proc object==, which is then passed to the `select!` method. It will allow us to call the method named `even?` on each element of the array.

Since `!` is used after `select`, it modifies the original array itself.

The `&` operator in Ruby is quite versatile and serves different purposes depending on the context, particularly when dealing with blocks, Procs, and methods.

### 1. Converting Blocks to Procs and Vice Versa

**Turning a Block into a Proc**: When defining a method, if you prefix the last parameter with `&`, any block passed to this method is converted to a `Proc` object. This allows you to store the block in a variable, pass it around, or call it later.

```ruby
def my_method &block
	block.call
end
my_method { puts "Hello, world!" } # => "Hello, world!"
```

The method is defined with an argument named `block`, prefixed with `&`. This means that ==if a block is given when the method is called==, it will be converted to a `Proc` object and ==assigned to the parameter `block`==. 

% 當 my_method 被調用時，block 才被賦予了 "Hello, world!" 這個值。 20230113 % 

When a method is defined with a parameter using the `&` syntax, it means that ==the method can accept a block, but it's not mandatory to pass one==. However, if you don't pass a block and then try to call the `block` variable as a `Proc` object (with `block.call`), an error will be encountered because `block` will be `nil` if no block is provided. To handle this case, you should check if the `block` is `nil` before calling it. 

```ruby
def my_method(&block)
	block.call if block
end
```

**Passing a Proc as a Block**: When calling a method that expects a block, you can pass a `Proc` object with an `&` prefix. This converts the `Proc` back into a block.

```ruby
my_proc = proc { puts "Hello from Proc!" }
some_method(&my_proc)
```

Here, `some_method` is called with `my_proc` being converted to a block.

### 2. Converting Method Objects to Blocks

**Referencing a Method**: You can reference a method on an object using the `method` method, which returns a `Method` object. You can then convert this `Method` object to a block by using the `&` operator. This is useful for passing a method as a block to another method.

```ruby
class MyClass
	def my_method
		puts "Method called"
	end
end

obj = MyClass.new
method_obj = obj.method(:my_method)
[1, 2, 3].each(&method_obj)
```

In this example, `method_obj` (a `Method` object) is converted to a block and passed to `each`.

### 3. Symbol to Proc Conversion

**Shorthand for Calling Methods**: The `&` operator is also used in a popular Ruby idiom that converts symbols to blocks, known as the "Symbol to Proc" conversion. This is often used with methods like `map`, `select`, or `each`.

```ruby
[1, 2, 3].map &:to_s
```

In this case, `&:to_s` converts the `:to_s` symbol to a `Proc` that calls the `to_s` method on every object in the array.

#### A real-world example

```ruby
# Rails ActionView::Template::Handlers#template_handler_extensions
@@template_handlers.keys.map(&:to_s).sort
```

==The `&:to_s` syntax is a shorthand for `{ |key| key.to_s }`. It converts each symbol to its string representation (e.g., `:erb` becomes `"erb"`).== 

`.sort` sorts the resulting array of strings alphabetically. 

### Summary

The `&` operator in Ruby is a powerful tool, used for converting blocks to Procs and vice versa, converting method references to blocks, and enabling the concise Symbol to Proc idiom. This operator is a key part of Ruby's flexible and expressive method and block handling capabilities, contributing to the language's reputation for elegant and readable code.
