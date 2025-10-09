# Data and Struct

## Data.define

`Data.define` is a built-in way to define simple, immutable data classes. Think of it as a modern replacement for Struct.new, but immutable, comparable, and pattern-matchable.

The whole point of `Data` in Ruby is to give you a **pure value object**:

- ==It only has attributes== (the ones you list in Data.define).
- ==You cannot add your own methods to it== (unlike `Struct`), autho it automatically has a few standard methods (#==, #hash, #to_h, #inspect, etc.).

### When to use Data.define

- When you need **lightweight value objects** (like records, events, DTOs).
- When immutability is a benefit (e.g. for safety, concurrency).
- As a **simpler alternative to Struct**, without custom methods.

Example in your BOS context:

```ruby
Order = Data.define(:id, :item, :quantity, :price)
o = Order.new(1, "Croissant", 10, 3.5)

puts "Order ##{o.id}: #{o.quantity} x #{o.item} at #{o.price} each"
# => Order #1: 10 x Croissant at 3.5 each
```

### Key Properties

#### Immutable:

==attributes can’t be reassigned.==

```ruby
p1.x = 3 # => NoMethodError (cannot modify frozen Data)
```

#### Keyword init

```ruby
User = Data.define(:id, :name)
u = User.new(id: 1, name: "Hailei")
u.name   # => "Hailei"
```

#### Pattern matching

works smoothly with Ruby’s case/pattern matching.

```ruby
case Point.new(3, 4)
in Point(x:, y:)
	puts "x=#{x}, y=#{y}"
end
# => x=3, y=4
```

#### Comparable

instances can be compared with ==, eql?, and hash).

```ruby
Point = Data.define(:x, :y)
p1 = Point.new(1, 2)
p2 = Point.new(1, 2)
p1.x       # => 1
p1.y       # => 2
p1 == p2   # => true (structural equality)
```

#### Value semantics

```ruby
Point.new(1,2).hash == Point.new(1,2).hash # => true
```

### Why Data was added to Ruby?



Ruby’s Struct is **mutable** (you can change values after creation). But modern Ruby is moving toward **immutability for safety** (esp. in concurrent systems). So Ruby 3.2 added Data.define — a **safer, immutable alternative**. 

- ==**Struct** = the old tool to make quick classes== (mutable, can have methods).
- ==**Data.define** = the new tool== (immutable, pure data).

## Struct

Struct is a Ruby class that lets you **define simple data containers** quickly. You can think of it as a **lightweight class generator**. Before Ruby 3.2, people often used Struct when they needed a class but didn’t want to write all the boilerplate.

```ruby
# Define a struct called Point with two fields
Point = Struct.new(:x, :y)
p1 = Point.new(3, 4)
p1.x   # => 3
p1.y   # => 4
p1.x = 10   # allowed (Struct is mutable)
p1.x        # => 10
```

That’s exactly the kind of thing Data.define does now — but with **immutability**.

### With Methods

Unlike `Data.define`, a `Struct` can also carry behaviour:

```ruby
Point = Struct.new(:x, :y) do
  def distance_from_origin
    Math.sqrt(x**2 + y**2)
  end
end

p1 = Point.new(3, 4)
p1.distance_from_origin  # => 5.0
```

This is why Struct is still useful if you want both **data + methods** in one go.

## Class vs Struct in Ruby

Both **Class** and **Struct** let you define objects, but they serve different purposes.

| **Aspect**      | Class                                                     | Struct                                                       |
| --------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| **Purpose**     | General-purpose blueprint for objects (data + behaviour). | Quick way to define a class that only holds attributes (plus optional methods). |
| **Definition**  | Verbose: you write attr_accessor, initialize, etc.        | Concise: attributes + init auto-generated.                   |
| **Attributes**  | You must declare accessors manually.                      | Automatically generated from the symbol list.                |
| **Methods**     | You can freely define any methods.                        | You can define methods too, but that’s secondary.            |
| **Mutability**  | By default mutable (but you can freeze).                  | Mutable too (unless you freeze it yourself).                 |
| **Inheritance** | Full support (can subclass, include modules, etc.).       | You can subclass Struct, but its design is for lightweight containers, not deep hierarchies. |
| **Use cases**   | Complex domain models, business logic, services.          | Small data carriers, quick throwaway objects, prototypes.    |

#### Using Class

- ✅ Full control
- ❌ More boilerplate

```ruby
class Order
    attr_accessor :id, :item, :qty, :price

    def initialize(id, item, qty, price)
    	@id = id
    	@item = item
    	@qty = qty
    	@price = price
    end

    def total
    	qty * price
    end
end
o = Order.new(1, "Croissant", 10, 3.5)
o.total   # => 35.0
```

#### Using Struct

- ✅ Much shorter (auto initialize, attr_accessor)
- ✅ Can still define methods
- ❌ Less explicit than a full class

```ruby
Order = Struct.new(:id, :item, :qty, :price, keyword_init: true) do
    def total
    	qty * price
    end
end
o = Order.new(id: 1, item: "Croissant", qty: 10, price: 3.5)
o.total   # => 35.0
```

#### Using Data.define

- ✅ Pure immutable data
- ❌ No custom methods

```ruby
Order = Data.define(:id, :item, :qty, :price)
o = Order.new(1, "Croissant", 10, 3.5)
o.price   # => 3.5
o.price = 4.0   # => NoMethodError (immutable)
```

### Rule of Thumb

**Use Class** → when building **domain logic** with lots of behaviour.

**Use Struct** → when you want a **data container with some methods** quickly.

**Use Data.define** → when you want **immutable data-only objects**.

👉 So in BOS:

- Your **business models** (Bakery, Recipe, Employee) → **normal classes**.
- Your **small helper objects** (Point, Color, Size) → maybe Struct.
- Your **pure records for BI dashboards** (DailyPerformance, OrderRow) → Data.define.