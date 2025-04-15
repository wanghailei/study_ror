# Naming Conventions in Ruby and Rails

You raise a good question about naming conventions. The general principles vary somewhat between languages and frameworks, but here are the common conventions in Ruby and Rails:

## Method Naming

In Ruby, there are several patterns for method names:

1. **Verbs/Actions** (most common): `calculate_total`, `save_record`, `find_by_name`
	1. **Exclamation methods** (indicating modification or danger): `save!`, `destroy!`

2. **Noun/attribute accessors**: `name`, `title`, `age` (especially for attribute readers/writers)
3. **Question methods** (returning boolean): `paid?`, `admin?`, `valid?`

Rails has methods that appear as nouns because they're often:
- Attribute accessors
- Named associations: `user.posts` (looks like a noun but conceptually retrieving records)
- Dynamic finders: `find_by_email`

## Module Naming

For modules, the conventions are more flexible:

1. **Nouns for namespacing**: `ActiveRecord`, `ActionController`
2. **Adjectives for mixins**: `Enumerable`, `Comparable`, `Validatable`
3. **Plural nouns for collections of related functionality**: `Callbacks`, `Validations`

### Rails Examples

Rails uses many noun-named modules for organization:
- `ActiveSupport`, `ActiveRecord`, `ActionView`
- Mixins like `Enumerable`, `Configurable`, `Serializable`

## Best Practices

1. Methods that perform actions should be verbs
2. Methods that return a property can be nouns
3. Module names should represent their purpose - adjectives for behaviors, nouns for namespaces

There's no strict rule that modules must be adjectives - naming should prioritize clarity and purpose over rigid grammatical rules.