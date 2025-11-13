# Concerns

Ruby on Rails, being an opinionated framework, treats modules in a special way. Instead of using pure Ruby modules, Rails advises using concerns.

## What is a Rails concern? 

A concern is an enhanced Ruby module; compared to plain modules, Rails concerns come with the additional functionalities:

- Concerns provide a DSL to simplify injecting standard Rails operations (defining callbacks, associations, and so on).
- Concerns support dependency resolution for included modules.

## `ActiveSupport::Concern`

## Concerns in ActiveRecord



### When to extract code as concerns

Extracting common atomic behaviors into concerns can help reduce the corresponding models’ conceptual overhead.

Having concepts in isolated locations makes it harder to introduce dependencies between them accidentally.

Concerns are usually associated with common logic extraction from multiple models.

There is a good rule of thumb for model concerns – if removing a concern from a model makes most of the tests involving this model fail, then this concern is an essential part of the logic. In other words, if a concern can’t be detached, it’s not a concern but just a piece of extracted code.

