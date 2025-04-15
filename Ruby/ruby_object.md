# Object



## The class `Object`

[`Object`](https://ruby-doc.org/3.4.1/Object.html) is the default root of all Ruby objects. `Object` inherits from [`BasicObject`](https://ruby-doc.org/3.4.1/BasicObject.html) which allows creating alternate object hierarchies. Methods on `Object` are available to all classes unless explicitly overridden.

`Object` mixes in the [`Kernel`](https://ruby-doc.org/3.4.1/Kernel.html) module, making the built-in kernel functions globally accessible. The instance methods of `Object` are defined by the `Kernel` module.

## An object's three components

“Internally, a Ruby object has three components: a set of flags, some instance variables, and an associated class.” 

### When you create a Ruby object

```ruby
s = "hello"
```

Internally, Ruby creates a memory object for `s`. This object stores:

- **Flags** (e.g., that it’s a String, whether it’s frozen): Metadata about the object’s state
- **Instance variables** (if you attach any, like `@name`): The object’s unique data
- **A reference to its class** (`String`)The blueprint that defines the object’s behavior and type, including where its methods live. Methods, being part of the behavior rather than the object’s individual state, are encapsulated within the class system.

### What are flags in Ruby?

In the context of a Ruby object’s internal structure, “flags” refer to a set of bits used by the Ruby interpreter to store metadata about the object. 

Flags are tiny bits of information stored inside every Ruby object to describe special properties, such as:

- **What kind of object it is** (e.g., an Array, a String, a custom class)
- **Whether the object is frozen** (`.freeze`d and immutable)
- **Whether the object is tainted or trusted** (older Ruby security models)
- **Whether the object is "singleton"** (meaning it has a *singleton class* attached)
- **Garbage collection information** (like whether it’s marked for GC)

Each flag is just a **bit** (0 or 1) in a small block of memory. By packing a lot of these bits together, Ruby can quickly check an object's state without using a lot of memory. These flags are part of the object’s header and help the interpreter manage the object’s state and behavior efficiently.

1. **Object State Information**: Flags can indicate special states of an object, such as whether it is frozen (immutable), tainted (marked as potentially unsafe), or untrusted.
2. **Type Information**: Some flags help identify the object’s type or how it’s stored internally. 
3. **Memory Management**: Flags assist the garbage collector by indicating whether an object is “marked” during the mark-and-sweep process or if it has specific memory-related properties, like being a singleton or having a shared structure.
4. **Visibility and Behavior**: Flags can denote things like the visibility of methods (e.g., private or protected) or whether an object has been modified in a way that affects its class or instance variables.

From a programmer’s perspective, you don’t interact with flags directly.

### The Role of the Associated Class

The “associated class” component is the key link. It’s not just the object’s class (e.g., `String` for a string object) but can also include modifications like singleton classes (for object-specific methods). For example:

- If you define a method on a specific object using `def obj.some_method`, Ruby creates a singleton class for that object and stores the method there. The object’s “associated class” then points to this singleton class, which still adheres to the class-based method lookup system.
- This means the “associated class” component indirectly provides access to methods without needing to list them as a separate part of the object’s structure.

## Why an object does not have methods?

Methods are not listed as a direct component of the object itself. This is because of how Ruby implements its object model: ==methods are not stored within the object but are instead defined in the object’s class (or its ancestors), and the object merely references that class.== ==A Ruby object doesn't carry methods around with it. It just knows where to find them==. 

### Class-Centric Method Storage

==In Ruby, methods are defined at the class level, not the object level.== When you create an object, it doesn’t carry its own copy of methods. Instead, the object’s “associated class” (the third component) serves as a pointer to the class where the methods are stored. This class contains a method table — a mapping of method names to their implementations. When you call a method on an object, Ruby looks up the method in the object’s class (and its inheritance chain if needed) via a process called *method dispatch*.

Storing methods directly in each object would be inefficient. Imagine if every instance of a class had to duplicate the method definitions—memory usage would skyrocket, especially for classes with many instances or complex methods. By keeping methods in the class, Ruby ensures that all objects of the same class share the same method definitions, reducing redundancy.

### Where do methods live?

- If you call `s.upcase`, Ruby says:
  1. "What's the class of `s`?" → `String`
  2. "Does `String` have an `upcase` method?" → yes
  3. "Call it!"

Methods are **defined on classes** (or modules), and objects **share** them.
