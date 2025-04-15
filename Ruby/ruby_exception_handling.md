# Exception Handling





## `rescue`





### `retry`

In Ruby, `retry` is a keyword. It has special meaning in the context of exception handling. When used within a `rescue` block, it causes the `begin` block to be executed again from the beginning.

Here's how it works:
- When used inside a `rescue` block, `retry` jumps back to the beginning of the corresponding `begin` block.
- It restarts the execution of the code in that block.
- This allows you to attempt an operation again after handling an exception.

For example, in the code I provided:

```ruby
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
        sleep(attempts)
        retry  # This is the keyword
    else
        puts "Max retries reached. Giving up."
        raise e
    end
end
```

The `retry` keyword causes execution to jump back to the beginning of the `begin` block, creating the retry loop logic.
