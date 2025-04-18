# Naming Conventions

The general principles vary somewhat between languages and frameworks, but here are the common conventions in Ruby and Rails.

## Method Naming

In Ruby, there are several patterns for method names:

### Methods that perform actions should be verbs.

Most common): `calculate_total`, `save_record`, `find_by_name`

#### Exclamation methods

Indicating modification or danger: `save!`, `destroy!`

#### Question methods

Returning boolean: `paid?`, `admin?`, `valid?`

### Methods that return a property can be nouns.

Rails has methods that appear as nouns because they're often **attribute accessors** like `name`, `title`, `age` or **named associations** like `user.posts`, which looks like a noun but conceptually retrieving records.

## Module Naming

Module names should represent their purpose - adjectives for behaviors, nouns for namespaces. There's no strict rule that modules must be adjectives - naming should prioritize clarity and purpose over rigid grammatical rules.

### Nouns for namespacing

`ActiveRecord`, `ActionController`

### ==Adjectives for mixins==

`Enumerable`, `Comparable`, `Validatable`, `Serializable`

### Plural nouns for collections of related functionality

`Callbacks`, `Validations`

