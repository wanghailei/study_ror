# Ruby Demystified: A Clear Mental Model for Learning

## Introduction

==Programming languages are tools for human expression that happen to be understood by computers.== Yet, the way we teach and discuss these languages often introduces unnecessary complexity, creating artificial barriers to learning. Ruby, designed by Yukihiro Matsumoto with the principle of "programmer happiness" in mind, offers an elegant and intuitive approach to programming — if only we could strip away the confusing terminology and conceptual overhead that has accumulated around it.

This essay presents a refined mental model for understanding Ruby, developed through extensive analysis of how the language actually works. By replacing traditional object-oriented jargon with intuitive concepts that align with how people naturally think, we can make Ruby more accessible to beginners and more clearly understood by experienced developers.

## Part I: The Essential Elements of Ruby

### Simplifying Ruby's Core Concepts

At its heart, Ruby can be understood with just three fundamental concepts:

1. **Machines (Classes)** - Devices that create and define objects
2. **Objects** - Things with elements and actions
3. **Variables** - Names that refer to objects

These three concepts form the foundation of Ruby programming. Everything else builds upon them, expanding what we can express but not fundamentally changing the nature of how Ruby works.

### The Problem with Traditional Terminology

Traditional object-oriented terminology introduces unnecessary complexity when learning Ruby:

- "Instantiating objects from classes"
- "Invoking methods on instances"
- "Type-checking with polymorphic inheritance"
- "Encapsulating state in instance variables"

None of these terms appear as keywords in Ruby itself. They represent abstract concepts imported from computer science theory rather than reflecting how Ruby actually works. This disconnect creates a significant learning barrier, requiring programmers to learn not just Ruby but also a conceptual framework that sits awkwardly on top of it.

### A Cleaner Mental Model

By refining our terminology to match Ruby's actual implementation, we create a mental model that's both simpler and more accurate:

```ruby
# Traditional explanation:
# "We instantiate an object of the Person class, set instance variables,
# and invoke instance methods on the object."

# Simplified explanation:
# "We create a person with a name, and ask it to greet us."

person = Person.new("Ruby")
person.greet
```

This simpler explanation doesn't sacrifice technical accuracy—it actually enhances it by focusing on what's really happening in the language rather than the theoretical framework we've imposed upon it.

## Part II: Machines, Molds, and Objects

### The Machine Metaphor

In our refined model, a class in Ruby is best understood as a **machine with a mold**:

- The machine (class) contains a mold defining what objects will look like
- The machine has mechanisms (methods) that created objects can use
- The machine produces objects according to its mold
- Each object has the same structure but can contain different values

This machine metaphor captures the active nature of classes in Ruby. Unlike passive blueprints that merely inform, Ruby classes actively create and define the capabilities of objects.

### Molds Shape Objects

The "mold" component of our metaphor deserves special attention:

```ruby
class Book
  def initialize(title, author, pages)
    @title = title    # Cavity in the mold for title
    @author = author  # Cavity in the mold for author
    @pages = pages    # Cavity in the mold for page count
  end
  
  def read
    puts "Reading #{@title} by #{@author}..."
  end
end
```

Here, the `Book` class contains a mold with three cavities (`@title`, `@author`, and `@pages`) that get filled with different values when objects are created. Every book object will have these same three elements, but with potentially different values in each.

The mold metaphor is superior to alternatives like "blueprint" because:

1. Molds are directly involved in creating objects (not just referenced)
2. Molds create identical structures with different contents
3. Molds have cavities that align perfectly with instance variables
4. Molds are used repeatedly to create multiple objects

### Objects: Things with Elements and Actions

In Ruby, objects are things that:

1. Have **elements** (what they're made of)
2. Perform **actions** (what they can do)

Elements correspond to instance variables in traditional terminology, while actions correspond to methods. But by using more intuitive language, we create a direct bridge between programming concepts and how people naturally think about things in the world.

```ruby
# Creating a book object
novel = Book.new("The Great Gatsby", "F. Scott Fitzgerald", 180)

# The book has elements (title, author, pages)
# The book can perform actions (read)
novel.read  # Output: Reading The Great Gatsby by F. Scott Fitzgerald...
```

This model eliminates the need to think about "types" or "classes" when working with objects. Just as in real life we don't constantly remind ourselves that a chair is an instance of the Chair class, in Ruby we can simply work with objects directly, understanding them by what they contain and what they can do.

## Part III: Elements and Actions

### Elements: What Objects Are Made Of

Elements are the pieces of data that make up an object. In traditional Ruby terminology, these are "instance variables," but this technical term obscures their basic purpose: they define what an object is.

```ruby
class Person
  def initialize(name, age)
    @name = name    # Element: the person's name
    @age = age      # Element: the person's age
  end
end
```

Elements have three key characteristics:

1. **They belong to specific objects**: Each object has its own set of elements
2. **They hold values**: Elements contain information that defines the object
3. **They persist**: Elements remain with the object throughout its existence

The term "element" provides an intuitive understanding of these data components without requiring knowledge of object-oriented programming theory.

### Actions: What Objects Can Do

Actions are the behaviors that objects can perform. Traditionally called "methods," actions define the capabilities of an object.

```ruby
class Person
  def speak(message)
    puts "#{@name} says: #{message}"
  end
  
  def celebrate_birthday
    @age += 1
    puts "#{@name} is now #{@age} years old!"
  end
end
```

Actions have several important characteristics:

1. **They're defined by the machine**: All objects from the same machine have the same actions
2. **They can use elements**: Actions can read and modify an object's elements
3. **They can take inputs**: Actions can accept additional information when performed
4. **They can produce outputs**: Actions can return results or create effects

By thinking in terms of "actions" rather than "methods," we align our mental model with natural language and intuitive understanding.

### The Being and Doing Model

This framework of elements and actions creates what we can call the "Being and Doing" model:

- **Being**: What an object is (its elements)
- **Doing**: What an object can do (its actions)

```ruby
# Define what a Bird is and what it can do
class Bird
  # Being
  def initialize(species, color)
    @species = species  # A bird is of some species
    @color = color      # A bird is some color
    @flying = false     # A bird is initially not flying
  end
  
  # Doing
  def fly
    @flying = true
    puts "The #{@color} #{@species} is soaring through the air!"
  end
  
  def land
    @flying = false
    puts "The #{@color} #{@species} has landed."
  end
end

# Create a bird object
robin = Bird.new("robin", "red-breasted")

# Ask the bird to do things
robin.fly  # Output: The red-breasted robin is soaring through the air!
robin.land  # Output: The red-breasted robin has landed.
```

This model connects programming directly to how we naturally think about things in the world, making Ruby more intuitive and accessible.

## Part IV: Interactions Between Objects

### Telling Objects to Perform Actions

In traditional Ruby terminology, we talk about "calling methods on objects" or "sending messages to objects." Both phrases introduce unnecessary complexity. In our refined model, the Ruby interpreter simply tell objects to perform actions:

```ruby
car.drive(50)  # Tell the car to drive at 50 mph
```

This reflects what's actually happening without introducing metaphors about "messages" or "method calls" that don't exist in the language itself. % 一個程序，被寫好後，就被交給interpreter去執行：要求哪個object做哪個action，是interpreter在按照程序指揮。這裡是有上帝視角的，不是各個object之間互相 send message。這都是胡扯。%

### What Actually Happens

From a technical perspective, when we write `car.drive(50)`:

1. The interpreter identifies the object referenced by `car`
2. The interpreter looks for the `drive` action in the object's class
3. The interpreter sets up an execution context for the action
4. The interpreter executes the action with the provided argument (50)

The interpreter handles all the details of finding and executing the appropriate code. We don't need to think about "message passing" or "method dispatch" — we can simply understand that the interpreter is telling the `car` to do `drive`.

### Objects Interacting with Other Objects

The real power of Ruby emerges when objects interact with each other:

```ruby
class Driver
  def initialize(name)
    @name = name
  end
  
  def operate_vehicle(vehicle, speed)
    puts "#{@name} is operating the vehicle..."
    vehicle.drive(speed)
  end
end

driver = Driver.new("Alex")
car = Car.new("sedan")
driver.operate_vehicle(car, 65)
```

In this example:
1. We tell the driver to operate the vehicle
2. The driver tells the car to drive at a specific speed

This ==chain of actions== allows us to build complex behaviors from simple components, creating a rich model of interacting objects that can represent real-world systems.

## Part V: Organizing Code with Toolboxes

### Toolboxes for Shared Functionality

While machines (classes) create objects of specific elements and actions, Ruby also provides a way to create collections of reusable actions: modules, which we'll call **toolboxes**.

A toolbox in Ruby is a collection of:
- Actions (methods) that can be shared across different machines
- Constants (reference materials) for standardized values
- Organizational structures (namespaces) for related code

```ruby
# A Module (Toolbox of flight-related capabilities)
module Flyable
  # A reference material (constant)
  MAX_ALTITUDE = 30000
  
  # Tools (actions)
  def fly
    puts "#{self.class} is flying up to #{MAX_ALTITUDE} feet!"
  end
  
  def land
    puts "#{self.class} is landing!"
  end
  
  # A standalone tool (module method)
  def self.calculate_flight_time(distance, speed)
    distance / speed
  end
end

# Machines (Classes) using the toolbox
class Airplane
  include Flyable  # This machine uses the flying toolbox
  
  def initialize(model)
    @model = model
  end
end

class Bird
  include Flyable  # A completely different machine also uses the same toolbox
  
  def initialize(species)
    @species = species
  end
end
```

### How Toolboxes Differ from Machines

Toolboxes (modules) differ from machines (classes) in several important ways:

1. **Toolboxes don't create objects**: You can't create a `Flyable.new`
2. **Toolboxes can be shared**: The same toolbox can be used by many different machines
3. **Toolboxes can be used directly**: Some tools can be used without objects (`Flyable.calculate_flight_time`)
4. **Toolboxes organize related tools**: They group related functionality for better organization

This toolbox metaphor helps clarify what modules are for: sharing and organizing reusable actions across different types of objects.

## Part VI: Dynamic Creation and Modification

### Ruby's Flexible Nature

One of Ruby's most powerful features is its ability to create and modify code dynamically at runtime. This means machines, toolboxes, and objects can be created and changed while the program is running.

#### Creating Machines Dynamically

You can create machines (classes) on the fly:

```ruby
def build_entity_class(type, elements, actions)
  Class.new do
    # Set up the elements
    elements.each do |element_name|
      attr_accessor element_name
    end
    
    # Initialize with the elements
    define_method :initialize do |values = {}|
      @type = type
      elements.each do |element_name|
        instance_variable_set("@#{element_name}", values[element_name])
      end
    end
    
    # Add the actions
    actions.each do |action_name, implementation|
      define_method action_name, &implementation
    end
  end
end

# Use the method to create a new machine
hero_elements = [:name, :strength, :health]
hero_actions = {
  attack: ->(target) { puts "#{@name} attacks #{target} with #{@strength} power!" },
  heal: -> { @health += 10; puts "#{@name} healed to #{@health} health!" }
}

HeroClass = build_entity_class("Hero", hero_elements, hero_actions)

# Create an object with the new machine
hero = HeroClass.new(name: "Aragorn", strength: 95, health: 100)
hero.attack("Orc")  # Output: Aragorn attacks Orc with 95 power!
```

#### Creating Toolboxes Dynamically

Similarly, you can create toolboxes (modules) at runtime:

```ruby
def create_helper_module(prefix)
  Module.new do
    define_method :format_text do |text|
      "#{prefix}: #{text}"
    end
    
    def self.utility_method(value)
      value.to_s.upcase
    end
  end
end

# Create a dynamic toolbox
TextHelper = create_helper_module("MESSAGE")

# Use the toolbox in a machine
class MessageFormatter
  include TextHelper
end

formatter = MessageFormatter.new
puts formatter.format_text("Hello")  # Output: MESSAGE: Hello
puts TextHelper.utility_method("test")  # Output: TEST
```

This dynamic nature allows Ruby programs to adapt and evolve at runtime, creating new capabilities based on changing conditions or requirements.

## Part VII: The Ruby Ecosystem

### Standard Library Toolboxes

Ruby comes with a rich set of standard toolboxes (modules) that provide common functionality:

- `Enumerable`: Actions for working with collections
- `Comparable`: Actions for comparing objects
- `FileUtils`: Actions for working with files
- `JSON`: Actions for working with JSON data

These standard toolboxes follow the same principles we've discussed, providing reusable actions that can be shared across different machines.

### Ruby's Core Machines

Ruby's core includes many built-in machines (classes) for common types of objects:

- `String`: For text
- `Array`: For collections of objects
- `Hash`: For key-value mappings
- `Integer` and `Float`: For numbers
- `Time`: For dates and times

Each of these machines creates objects with specific elements and actions appropriate to their purpose.

### External Gems

The Ruby ecosystem extends beyond the core language through gems—packages of code that provide additional machines and toolboxes. Popular gems include:

- Rails: For web application development
- RSpec: For testing
- Nokogiri: For XML/HTML processing
- Devise: For authentication

These gems follow the same mental model we've developed, creating new machines and toolboxes that work seamlessly with Ruby's core functionality.

## Part VIII: Practical Application

### Building a Complete System

Let's apply our mental model to build a simple library system:

```ruby
# A toolbox for items that can be borrowed
module Borrowable
  def borrow(person)
    if @borrowed_by.nil?
      @borrowed_by = person
      @borrowed_at = Time.now
      true
    else
      false
    end
  end
  
  def return
    if @borrowed_by
      @borrowed_by = nil
      @borrowed_at = nil
      true
    else
      false
    end
  end
  
  def borrowed?
    !@borrowed_by.nil?
  end
  
  def borrowed_by
    @borrowed_by
  end
  
  def borrowed_at
    @borrowed_at
  end
end

# A machine for creating books
class Book
  include Borrowable
  
  # Elements that make up a book
  def initialize(title, author, pages)
    @title = title
    @author = author
    @pages = pages
    @borrowed_by = nil
    @borrowed_at = nil
  end
  
  # Actions a book can perform
  def information
    "#{@title} by #{@author}, #{@pages} pages"
  end
end

# A machine for creating people
class Person
  # Elements that make up a person
  def initialize(name)
    @name = name
    @borrowed_items = []
  end
  
  # Actions a person can perform
  def borrow_item(item)
    if item.borrow(self)
      @borrowed_items << item
      puts "#{@name} borrowed #{item.information}"
      true
    else
      puts "Sorry, that item is already borrowed"
      false
    end
  end
  
  def return_item(item)
    if item.return
      @borrowed_items.delete(item)
      puts "#{@name} returned #{item.information}"
      true
    else
      puts "This item wasn't borrowed by #{@name}"
      false
    end
  end
  
  def borrowed_items
    @borrowed_items
  end
  
  def to_s
    @name
  end
end

# A machine for creating libraries
class Library
  # Elements that make up a library
  def initialize(name)
    @name = name
    @inventory = []
  end
  
  # Actions a library can perform
  def add_item(item)
    @inventory << item
    puts "Added #{item.information} to #{@name}"
  end
  
  def available_items
    @inventory.reject(&:borrowed?)
  end
  
  def borrowed_items
    @inventory.select(&:borrowed?)
  end
  
  def inventory_report
    puts "#{@name} Library Inventory Report"
    puts "Total items: #{@inventory.size}"
    puts "Available items: #{available_items.size}"
    puts "Borrowed items: #{borrowed_items.size}"
    
    unless borrowed_items.empty?
      puts "\nCurrently borrowed items:"
      borrowed_items.each do |item|
        puts "- #{item.information} (borrowed by #{item.borrowed_by})"
      end
    end
  end
end

# Using our system
library = Library.new("City")
book1 = Book.new("The Ruby Way", "Hal Fulton", 888)
book2 = Book.new("Eloquent Ruby", "Russ Olsen", 442)

library.add_item(book1)
library.add_item(book2)

alice = Person.new("Alice")
bob = Person.new("Bob")

alice.borrow_item(book1)
bob.borrow_item(book2)

library.inventory_report

bob.return_item(book2)
library.inventory_report
```

This example demonstrates how our mental model applies to real-world problems, creating a system of interacting objects that perform actions and manage their elements.

## Part IX: Learning Implications

### Reducing Cognitive Load

One of the most important insights from our exploration is that each additional concept in a programming language produces an exponential increase in cognitive load. By reducing Ruby to its essential components—machines, objects, elements, actions, toolboxes, and variables—we dramatically decrease the mental overhead required to learn and use the language.

### A Progressive Learning Path

This simplified mental model allows for a progressive learning path:

1. **Start with objects**: Understand that objects have elements and actions
2. **Move to machines**: Learn how machines create objects with consistent structures
3. **Add toolboxes**: Explore how toolboxes share actions across different machines
4. **Explore interaction**: Develop systems of interacting objects

At each step, the learner builds on familiar concepts without having to assimilate an entirely new conceptual framework.

### Teaching Recommendations

Based on our refined mental model, we recommend teaching Ruby by:

1. **Using intuitive language**: Stick to terms like objects, actions, elements, machines, and toolboxes
2. **Focusing on behavior first**: Emphasize what objects can do before how they're created
3. **Building from concrete to abstract**: Start with specific objects before introducing machines
4. **Minimizing jargon**: Avoid unnecessary terminology that isn't reflected in the language itself
5. **Connecting to real-world metaphors**: Use analogies that match how people naturally think

This approach makes Ruby more accessible to beginners while also providing a clearer conceptual framework for experienced developers.

## Conclusion: The Essence of Ruby

Ruby's creator, Yukihiro Matsumoto, designed the language with human developers in mind, emphasizing simplicity, consistency, and joy of use. By stripping away unnecessary conceptual overhead and aligning our mental model with how Ruby actually works, we can better appreciate and utilize the elegant simplicity that makes Ruby special.

The mental model presented in this essay—understanding Ruby in terms of machines, molds, objects with elements and actions, and toolboxes for shared functionality—provides a clearer, more intuitive framework for learning and using Ruby. By focusing on what things are and what they can do, rather than abstract computer science concepts, we make the language more accessible while remaining true to its technical implementation.

Programming languages don't need to feel complex and mystifying. With the right mental model—one that aligns with how the language actually works rather than theoretical constructs imported from elsewhere—Ruby becomes a natural extension of human thought, a tool for expressing ideas clearly and effectively.

As we continue to teach and learn programming, let us remember that the goal is not to master jargon or theoretical frameworks, but to communicate effectively with computers in service of human needs. Ruby, when properly understood, excels at this goal. By simplifying our mental model, we can better appreciate and leverage the language's expressive power and elegant design.

Ruby is, at its heart, a language designed for human understanding. By teaching and discussing it in human terms, we honor that design and make programming more accessible to all.