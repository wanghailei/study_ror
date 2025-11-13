# `begin ... end`

In Ruby, a `begin` and `end` pare creates a boundary.

### Creates a boundary for Compound Expression

Multiple lines of code are treated as one unit that produces a value - the value of the last expression evaluated. The entire sequence can be used in places where a single expression is expected. It's useful for complex expressions.

```ruby
begin
    statement1
    statement2
    statement3
end
```

### Creates a boundary for Exception Handling

The most common use of `begin`/`end` is for exception handling (similar to try/catch in other languages):

```ruby
begin
    result = dangerous_operation() # Code that might raise an exception
rescue SomeException => e
    puts "Error: #{e.message}" # Handle the exception
ensure
    cleanup_resources() # This code always runs, regardless of exceptions
end
```

### Use `begin`/`end` with statement modifiers

```ruby
begin
    attempt_connection()
rescue ConnectionError
    retry_count += 1
    retry
end while retry_count < max_retries
```



### begin ... ensure ... end

In Ruby, the `begin … ensure … end` construct is used to guarantee that a block of code will **always run**, no matter whether an exception is raised or not. It’s most often used for cleanup tasks (closing files, releasing resources, rolling back connections, etc.).

```ruby
begin
	# code that may raise exceptions
ensure
	# code that will always run
end
```

For example, even though an exception is raised, the `file.close` inside ensure will always execute. After that, the exception will propagate up unless it’s rescued elsewhere.

```ruby
file = File.open("data.txt", "w")

begin
	file.write("Hello, world!")
	raise "Something went wrong"
ensure
	file.close
	puts "File closed, no matter what"
end
```

### `return` together with `ensure`

In Ruby, **return does end execution of a method**. Once Ruby hits a return, the method immediately exits and gives back the value after return.

When `return` with `ensure`, the method *still* returns immediately, but the ensure block will **always run first**, before giving control back:

```ruby
def demo
    begin
	    return 42
    ensure
	    puts "ensure always runs"
    end
end

puts demo

# output: ensure always runs
# output: 42
```

Output:

```

```





------



👉 Do you want me to also explain how break and next differ from return in Ruby (they sometimes confuse people in loops and blocks)?

### `BEGIN` and `END`

Ruby also has special `BEGIN` and `END` blocks (note the capital letters):

```ruby
BEGIN {
	# Code that runs when the program starts, before any other code
}

END {
	# Code that runs when the program ends, after all other code
}
```



```ruby
=begin

=end
```

