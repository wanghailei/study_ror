# Concerns

Ruby on Rails, being an opinionated framework, treats modules in a special way. Instead of using pure Ruby modules, Rails advises using concerns.

## What is a Rails concern? 

==A concern is an enhanced Ruby module==; compared to plain modules, Rails concerns come with the additional functionalities:

- Concerns provide a DSL to simplify injecting standard Rails operations (defining callbacks, associations, and so on).
- Concerns support dependency resolution for included modules.

## `ActiveSupport::Concern`

## Concerns in ActiveRecord

### When to extract code as concerns

Extracting common atomic behaviors into concerns can help reduce the corresponding models’ conceptual overhead.

Having concepts in isolated locations makes it harder to introduce dependencies between them accidentally.

Concerns are usually associated with common logic extraction from multiple models.

There is a good rule of thumb for model concerns – if removing a concern from a model makes most of the tests involving this model fail, then this concern is an essential part of the logic. In other words, if a concern can’t be detached, it’s not a concern but just a piece of extracted code.

## The Dual Role of `included` in ActiveSupport::Concern

In standard Ruby, `self.included(base)` is a callback invoked whenever a module is included in another module or class. However, `ActiveSupport::Concern` repurposes this method to serve two distinct functions: defining a callback block and executing the actual inclusion logic.

When you use `included do ... end` inside a Concern, you are invoking the `if base.nil?` branch. Here, `included` acts as a macro or DSL method. It captures the provided block and assigns it to the instance variable `@_included_block`. This block effectively becomes a "deferred recipe" of class macros (like `has_many`, `validates`, or `scope`) that will be evaluated later within the context of the host class.

The implementation includes a specific guard clause regarding `@_included_block`. If you attempt to define `included`multiple times within the same module, it raises `MultipleIncludedBlocks`. The check `if @_included_block.source_location != block.source_location` is a clever idempotency check. It allows the *exact same*block definition to be re-evaluated without error (which can happen during code reloading), but it strictly forbids defining two different `included` blocks, enforcing a singular initialization context for the Concern.

The `else` branch handles the standard Ruby lifecycle. When the module is eventually included into a class (e.g., `class User; include MyConcern; end`), Ruby calls `included(base)` passing the host class as `base`. Since `base` is not nil, it triggers `super`. The `ActiveSupport::Concern` implementation of `append_features` (which usually runs before this) ensures that the stored `@_included_block` is evaluated against the `base` class.

In essence, this method allows the module to masquerade as a configuration builder when called without arguments, while still maintaining its ability to function as a standard Ruby hook when the inclusion actually occurs.

Would you like to examine how `append_features` utilizes this stored block to manage dependencies between Concerns?
