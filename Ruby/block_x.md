# Returning from Ruby Blocks, Procs and Lambdas

How does `return` work within blocks, procs and lambda?

September 29, 2025 · 3 min · Updated: October 20, 2025

```ruby
fruits = ["apple", "orange", "kiwi", "mango", "banana"]

fruits.each do |fruit|
	puts "I like to eat #{fruit}" unless fruit == "kiwi"
end

```

This is a simple block, and if you don’t feel you understand them because you’re new to Ruby (like me at the time of this writing), you should read [this StackOverflow answer](https://stackoverflow.com/a/4911787).

Let’s add a return to this.

```ruby
fruits = %w[apple orange kiwi mango banana]

fruits.each do |fruit|
	puts "I like to eat #{fruit}" unless fruit == "kiwi"
	return "unacceptable" if fruit == "kiwi"
end

```

The `return` here works like a `break` in a traditional loop. It doesn’t really `return` anything since the `each` method isn’t returning anything. However, this exits out of the scope that’s using the block. Which means that this `return` is just exitting out of my script here.

Let’s put this inside a function to make things easier for us to snoop around.

```ruby
def main
    fruits = %w[apple orange kiwi mango banana]

    fruits.each do |fruit|
    	puts "I like to eat #{fruit}" unless fruit == 'kiwi'
    	return fruit if fruit == 'kiwi'
    end
end

fave_fruit = main
puts "I love #{fave_fruit}!"
```

Here, the method exits after 2 runs, and I get the value of `fave_fruit`.

==The `return` exits us out of the scope that it’s called it.== Quite neat. But I’m certain this is going to trip me up some day so I’m writing it down.

If I were to do this with a Proc:

```ruby
def main
    fruits = %w[apple orange kiwi mango banana]

    fruit_proc = Proc.new do |fruit|
        puts "I like to eat #{fruit}" unless fruit == 'kiwi'
    	return fruit if fruit == 'kiwi'
    end

    fruits.each(&fruit_proc)
    puts "I've processed all the fruits."
end

fave_fruit = main
puts "I love #{fave_fruit}!"
```

```
$ ruby code/posts/ruby-block-return/ex_1.rb
I like to eat apple
I like to eat orange
I love kiwi!
```

This block does the same thing, with procs. Notice how the `puts "I've processed all the fruits."` line is not printing.

Let’s try this with a lambda:

```ruby
def main
  fruits = %w[apple orange kiwi mango banana]
  fruit_lambda = lambda do |fruit|
    puts "I like to eat #{fruit}" unless fruit == 'kiwi'
    fruit if fruit == 'kiwi'
  end
  fruits.each(&fruit_lambda)
  puts "I've processed all the fruits."
end
fave_fruit = main
puts "I love #{fave_fruit}!" unless fave_fruit.nil?

```

The output is very different now.

```
❯ ruby code/posts/ruby-block-return/ex_1.rb
I like to eat apple
I like to eat orange
I like to eat mango
I like to eat banana
I've processed all the fruits.
```

It doesn’t get the return value `fruit if fruit == 'kiwi'` at all now.

==A `lambda` is useful when you want to eject only out of the current loop call, not the entire scope that you’re calling this from.==



# Ruby Blocks

How to start really getting Ruby, especially blocks.

October 20, 2025



I think I’m really starting to *get* Ruby. At least I can read Ruby and tell you what’s happening.

I haven’t read a single tutorial on RSpec, the testing framework we use at Chatwoot for the Rails backend. I didn’t want to spend too much time in tutorial hell, and I’m sure that I *should* read more about it soon.

I’ve written about Ruby blocks [before](https://tech.stonecharioteer.com/posts/2025/ruby-block-return/), but I think it bears repeating. This is a method call with a block as an input.

```ruby
perform "input_value" do
	puts "I'll get called with that function"
end
```

Don’t worry about how the function is *implemented* for now. I’m hand-waving this so that you can notice something.

This is also a method call with a block as an input.

| ``   | `it("can do something") { puts "The cake is a lie" } ` |
| ---- | ------------------------------------------------------ |
|      |                                                        |

But here’s something else that’s a method call with a block as an input.

In `lib/calculator.rb`:

| ``   | `class Calculator  def add(a, b)    a + b  end end ` |
| ---- | ---------------------------------------------------- |
|      |                                                      |

In `spec/calculator_spec.rb`:

| ``   | `require_relative "../lib/calculator" RSpec.describe Calculator do  it "adds two numbers" do    calc = Calculator.new    expect(calc.add(2, 3)).to eq(5)  end end ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The highlighted lines include a method call to `it` with a block.

Look at that again. It’s a *method* call.

That is crazy. It is so sublime that I can’t explain how excited this makes me. If I have to teach [Ruby to a Pythonista](https://tech.stonecharioteer.com/posts/2025/ruby/) I’d ask them to ensure they *see* what this is. Until you grok this, it won’t matter how much you try to understand Ruby. Ruby’s readability comes from this feature. I’m still not sold on RSpec though, but I *am* open to learning it because it is, ultimately, Ruby.

## Block Magic

Let’s *really* see what you can do with blocks.

### Building Your Own Language

| ``   | `3.times do  puts "Hello" end 5.times { puts "World" } 10.downto(1) { |i| puts i } ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

These are *all* method calls on *integers*. `times` is a method. `downto` is a method. And every method takes blocks.

But we can also take this to the monke.

| ``   | `class Integer  def seconds     self  end  def minutes    self * 60  end  def from_now    Time.now + self  end end meeting_time = 30.minutes.from_now puts "Meeting is in 30 minutes: #{meeting_time}" ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

That line is *readable*, not because Ruby is magically more readable than other languages, but because we’re daisy-chaining methods we implemented *on the built-in type*. We gave `Integer` wings.

We created a mini language that looks like it somehow isn’t Ruby but, sweet God in Vaikuntha, it *really* is.

### Resource Management

In Python I’m used to doing this.

| ``   | `with open("data.txt", "w") as file:  file.write("This is some data\n")  file.write("This is some more data\n") ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

But you’d be surprised *how much production code* is in the wild that doesn’t use this. You *should* be writing it like this in Python, but `with` is a language keyword in Python.

What would I do in Ruby?

| ``   | `File.open("data.txt", "w") do |file|   file.puts "This is some data"  file.puts "This is some more data" end ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`File.open` is a regular old Ruby method. It takes block. *THEY ALL TAKE BLOCKS*.

| ``   | `def with_timer(name)  start_time = Time.now  yield  end_time = Time.now  puts "#{name} took #{end_time - start_time} seconds"  end with_timer("Database query") do  # Some expensive operation  sleep(2) end ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

That just added a neat side-effect to the database operation.

### Building a DSL

Now let’s really turn things up.

| ``   | `class TodoList  def initialize    @tasks = []  end   def task(description, &block)    task_obj = Task.new(description)    task_obj.instance_eval(&block) if block    @tasks << task_obj  end   def show    @tasks.each { |t| puts t}  end end class Task  attr_accessor :priority, :due_date, :description   def initialize(description)    @description = description  end   def priority(level)    @priority = level  end   def due(date)    @due_date = date  end   def to_s    "#{@description} (Priority: #{@priority}, Due: #{@due_date})"  end end  # let's use this def todo(&block)  list = TodoList.new  list.instance_eval(&block)  list end my_todos = todo do  task "Write a blog post" do    priority "high"    due "2025-10-15"  end    task "Review PR" do    priority "medium"    due "2025-10-14"  end end my_todos.show ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Just stop reading and look at that syntax.

I had a problem when I first tried learning Ruby in 2021. I had tried, because I was trying to just learn a new programming language (as one should definitely do annually), and I was a little gob-smacked because I didn’t understand the `config.rb` file from Rails. It had a bunch of things like this.

It looked like a damned configuration file.

Django *tries* to do this with the `settings.py` file, but it doesn’t succeed.

This is *exactly* why Rails routing looks like this.

| ``   | `Rails.application.routes.draw do  resources :users do    member do      get :profile      post :activate    end  end   namespace :admin do    resources :posts  end end ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

That’s not something special from Rails. That’s just a bunch of *Ruby methods taking blocks!* The `draw`, `resources`, `member`, `namespace`, they’re *all just regular methods.*

All of this looked like some magical DSL to me. But it was **not**. It was just regular old Ruby.

### Custom Control Flow

Okay let’s make our own `unless` or `if` syntax. Just because I’m feeling like it.

| ``   | `def only_on_weekdays  yield unless [0, 6].include?(Time.now.wday) end only_on_weekdays do  puts "Let's get to work!" end ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Or maybe a `retry` function?

| ``   | `def with_retry(max_attempts: 3)  attempts = 0  begin    attempts += 1    yield  rescue StandardError => e    if attempts < max_attempts      puts "Attempt #{attempts} failed, retrying..."      retry    else      puts "Max attempts reached, giving up"      raise e    end  end end with_retry(max_attempts: 3) do  # some unreliable API   puts "Attempting connection..."  raise "Connection failed" if rand > 0.7  puts "Success!" end ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### Putting this all together

| ``   | `(1..5).select { |n| n.even? }  .map { |n| n * 2 }  .reduce(0) { |sum, n| sum + n } ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Each and every one of those was a method call **that took a block!**

It’s all a block. Once I saw this pattern, I could not *unsee it!*

And why would I want to unsee it? It’s beautiful. It sparkles, damn it.

💡The Real Magic

The magic isn’t just that you can do this. It’s that Ruby’s syntax makes it so natural that blocks become invisible. When you write 5.times { puts “Hello” }, you don’t think “I’m calling the times method and passing it a block.” You think “I’m doing something 5 times.” You read RSpec code and think “Yeah, that reads like English.”

That’s the genius of Ruby. The language gets out of your way and lets you think about the problem, not the syntax.

------

## Other Posts in This Series

- [Ruby](https://tech.stonecharioteer.com/posts/2025/ruby/)
- [Returning from Ruby Blocks, Procs and Lambdas](https://tech.stonecharioteer.com/posts/2025/ruby-block-return/)
- [Some Smalltalk about Ruby Loops](https://tech.stonecharioteer.com/posts/2025/ruby-loops/)
- [Symbolic Ruby](https://tech.stonecharioteer.com/posts/2025/ruby-symbols/)