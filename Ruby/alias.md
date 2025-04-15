## The `alias` Method in Ruby

The `alias` keyword in Ruby allows you ==to create an alternative name for an existing method or global variable==. It's a powerful metaprogramming feature that enables method name flexibility and supports backward compatibility.

### Basic Syntax

There are two forms of the `alias` keyword:

```ruby
# Form 1: with bare method names
alias new_name original_name

# Form 2: with symbols or expressions (preferred by many)
alias :new_name :original_name
```

### How `alias` Works

When you alias a method:
- Ruby creates a copy of the method with a new name.
- Both methods refer to the same functionality.
- ==Changes to one don't affect the other.==
- The alias is created at the moment the code is interpreted.

### Method Alias

```ruby
class Person
  def full_name
    "John Doe"
  end
  
  # Create an alias
  alias name full_name
end

person = Person.new
puts person.full_name  # => "John Doe"
puts person.name       # => "John Doe"
```

### Aliases in Module Extensions

```ruby
module Logging
  def log(message)
    puts "[LOG] #{message}"
  end
  
  # Provide alternative names
  alias write log
  alias record log
end

class App
  include Logging
end

app = App.new
app.log("Started")     # => [LOG] Started
app.write("Running")   # => [LOG] Running
app.record("Stopped")  # => [LOG] Stopped
```

### Preserving Original Behavior When Overriding

Here's a practical example of ==using `alias` to preserve original behavior while extending functionality== in a web application context:

```ruby
class HttpClient
  def initialize
    @headers = {}
  end
  
  def request(url, method: :get, data: nil)
    puts "Making #{method.upcase} request to #{url}"
    puts "With data: #{data}" if data
    puts "Headers: #{@headers}"
    # ... actual request code would go here
    
    return { status: 200, body: "Response data" }
  end
end

# Now we want to extend the HttpClient with automatic retry functionality
class RetryableHttpClient < HttpClient
    # Preserve the original request method
    alias original_request request
  
  # Override with new retry functionality
  def request(url, method: :get, data: nil, retries: 3)
    attempts = 0
    
    begin
      attempts += 1
      puts "Attempt #{attempts} of #{retries + 1}"
      
      # Call the original behavior
      response = original_request(url, method: method, data: data)
      return response
      
    rescue StandardError => e
      puts "Request failed: #{e.message}"
      
      if attempts <= retries
        puts "Retrying in #{attempts} seconds..."
        sleep(attempts)  # Exponential backoff
        retry
      else
        puts "Max retries reached. Giving up."
        raise e
      end
    end
  end
end

# Using the enhanced client
client = RetryableHttpClient.new
result = client.request("https://api.example.com/data", 
                       method: :post, 
                       data: {user_id: 123}, 
                       retries: 2)
puts "Final result: #{result}"
```

In this example:

1. We start with a basic `HttpClient` class that can make HTTP requests
2. We create a subclass `RetryableHttpClient` that adds retry functionality
3. Using `alias`, we preserve the original `request` method as `original_request`
4. We override `request` to add retry logic while still using the original behavior
5. The new method accepts an additional `retries` parameter that wasn't in the original

This pattern is particularly useful in real-world applications for:

- Adding logging around existing methods
- Implementing retry logic around network calls
- Adding validation before calling the original behavior
- Implementing caching while preserving the original data-fetching logic
- Adding performance monitoring around critical methods

The key benefit is that we've extended the functionality without duplicating the core implementation, making our code more maintainable and following the DRY (Don't Repeat Yourself) principle.



## `alias` vs. `alias_method`

Ruby also provides `alias_method`, which is a method rather than a keyword:

```ruby
class Example
  def hello
    "Hello"
  end
  
  # Using alias_method instead of alias
  alias_method :greet, :hello
end
```

Key differences:
- `alias` is a keyword evaluated when the code is parsed
- `alias_method` is a method evaluated at runtime
- `alias_method` respects the current value of `self`
- `alias_method` can take variables and expressions

## Common Use Cases

1. **API Compatibility**: Maintaining backward compatibility when renaming methods
2. **Method Decoration**: Preserving original behavior while adding new functionality
3. **DSL Creation**: Providing alternative, more natural-language-like method names 
4. **Method Delegation**: Creating shorter or more contextual names for existing methods

## Best Practices

- Document your aliases to avoid confusion
- Use `alias_method` when working with dynamic method names or in class methods
- Consider using `alias` in simpler, static cases
- Be aware that aliases create copies, so they won't inherit changes to the original method

The `alias` keyword is a powerful Ruby feature that enables elegant metaprogramming solutions when used appropriately.