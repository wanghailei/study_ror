# Delegation 委託別人代為執行

`ActiveSupport::Delegation` in Rails provides a concise way ==to delegate method calls to associated objects==, enhancing code readability. This feature enables developers to forward method calls to associated objects, such as delegating a `name` method from a `Profile` model to a `User` model, allowing `user.name` instead of `user.profile.name`. 

For example:

```ruby
# db/migrate/20240501000001_create_users.rb
class CreateUsers < ActiveRecord::Migration[8.1]
    def change
        create_table :users do |t|
            t.string :email
            t.string :password_digest
            t.string :username
            t.boolean :admin, default: false

            t.timestamps
        end
    end
end

# db/migrate/20240501000002_create_profiles.rb
class CreateProfiles < ActiveRecord::Migration[8.1]
    def change
        create_table :profiles do |t|
            t.references :user, foreign_key: true

            t.string :first_name
            t.string :last_name
            t.date :date_of_birth
            t.string :bio
            t.string :location
            t.string :avatar_url
            t.string :phone_number

            t.timestamps
        end
    end
end
```

Without delegation, you'd need to write many forwarding methods:

```ruby
# app/models/user.rb
class User < ApplicationRecord
	has_one :profile, dependent: :destroy
  
	# Lots of manual forwarding methods
    def first_name
    	profile&.first_name
    end
  
    def first_name=(value)
    	profile_or_build.first_name = value
    end
  
  def last_name
    profile&.last_name
  end
  
  def last_name=(value)
    profile_or_build.last_name = value
  end
  
  def date_of_birth
    profile&.date_of_birth
  end
  
  def date_of_birth=(value)
    profile_or_build.date_of_birth = value
  end
  
  def bio
    profile&.bio
  end
  
  def bio=(value)
    profile_or_build.bio = value
  end
  
    def full_name
        if first_name.present? || last_name.present?
        	"#{first_name} #{last_name}".strip
        else
        	username
        end
    end
  
	private
  
    def profile_or_build
    	profile || build_profile
    end
end

# app/models/profile.rb
class Profile < ApplicationRecord
	belongs_to :user
end
```

With delegation, this becomes:

```ruby
# app/models/user.rb
class User < ApplicationRecord
	has_one :profile, dependent: :destroy
  
    # Clean, DRY delegation of methods to profile
	delegate :first_name, :last_name, :date_of_birth, :bio, :location, :avatar_url, :phone_number,
		to: :profile_or_build, allow_nil: true
           
	# Automatically delegates getters AND setters
	delegate :first_name=, :last_name=, :date_of_birth=, :bio=, :location=, :avatar_url=, :phone_number=,
		to: :profile_or_build
  
    def full_name
        if first_name.present? || last_name.present?
	        "#{first_name} #{last_name}".strip
        else
	        username
        end
    end
  
	private
  
    def profile_or_build
	    profile || build_profile
    end
end

# app/models/profile.rb
class Profile < ApplicationRecord
    belongs_to :user

    def age
        return nil unless date_of_birth
        ((Time.current.to_date - date_of_birth) / 365.25).to_i
    end
end

```



## OO Perspective

From an object-oriented (OO) mindset, delegation may seem confusing due to potential encapsulation concerns, as ==it allows one object to directly access another object's methods==, possibly blurring object boundaries. 

However, in the context of Rails, models often represent database tables with predefined associations (e.g., `has_one`, `belongs_to`), and delegation is a pragmatic tool for simplifying navigation. It does not violate the application's overall structure but rather leverages Rails' convention-over-configuration  approach. 

## `Module#delegate`

The delegation feature, specifically the `delegate` method, is part of the `Module` class extensions and allows developers to expose public methods of contained objects as their own. 

For instance, in a Rails application, a `User` model with a `has_one :profile` relationship can use `delegate :name, to: :profile` to enable `user.name` to call `user.profile.name`, as detailed in the Rails API documentation ([Module#delegate](https://api.rubyonrails.org/v8.0.2/classes/Module.html#method-i-delegate)).

The `delegate` method supports multiple options, including:
- `:to`: Specifies the target object, evaluated in the receiver's context (e.g., `delegate :logger, to: :Rails`).
- `:prefix`: Adds a prefix to delegated methods (e.g., `prefix: true` generates `address_street` for `street`).
- `:allow_nil`: Returns `nil` if the target is `nil`, preventing `Module::DelegationError` (e.g., `delegate :name, to: :profile, allow_nil: true`).
- `:private`: Makes delegated methods private (e.g., `delegate :date_of_birth, to: :profile, private: true`).

## DRY Principle

The primary rationale for using `ActiveSupport::delegation` is to enhance code readability and reduce redundancy. ==By delegating methods, developers avoid writing explicit boilerplate methods that simply for accessing associated objects' attributes==. 

This approach is particularly beneficial in complex applications with numerous model relationships, where nested method calls (e.g., `user.profile.name`) can become cumbersome.

## Alternatives

`ActiveSupport::delegation` is not strictly necessary, as developers can achieve similar functionality through explicit method definitions or Ruby's standard library alternatives. For instance, the `Forwardable` module, part of Ruby's standard library, allows method delegation via `def_delegators`.

```ruby
require 'forwardable'
class Computer
    extend Forwardable
    def_delegators :@memory, :read, :write
    def initialize
		@memory = Memory.new
    end
end
```

Similarly, `SimpleDelegator` wraps an object to expose all its methods, but it lacks the granularity of `Forwardable` or Rails' `delegate` method. One concern is control over the public interface: `SimpleDelegator` exposes all methods of the wrapped object, which may be undesirable if certain methods should remain hidden.

#### Comparative Analysis
To further illustrate, the following table compares ActiveSupport::delegation with alternative approaches:

| Technique             | Description                                                  | Example Usage                                                | Notes                                                        |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Rails Delegate Method | Uses `delegate` from ActiveSupport gem to forward specific methods to another object | `delegate :read, :write, to: :@memory` or with prefix: `delegate :read, :write, prefix: "memory", to: :@memory` | Creates methods like `memory_read`, `memory_write` when using prefix; Requires Rails or ActiveSupport |
| Forwardable Module    | Uses Ruby's `Forwardable` module to define delegator methods | `extend Forwardable`; `def_delegators :@memory, :read, :write` | Handles method arguments and blocks automatically; Typo noted: `def_delegator` vs correct `def_delegators` |
| SimpleDelegator Class | Wraps an object to expose all its methods, part of Ruby's `delegate` library | `class CoolArray < SimpleDelegator; end; cool = CoolArray.new([]); cool << "ruby"` | Useful for adding new methods without changing the original class; Requires `require 'delegate'` |



## Is Explicit Delegation Necessary in a Has_One Relationship?

Yes, explicit delegation is necessary in a `has_one` relationship in ActiveRecord. Despite being a close association, Rails does not automatically create delegated methods for `has_one` associations (or any associations).

### Rails Association Behavior

When you define a `has_one` relationship in Rails:

```RUBY
class User < ApplicationRecord
	has_one :profile
end
```

Rails automatically provides these methods:

- `user.profile` - accessor for the associated object
- `user.profile=` - setter for the associated object
- `user.build_profile` - builds a new associated object
- `user.create_profile` - creates a new associated object
- `user.create_profile!` - creates a new associated object with bang version
- `user.reload_profile` - reloads the associated object

### Automatic delegation vs Selective delegation

However, Rails does not automatically create methods to access the attributes of the associated object. For example, if your `Profile` model has a `first_name` attribute, Rails does not automatically create a `user.first_name` method that delegates to `user.profile.first_name`. 

There are several reasons for this design decision, but the main reason is avoiding Name Collisions - automatic delegation could create method name conflicts.

#### A Better Pattern: Selective Delegation

Most Rails applications use selective delegation:

```ruby
class User < ApplicationRecord
    has_one :profile
    delegate :first_name, :last_name, :bio, to: :profile_or_build
private
    def profile_or_build
    	profile || build_profile
    end
end
```

This approach makes it explicit which methods are delegated, handles the `nil` case by building a `profile` if needed, and keeps API clean while maintaining clear separation of responsibilities.

### Options for Delegation

When working with `has_one` associations, you have several options:

#### 1. Explicit Delegation with `delegate`

```ruby
class User < ApplicationRecord
    has_one :profile
    delegate :first_name, :last_name, :bio, to: :profile, allow_nil: true
end
```

这行代码的意思是：==當 user 對象被調用 first_name、last_name 和 bio 功能時，user 委托给 profile 对象来執行。==

#### 2. Use Nested Attributes Instead

For forms and updates, you can use `accepts_nested_attributes_for`:

```ruby
class User < ApplicationRecord
    has_one :profile
    accepts_nested_attributes_for :profile
end
```

This doesn't provide direct delegation but helps with form handling:

```ruby
# In controller
def user_params
	params.require(:user).permit(:email, profile_attributes: [:first_name, :last_name])
end
```

### 3. Delegate Missing Methods (Decoration Pattern)

For full automatic delegation:

```ruby
class User < ApplicationRecord
    has_one :profile
    delegate_missing_to :profile, allow_nil: true
end
```

This is more "magical" but delegates all undefined methods to the profile.