# ActionController Callbacks

Controller callbacks are methods that are defined to automatically run before and/or after a controller action. A controller callback method can be defined in a given controller or in the `ApplicationController`. Since all controllers inherit from `ApplicationController`, callbacks defined here will run on every controller in your application.

### `before_action`

Callback methods registered via `before_action` ==run *before* a controller action==. They may halt the request cycle. A common use case for `before_action` is ensuring that a user is logged in:

```ruby
class ApplicationController < ActionController::Base
    before_action :require_login
    private
    def require_login
		unless logged_in?
			flash[:error] = "You must be logged in to access this section"
			redirect_to new_login_url # halts request cycle
		end
	end
end
```

The method stores an error message in the flash and redirects to the login form if the user is not already logged in. When a `before_action` callback renders or redirects (like in the example above), the original controller action is not run. If there are additional callbacks registered to run, they are also cancelled and not run.

In this example, the `before_action` is defined in `ApplicationController` so all controllers in the application inherit it. That implies that all requests in the application will require the user to be logged in. This is fine except for the "login" page. The "login" action should succeed even when the user is not logged in (to allow the user to log in) otherwise the user will never be able to log in. You can use [`skip_before_action`](https://api.rubyonrails.org/v8.0.3/classes/AbstractController/Callbacks/ClassMethods.html#method-i-skip_before_action) to allow specified controller actions to skip a given `before_action`:

```ruby
class LoginsController < ApplicationController
	skip_before_action :require_login, only: [:new, :create]
end
```

Now, the `LoginsController`'s `new` and `create` actions will work without requiring the user to be logged in.

The `:only` option skips the callback only for the listed actions; there is also an `:except` option which works the other way. These options can be used when registering action callbacks too, to add callbacks which only run for selected actions.

If you register the same action callback multiple times with different options, the last action callback definition will overwrite the previous ones.

### `after_action` and `around_action`

You can also define action callbacks to run *after* a controller action has been executed with [`after_action`](https://api.rubyonrails.org/v8.0.3/classes/AbstractController/Callbacks/ClassMethods.html#method-i-after_action), or to run both before and after with [`around_action`](https://api.rubyonrails.org/v8.0.3/classes/AbstractController/Callbacks/ClassMethods.html#method-i-around_action).

The `after_action` callbacks are similar to `before_action` callbacks, but because the controller action has already been run they have access to the response data that's about to be sent to the client.

`after_action` callbacks are executed only after a successful controller action, and not if an exception is raised in the request cycle.

The `around_action` callbacks are useful when you want to execute code before and after a controller action, allowing you to encapsulate functionality that affects the action's execution. They are responsible for running their associated actions by yielding.

For example, imagine you want to monitor the performance of specific actions. You could use an `around_action` to measure how long each action takes to complete and log this information:

```ruby
class ApplicationController < ActionController::Base
  around_action :measure_execution_time

  private
    def measure_execution_time
      start_time = Time.now
      yield  # This executes the action
      end_time = Time.now

      duration = end_time - start_time
      Rails.logger.info "Action #{action_name} from controller #{controller_name} took #{duration.round(2)} seconds to execute."
    end
end
```

Action callbacks receive `controller_name` and `action_name` as parameters you can use, as shown in the example above.

The `around_action` callback also wraps rendering. In the example above, view rendering will be included in the `duration`. The code after the `yield` in an `around_action` is run even when there is an exception in the associated action and there is an `ensure` block in the callback. (This is different from `after_action` callbacks where an exception in the action cancels the `after_action` code.)

### Other Ways to Use Callbacks

In addition to `before_action`, `after_action`, or `around_action`, there are two less common ways to register callbacks.

The first is to use a block directly with the `*_action` methods. The block receives the controller as an argument. For example, the `require_login` action callback from above could be rewritten to use a block:

```
class ApplicationController < ActionController::Base
  before_action do |controller|
    unless controller.send(:logged_in?)
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url
    end
  end
end
```

Note that the action callback, in this case, uses `send` because the `logged_in?` method is private, and the action callback does not run in the scope of the controller. This is not the recommended way to implement this particular action callback, but in simpler cases, it might be useful.

Specifically for `around_action`, the block also yields in the `action`:

```
around_action { |_controller, action| time(&action) }
```

The second way is to specify a class (or any object that responds to the expected methods) for the callback action. This can be useful in cases that are more complex. As an example, you could rewrite the `around_action`callback to measure execution time with a class:

```
class ApplicationController < ActionController::Base
  around_action ActionDurationCallback
end

class ActionDurationCallback
  def self.around(controller)
    start_time = Time.now
    yield # This executes the action
    end_time = Time.now

    duration = end_time - start_time
    Rails.logger.info "Action #{controller.action_name} from controller #{controller.controller_name} took #{duration.round(2)} seconds to execute."
  end
end
```

In above example, the `ActionDurationCallback`'s method is not run in the scope of the controller but gets `controller` as an argument.

In general, the class being used for a `*_action` callback must implement a method with the same name as the action callback. So for the `before_action` callback, the class 