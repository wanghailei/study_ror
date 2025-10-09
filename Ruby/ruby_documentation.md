# Documentation

## The difference between `#method_name` and `.method_name` syntax

This notation difference (# vs .) is a common Ruby **documentation convention** used to distinguish between instance methods and class methods.

The key difference is in the notation:

1. `Some::Rendering#render_to_string` (with the `#` symbol):
	- This refers to an **instance method** named `render_to_string` in the `Some::Rendering` class/module
	- It's a method called on instances of the class (objects)
	- You would call it like: `some_rendering_object.render_to_string`
	- It operates in the context of a specific instance with access to instance variables
2. `Another::Rendering.render_to_string` (with the `.` symbol):
	- This refers to a **class method** named `render_to_string` in the `Another::Rendering` class/module
	- It's a method called directly on the class itself, not on instances
	- You would call it like: `Another::Rendering.render_to_string`
	- It operates in the class context without access to instance-specific data (unless provided as arguments)



- ==`#method_name` indicates an instance method.==
- ==`.method_name` indicates a class method.==

