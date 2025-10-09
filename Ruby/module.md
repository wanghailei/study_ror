# Module

Modules are a way of grouping together methods, classes, and constants. Modules give you two major benefits:&#x20;

* Modules provide a namespace to prevent name clashes.&#x20;
* Modules can be included in other classes, a facility known as a mixin.

==Like a class, a module name is also global constants.==&#x20;

Module methods are defined like class methods, using the `def self.method_name` syntax. As with class methods, a module method is called by preceding its name with the module’s name and a period, like `ActiveSupport.run_load_hooks(:active_record, Base)`.

Module constants are referenced using the module name followed by two colons, which is called ==the scope resolution operator==, like Active

A module can’t have instances, because a module isn’t a class. &#x20;

You can `include` a module within a class definition. When this happens, all the module’s instance methods are suddenly available as instance methods in the class as well. This is called mixin.

## `extend` and `include`

These are two different ways to add module functionality to classes, but they target different levels. When `extend` a module in a class, the module's methods are added as class methods of the target class. When `include` a module into a class, the module's methods are added as instance methods of the class.

### Include for instance methods

`include` adds the module's methods as **instance methods** to the class, and the methods become available to instances (objects) of the class. Used with `include Module` in a class definition.

```ruby
class MyClass
	include MyModule
end

obj = MyClass.new
obj.module_method  # Works because module methods become instance methods
```

### Extend for class methods

`extend` adds the module's methods as **class methods** to the class, and the methods become available directly on the class itself. Used with `extend Module` in a class definition.

```ruby
class MyClass
	extend MyModule
end

MyClass.module_method  # Works because module methods become class methods
obj = MyClass.new
obj.module_method # would fail
```

#### Why "Extend" is Necessary

Extending modules is necessary when you need functionality at the class level rather than the instance level. This way has organizational Benefits:

- **Namespace organization**: Keeps related class-level functionality grouped together
- **Code reusability**: Allows the same class methods to be shared across multiple classes
- **Separation of concerns**: Helps separate different types of functionality into dedicated modules

#### 2. Technical Scenarios Where Extend Shines

- **Factory methods**: When you need methods that create new instances with special logic.
- **Utility functions**: Methods that operate on the class itself or provide class-level utilities.
- **Configuration systems**: Methods that set up class-level behavior

#### 3. Module Polymorphism

- Class methods via `extend` allow different classes to implement the same class-level interface
- Makes it possible to swap implementations while maintaining the same API

## `Module#autoload`

==`Module#autoload` is a method in Ruby that allows you to defer the loading of a module or class until it is first used.== This can be beneficial for reducing the startup time of a Ruby program. Ruby 3.0 introduced improvements to make constant autoloading thread-safe.

==Zeitwerk handles code loading in a more sophisticated way, making the explicit use of `autoload` less common in modern Rails applications.==

### How `Module#autoload` works

You specify a constant name and the name of a file to be loaded. The constant serves as a placeholder for the module or class you want to autoload. When the constant is first accessed in the code, Ruby automatically loads the specified file.

The method is used like this: `autoload :ModuleName, 'path/to/file'`. Here, `:ModuleName` is the name of the constant that will be used as a placeholder for the module or class, and `'path/to/file'` is the file path (relative or absolute) where the actual definition of the module or class is located.

```ruby
module MyModule
	autoload :MySubModule, 'path/to/my_submodule'
end
# At this point, MySubmodule is not yet loaded.
# Later in the program...
# The following line will trigger the loading of 'path/to/my_submodule'.
MyModule::MySubModule.do_something 
```



