# Method Override

To override a method defined in a Ruby module within a Rails application, you have several options depending on your specific use case. Here are some common techniques: 

### 1. Override in the Including Class

If a class includes a module, you can override the module’s method by defining a method with the same name in the class:

```ruby
module A
  def greet
    puts "Hello from A"
  end
end

class B
  include A

  def greet
    puts "Hello from B"
  end
end

B.new.greet
# Output: Hello from B
```

In this example, the greet method in class B overrides the one from module A. 

### 2. Override Using `Module#prepend`

Ruby’s `prepend` allows you to place a module before the class or module in the method lookup chain, effectively overriding existing methods: 

```ruby
module A
    def greet
    	puts "Hello from A"
    end
end

module B
    def greet
        puts "Hello from B"
        super
    end
end

class C
    include A
    prepend B
end

C.new.greet
# Output:
# Hello from B
# Hello from A
```

Here, `B#greet` is called first due to `prepend`, and it calls super to invoke `A#greet`. 

==With `Module#prepend` you only need to implement the methods you want to change. Anything you don’t redefine will continue to come from the original class.== With prepend, **only the methods you redefine are touched** — everything else stays as it is. 

`prepend` puts your module **before** the class in the ancestor chain, so `super` calls the original method.

### 3. Monkey Patching the Module

You can directly modify the module to override its methods. This is known as **monkey patching**: 

```ruby
module A
    def greet
    	puts "Hello from A"
    end
end

module A
    def greet
    	puts "Hello from patched A"
    end
end

class D
	include A
end

D.new.greet
# Output: Hello from patched A
```

While monkey patching can be powerful, it should be used cautiously as it affects all uses of the module and can lead to unexpected behavior. 

#### In Rails, put monkey patches in config/initializers.

All Ruby files in `config/initializers/` are loaded when Rails boots, and no extra manual includes needed. It works automatically everywhere (views, helpers, emails). Clean. It’s intended for monkey patches, global config**, **third-party tweaks. Perfect place for small overrides.

### 4. Override via Module Inclusion Order

When including multiple modules, ==the last included module’s methods take precedence==: 

```ruby
module A
  def greet
    puts "Hello from A"
  end
end

module B
  def greet
    puts "Hello from B"
  end
end

class E
  include A
  include B
end

E.new.greet
# Output: Hello from B
```

In this case, B#greet overrides A#greet because B was included after A. 

### 5. Using `alias_method` for Method Wrapping

To extend or modify existing methods while preserving the original, you can use alias_method: 

```ruby
module A
    def greet
    	puts "Hello from A"
    end
end

class F
    include A
    alias_method :original_greet, :greet

    def greet
        puts "Before greet"
        original_greet
        puts "After greet"
    end
end

F.new.greet
# Output:
# Before greet
# Hello from A
# After greet
```

This technique allows you to add behavior before and after the original method call. 

### 6. Overriding Methods from External Gems

If you need to override a method from a module provided by an external gem, you can prepend your module to the gem’s module: 

```ruby
module MyOverride
    def target_method
    	puts "Overridden method"
    	super
    end
end

GemModule.prepend(MyOverride)
```

This approach is useful for customizing behavior without modifying the gem’s source code.

Each of these methods has its use cases and implications. Choose the one that best fits your scenario, keeping in mind maintainability and potential side effects.