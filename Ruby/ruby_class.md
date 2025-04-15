# Class

Classes are, at heart, a way to organise objects and methods.\
==% 與其說 organise，毋寧說 define 或 design，可以說是「藍圖」。 20231117%==

Defining a class lets you group behaviours (methods) into convenient bundles, so that you can quickly create many objects that behave essentially the same way.

Everything you handle in Ruby is either an object or a construct that evaluates to an object, and every object is an instance of some class.

When you use the dot notation on a class, you send a message to the class. Classes can respond to messages.



Ruby is about objects.

Objects are instances of classes.

Classes are objects.

`Object` is a built-in Ruby class.

Classes are named with constants.

==% 定義一個 Class，就像「開模」來製造一個銅盤一樣。Class就是模具。20230624 %==



***

### Learned from Xavier Noria&#x20;

In Ruby, a class or a module defined with the `class` or `module` keywords, are indeed ==stored as constants==. This is a unique aspect of Ruby's object model. For example, when you define `class MyClass; end`, ==`MyClass` is a constant that references the class object.==

==In Ruby, constants are scoped within classes and modules, and they are not global.== This means you can have different constants with the same name in different scopes without conflict. For instance, you could have `class A; MY_CONST = 1; end` and `class B; MY_CONST = 2; end` without any conflict between `A::MY_CONST` and `B::MY_CONST`.

In Ruby, top-level constants (those defined outside of any class or module) are treated as if they belong to the `Object` class. This is because `Object` is the default object context of Ruby at the top level. This means that any constant defined at the top-level is accessible from anywhere in the program, as all classes in Ruby are descendants of `Object`.

`Module#autoload` allows you to load constants on demand.

## Singleton Class

In Ruby, ==each object has its own singleton class== (also known as a metaclass or eigenclass). ==The singleton class is a subclass of the object's class.== This singleton class is automatically created by Ruby the first time you define a singleton method on the object.&#x20;

The singleton class is a core aspect of Ruby's object-oriented design, which emphasizes flexibility and expressiveness. Here are some key reasons why singleton classes are used:

1. **Implementing Class Methods**: ==In Ruby, class methods are actually instance methods of the class's singleton class.== This is because classes themselves are also objects in Ruby, so each class has its own singleton class where its class methods are stored.
2. **Individual Object Customisation**:  ==Singleton classes allow you to add methods to an individual object rather than to an entire class of objects.== Without singleton class, the only way would be to modify its class. However, modifying a class affects all instances of that class, which can lead to unexpected side effects. 
   Especially, when you want to add functionality to instances of core classes (like String, Array, etc.) without affecting all instances, singleton classes are the way to go. This is often used in Ruby on Rails and other frameworks to extend the capabilities of standard objects on a per-instance basis.
   &#x20;==% Why don't use mixin? 20231223 %==
3. **Supporting Metaprogramming**: Ruby's metaprogramming capabilities are largely facilitated by singleton classes. They allow dynamic addition of methods at runtime, making Ruby a very flexible and powerful language for metaprogramming.
4. **Method Look-Up Consistency**: Ruby maintains a consistent method look-up path. ==When you call a method on an object, Ruby first looks in the object's singleton class before looking in the object's actual class and its ancestors.==&#x20;







## `subclass = Class.new(self)`

`Class.new` is a method in Ruby that creates a new class.

* when you call `Class.new` without any arguments, it creates an anonymous class with `Object` as its superclass.
* when you provide an argument to `Class.new`, it creates a new class with the provided argument as its superclass.
* when you pass `self` as an argument to `Class.new`, it creates a new class that is a subclass of the current class (the one where this code is being executed).

This subclass inherits all the methods and properties of the original class but is a distinct class that can be modified independently.

This is a way to dynamically create a new subclass of the current class. This technique is useful in scenarios where you need to create classes dynamically based on runtime conditions or for advanced metaprogramming patterns.



## Why there is :: in the class name?

```ruby
class Room::MessagePusher
end
```

In Ruby, the `::` notation is used for namespacing classes and modules.

Here, `Room::MessagePusher` indicates that `MessagePusher` is a class that belongs to the `Room` namespace. This helps organize related classes together and prevents naming conflicts.

This namespacing pattern is common in Rails applications to group related functionality. For example, this class specifically handles pushing messages for a Room, so it's logically placed within the Room namespace.





## Instance Methods in Class Context

You won't call an instance method inside its class body. Within the class body (outside of any method definition), you cannot directly call instance methods because:

1. Instance methods require an instance (object) to operate on.
2. In the class body, no instance exists yet.

You **cannot** have an instance of a class available in the class body during definition, as the class itself is still being defined.

