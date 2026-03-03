# Rails Routing

## Routing

The Rails router matches incoming HTTP requests to specific controller actions in your Rails application based on the URL path. The router also generates path and URL helpers based on the resources configured in the router.

### Routes

Routes' primary purpose is to connect view to controller, and they also serve as a UI element--URL.

Rules:

* Always use canonical routes that conform to Rails’ defaults.
* Avoid custom actions in favor of creating new resources that use Rails’\
	default actions.
* User-friendly URLs should be added in addition to the canonical routes.

==Using resources to define routes: `resources :widgets`. It's the best way.==

`bin/rails routes -g widgets`

If the app’s routes are made up entirely of calls to resources, it becomes easy to understand the app at a high level.



The core router in Rails is `ActionDispatch::Journey::Router`, used via `ActionDispatch::Routing::RouteSet` to dispatch requests.

## The Purpose of the Rails Router

Routes' primary purpose is to connect view to controller, and they also serve as a UI element--URL. The Rails router matches incoming HTTP requests to specific controller actions in your Rails application based on the URL path. It can also forward to a [Rack](https://edgeguides.rubyonrails.org/rails_on_rack.html) application. ==The router also generates path and URL helpers based on the resources configured in the router.==

### Routing Rules

1. Always use canonical routes that conform to Rails’ defaults.
2. User-friendly URLs should be added in addition to the canonical routes.
3. Avoid custom actions in favour of creating new resources that use Rails’ default actions.
4. ==Using resources to define routes: `resources :widgets`. It's the best way.==

### Routing Incoming URLs to Code

When your Rails application receives an incoming request, it asks the router to match it to a controller action (aka method). For example, take the following incoming request:

```ruby
GET /users/17
```

If the first matching route is:

```ruby
get "/users/:id", to: "users#show"
```

The request is matched to the `UsersController` class's `show` action with `{ id: '17' }` in the `params` hash.

The `to:` option expects a `controller#action` format when passed a string. Alternatively, You can pass a symbol and use the `action:` option, instead of `to:`. You can also pass a string without a `#`, in which case the `controller:` option is used instead to `to:`. For example:

```ruby
get "/users/:id", controller: "users", action: :show
```

Rails uses snake_case for controller names when specifying routes. For example, if you have a controller named `UserProfilesController`, you would specify a route to the show action as `user_profiles#show`.

### Generating Paths and URLs from Code

==The Router automatically generates path and URL helper methods== for your application. With these methods you can avoid hard-coded path and URL strings.

For example, the `user_path` and `user_url` helper methods are available when defining the following route:

```ruby
get "/users/:id", to: "users#show", as: "user"
```

==The `as:` option is used to provide a custom name for a route, which is used when generating URL and path helpers.==

Assuming your application contains this code in the controller:

```ruby
@user = User.find(params[:id])
```

and this in the corresponding view:

```ruby
<%= link_to 'User Record', user_path(@user) %>
```

The router will generate the path `/users/17` from `user_path(@user)`. Using the `user_path` helper allows you to avoid having to hard-code a path in your views. This is helpful if you eventually move the route to a different URL, as you won't need to update the corresponding views.

It also generates `user_url`, which has a similar purpose. ==While `user_path` generates a relative URL like `/users/17`, `user_url` generates an absolute URL such as `https://example.com/users/17`== in the above example.

### Configuring the Rails Router

Routes live in `config/routes.rb`. Here is an example of what routes look like in a typical Rails application. The sections that follow will explain the different route helpers used in this file:

```ruby
Rails.application.routes.draw do
    resources :brands, only: [:index, :show] do
    	resources :products, only: [:index, :show]
    end

    resource :basket, only: [:show, :update, :destroy]

    resolve("Basket") { route_for(:basket) }
end
```

Since this is a regular Ruby source file, you can use all of Ruby's features (like conditionals and loops) to help you define your routes.

The `Rails.application.routes.draw do ... end` block that wraps your route definitions is required to establish the scope for the router DSL (Domain Specific Language) and must not be deleted.

Be careful with variable names in `routes.rb` as they can clash with the DSL methods of the router.

## Resourceful Routing

See the routing_resourceful page.

## Non-Resourceful Routes

In addition to resourceful routing with `resources`, Rails has powerful support for routing arbitrary URLs to actions. You don't get groups of routes automatically generated by resourceful routing. Instead, you set up each route separately within your application.

While you should usually use resourceful routing, there are places where non-resourceful routing is more appropriate. There's no need to try to force every last piece of your application into a resourceful framework if that's not a good fit.

One example use case for non-resourceful routing is mapping existing legacy URLs to new Rails actions.

### [3.1. Bound Parameters](https://edgeguides.rubyonrails.org/routing.html#bound-parameters)

When you set up a regular route, you supply a series of symbols that Rails maps to parts of an incoming HTTP request. For example, consider this route:

```
get "photos(/:id)", to: "photos#display"
```



If an incoming `GET` request of `/photos/1` is processed by this route, then the result will be to invoke the `display` action of the `PhotosController`, and to make the final parameter `"1"` available as `params[:id]`. This route will also route the incoming request of `/photos` to `PhotosController#display`, since `:id` is an optional parameter, denoted by parentheses in the above example.

### [3.2. Dynamic Segments](https://edgeguides.rubyonrails.org/routing.html#dynamic-segments)

You can set up as many dynamic segments within a regular route as you like. Any segment will be available to the action as part of `params`. If you set up this route:

```
get "photos/:id/:user_id", to: "photos#show"
```



This route will respond to paths such as `/photos/1/2`. The `params` hash will be { controller: 'photos', action: 'show', id: '1', user_id: '2' }.

By default, dynamic segments don't accept dots - this is because the dot is used as a separator for formatted routes. If you need to use a dot within a dynamic segment, add a constraint that overrides this – for example, `id: /[^\/]+/` allows anything except a slash.

### [3.3. Static Segments](https://edgeguides.rubyonrails.org/routing.html#static-segments)

You can specify static segments when creating a route by not prepending a colon to a segment:

```
get "photos/:id/with_user/:user_id", to: "photos#show"
```



This route would respond to paths such as `/photos/1/with_user/2`. In this case, `params` would be `{ controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### [3.4. The Query String](https://edgeguides.rubyonrails.org/routing.html#the-query-string)

The `params` will also include any parameters from the query string. For example, with this route:

```
get "photos/:id", to: "photos#show"
```



An incoming `GET` request for `/photos/1?user_id=2` will be dispatched to the `show` action of the `PhotosController` class as usual and the `params` hash will be `{ controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### [3.5. Defining Default Parameters](https://edgeguides.rubyonrails.org/routing.html#defining-default-parameters)

You can define defaults in a route by supplying a hash for the `:defaults` option. This even applies to parameters that you do not specify as dynamic segments. For example:

```
get "photos/:id", to: "photos#show", defaults: { format: "jpg" }
```



Rails would match `photos/12` to the `show` action of `PhotosController`, and set `params[:format]` to `"jpg"`.

You can also use a [`defaults`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Scoping.html#method-i-defaults) block to define the defaults for multiple items:

```
defaults format: :json do
  resources :photos
  resources :articles
end
```



You cannot override defaults via query parameters - this is for security reasons. The only defaults that can be overridden are dynamic segments via substitution in the URL path.

### [3.6. Naming Routes](https://edgeguides.rubyonrails.org/routing.html#naming-routes)

You can specify a name that will be used by the `_path` and `_url` helpers for any route using the `:as`option:

```
get "exit", to: "sessions#destroy", as: :logout
```



This will create `logout_path` and `logout_url` as the route helpers in your application. Calling `logout_path` will return `/exit`.

You can also use `as` to override routing helper names defined by `resources` by placing a custom route definition *before* the resource is defined, like this:

```
get ":username", to: "users#show", as: :user
resources :users
```



This will define a `user_path` helper that will match `/:username` (e.g. `/jane`). Inside the `show` action of `UsersController`, `params[:username]` will contain the username for the user.

### [3.7. HTTP Verb Constraints](https://edgeguides.rubyonrails.org/routing.html#http-verb-constraints)

In general, you should use the [`get`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-get), [`post`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-post), [`put`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-put), [`patch`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-patch), and [`delete`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-delete) methods to constrain a route to a particular verb. There is a [`match`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Base.html#method-i-match) method that you could use with the `:via` option to match multiple verbs at once:

```
match "photos", to: "photos#show", via: [:get, :post]
```



The above route matches GET and POST requests to the `show` action of the `PhotosController`.

You can match all verbs to a particular route using `via: :all`:

```
match "photos", to: "photos#show", via: :all
```



Routing both `GET` and `POST` requests to a single action has security implications. For example, the `GET` action won't check for CSRF token (so writing to the database from `GET` request is not a good idea. For more information see the [security guide](https://edgeguides.rubyonrails.org/security.html#csrf-countermeasures)). In general, avoid routing all verbs to a single action unless you have a good reason.

### [3.8. Segment Constraints](https://edgeguides.rubyonrails.org/routing.html#segment-constraints)

You can use the `:constraints` option to enforce a format for a dynamic segment:

```
get "photos/:id", to: "photos#show", constraints: { id: /[A-Z]\d{5}/ }
```



The above route definition requires `id` to be 5 alphanumeric characters long. Therefore, this route would match paths such as `/photos/A12345`, but not `/photos/893`. You can more succinctly express the same route this way:

```
get "photos/:id", to: "photos#show", id: /[A-Z]\d{5}/
```



The `:constraints` option takes regular expressions (as well as any object that responds to `matches?`method) with the restriction that regexp anchors can't be used. For example, the following route will not work:

```
get "/:id", to: "articles#show", constraints: { id: /^\d/ }
```



However, note that you don't need to use anchors because all routes are anchored at the start and the end.

For example:

```
get "/:id", to: "articles#show", constraints: { id: /\d.+/ }
get "/:username", to: "users#show"
```



The above routes would allow sharing the root namespace and:

- route paths that always begin with a number, like `/1-hello-world`, to `articles` with `id` value.
- route paths that never begin with a number, like `/david`, to `users` with `username` value.

### [3.9. Request-Based Constraints](https://edgeguides.rubyonrails.org/routing.html#request-based-constraints)

You can also constrain a route based on any method on the [Request object](https://edgeguides.rubyonrails.org/action_controller_overview.html#the-request-object) that returns a `String`.

You specify a request-based constraint the same way that you specify a segment constraint. For example:

```
get "photos", to: "photos#index", constraints: { subdomain: "admin" }
```



Will match an incoming request with a path to `admin` subdomain.

You can also specify constraints by using a [`constraints`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Scoping.html#method-i-constraints) block:

```
constraints subdomain: "admin" do
  resources :photos
end
```



Will match something like `https://admin.example.com/photos`.

Request constraints work by calling a method on the [Request object](https://edgeguides.rubyonrails.org/action_controller_overview.html#the-request-object) with the same name as the hash key and then comparing the return value with the hash value. For example: `constraints: { subdomain: 'api' }` will match an `api` subdomain as expected. However, using a symbol `constraints: { subdomain: :api }` will not, because `request.subdomain` returns `'api'` as a String.

Constraint values should match the corresponding Request object method return type.

There is an exception for the `format` constraint, while it's a method on the Request object, it's also an implicit optional parameter on every path. Segment constraints take precedence and the `format`constraint is only applied when enforced through a hash. For example, `get 'foo', constraints: { format: 'json' }` will match `GET  /foo` because the format is optional by default.

You can [use a lambda](https://edgeguides.rubyonrails.org/routing.html#advanced-constraints) like in `get 'foo', constraints: lambda { |req| req.format == :json }` to only match the route to explicit JSON requests.

### [3.10. Advanced Constraints](https://edgeguides.rubyonrails.org/routing.html#advanced-constraints)

If you have a more advanced constraint, you can provide an object that responds to `matches?` that Rails should use. Let's say you wanted to route all users on a restricted list to the `RestrictedListController`. You could do:

```
class RestrictedListConstraint
  def initialize
    @ips = RestrictedList.retrieve_ips
  end

  def matches?(request)
    @ips.include?(request.remote_ip)
  end
end

Rails.application.routes.draw do
  get "*path", to: "restricted_list#index",
    constraints: RestrictedListConstraint.new
end
```



You can also specify constraints as a lambda:

```
Rails.application.routes.draw do
  get "*path", to: "restricted_list#index",
    constraints: lambda { |request| RestrictedList.retrieve_ips.include?(request.remote_ip) }
end
```



Both the `matches?` method and the lambda gets the `request` object as an argument.

#### [3.10.1. Constraints in a Block Form](https://edgeguides.rubyonrails.org/routing.html#constraints-in-a-block-form)

You can specify constraints in a block form. This is useful for when you need to apply the same rule to several routes. For example:

```
class RestrictedListConstraint
  # ...Same as the example above
end

Rails.application.routes.draw do
  constraints(RestrictedListConstraint.new) do
    get "*path", to: "restricted_list#index"
    get "*other-path", to: "other_restricted_list#index"
  end
end
```



You can also use a `lambda`:

```
Rails.application.routes.draw do
  constraints(lambda { |request| RestrictedList.retrieve_ips.include?(request.remote_ip) }) do
    get "*path", to: "restricted_list#index"
    get "*other-path", to: "other_restricted_list#index"
  end
end
```



### [3.11. Wildcard Segments](https://edgeguides.rubyonrails.org/routing.html#wildcard-segments)

A route definition can have a wildcard segment, which is a segment prefixed with a star, such as `*other`:

```
get "photos/*other", to: "photos#unknown"
```



Wildcard segments allow for something called "route globbing", which is a way to specify that a particular parameter (`*other` above) be matched to the remaining part of a route.

So the above route would match `photos/12` or `/photos/long/path/to/12`, setting `params[:other]`to `"12"` or `"long/path/to/12"`.

Wildcard segments can occur anywhere in a route. For example:

```
get "books/*section/:title", to: "books#show"
```



would match `books/some/section/last-words-a-memoir` with `params[:section]` equals `'some/section'`, and `params[:title]` equals `'last-words-a-memoir'`.

Technically, a route can have even more than one wildcard segment. The matcher assigns segments to parameters in the order they occur. For example:

```
get "*a/foo/*b", to: "test#index"
```



would match `zoo/woo/foo/bar/baz` with `params[:a]` equals `'zoo/woo'`, and `params[:b]` equals `'bar/baz'`.

### [3.12. Format Segments](https://edgeguides.rubyonrails.org/routing.html#format-segments)

Given this route definition:

```
get "*pages", to: "pages#show"
```



By requesting `'/foo/bar.json'`, your `params[:pages]` will be equal to `'foo/bar'` with the request format of JSON in `params[:format]`.

The default behavior with `format` is that if included Rails automatically captures it from the URL and includes it in params[:format], but `format` is not required in a URL.

If you want to match URLs without an explicit format and ignore URLs that include a format extension, you could supply `format: false` like this:

```
get "*pages", to: "pages#show", format: false
```



If you want to make the format segment mandatory, so it cannot be omitted, you can supply `format: true` like this:

```
get "*pages", to: "pages#show", format: true
```



### [3.13. Redirection](https://edgeguides.rubyonrails.org/routing.html#redirection)

You can redirect any path to any other path by using the [`redirect`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Redirection.html#method-i-redirect) helper in your router:

```
get "/stories", to: redirect("/articles")
```



You can also reuse dynamic segments from the match in the path to redirect to:

```
get "/stories/:name", to: redirect("/articles/%{name}")
```



You can also provide a block to `redirect`, which receives the symbolized path parameters and the request object:

```
get "/stories/:name", to: redirect { |path_params, req| "/articles/#{path_params[:name].pluralize}" }
get "/stories", to: redirect { |path_params, req| "/articles/#{req.subdomain}" }
```



Please note that default redirection is a 301 "Moved Permanently" redirect. Keep in mind that some web browsers or proxy servers will cache this type of redirect, making the old page inaccessible. You can use the `:status` option to change the response status:

```
get "/stories/:name", to: redirect("/articles/%{name}", status: 302)
```



In all of these cases, if you don't provide the host (`http://www.example.com`), Rails will take those details from the current request.

### [3.14. Routing to Rack Applications](https://edgeguides.rubyonrails.org/routing.html#routing-to-rack-applications)

Instead of specifying `:to` as a String like `'articles#index'`, which corresponds to the `index` method in the `ArticlesController` class, you can specify any [Rack application](https://edgeguides.rubyonrails.org/rails_on_rack.html) as the endpoint for a matcher:

```
match "/application.js", to: MyRackApp, via: :all
```



As long as `MyRackApp` responds to `call` and returns a `[status, headers, body]`, the router won't know the difference between the Rack application and a controller action. This is an appropriate use of `via: :all`, as you will want to allow your Rack application to handle all verbs.

An interesting tidbit - `'articles#index'` expands out to `ArticlesController.action(:index)`, which returns a valid Rack application.

Since procs/lambdas are objects that respond to `call`, you can implement very simple routes (e.g. for health checks) inline, something like: `get '/health', to: ->(env) { [204, {}, ['']] }`

If you specify a Rack application as the endpoint for a matcher, remember that the route will be unchanged in the receiving application. With the following route your Rack application should expect the route to be `/admin`:

```
match "/admin", to: AdminApp, via: :all
```



If you would prefer to have your Rack application receive requests at the root path instead, use [`mount`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Base.html#method-i-mount):

```
mount AdminApp, at: "/admin"
```



### [3.15. Using `root`](https://edgeguides.rubyonrails.org/routing.html#using-root)

You can specify what Rails should route `'/'` to with the [`root`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Resources.html#method-i-root) method:

```
root to: "pages#main"
root "pages#main" # shortcut for the above
```



You typically put the `root` route at the top of the file so that it can be matched first.

The `root` route primarily handles `GET` requests by default. But it is possible to configure it to handle other verbs (e.g. `root "posts#index", via: :post`)

You can also use root inside namespaces and scopes as well:

```
root to: "home#index"

namespace :admin do
  root to: "admin#index"
end
```



The above will match `/admin` to the `index` action for the `AdminController` and match `/` to `index`action of the `HomeController`.

### [3.16. Unicode Character Routes](https://edgeguides.rubyonrails.org/routing.html#unicode-character-routes)

You can specify unicode character routes directly. For example:

```
get "こんにちは", to: "welcome#index"
```



### [3.17. Direct Routes](https://edgeguides.rubyonrails.org/routing.html#direct-routes)

You can create custom URL helpers by calling [`direct`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/CustomUrls.html#method-i-direct). For example:

```
direct :homepage do
  "https://rubyonrails.org"
end

# >> homepage_url
# => "https://rubyonrails.org"
```



The return value of the block must be a valid argument for the [`url_for`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/UrlFor.html) method. So, you can pass a valid string URL, Hash, Array, an Active Model instance, or an Active Model class.

```
direct :commentable do |model|
  [ model, anchor: model.dom_id ]
end
```



```
direct :main do
  { controller: "pages", action: "index", subdomain: "www" }
end

# >> main_url
# => "http://www.example.com/pages"
```



### [3.18. Using `resolve`](https://edgeguides.rubyonrails.org/routing.html#using-resolve)

The [`resolve`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/CustomUrls.html#method-i-resolve) method allows customizing polymorphic mapping of models. For example:

```
resource :basket

resolve("Basket") { [:basket] }
```



```
<%= form_with model: @basket do |form| %>
  <!-- basket form -->
<% end %>
```



This will generate the singular URL `/basket` instead of the usual `/baskets/:id`.

## Inspecting Routes

Rails offers a few different ways of inspecting and testing your routes.

### Listing Existing Routes

To get a complete list of routes available in an application, visit `http://localhost:3000/rails/info/routes` in the **development** environment. You can also execute the ==`bin/rails routes`== command in your terminal to get the same output.

Both methods will list all of your routes, in the same order that they appear in `config/routes.rb`. For each route, you'll see:

- The route name (if any)
- The HTTP verb used (if the route doesn't respond to all verbs)
- The URL pattern to match
- The routing parameters for the route

For example, here's a small section of the `bin/rails routes` output for a RESTful route:

```
    users GET    /users(.:format)          users#index
          POST   /users(.:format)          users#create
 new_user GET    /users/new(.:format)      users#new
edit_user GET    /users/:id/edit(.:format) users#edit
```



The route name (`new_user` above, for example) can be considered the base for deriving route helpers. To get the name of a route helper, add the suffix `_path` or `_url` to the route name (`new_user_path`, for example).

You can also use the `--expanded` option to turn on the expanded table formatting mode.

```
$ bin/rails routes --expanded

--[ Route 1 ]----------------------------------------------------
Prefix            | users
Verb              | GET
URI               | /users(.:format)
Controller#Action | users#index
--[ Route 2 ]----------------------------------------------------
Prefix            |
Verb              | POST
URI               | /users(.:format)
Controller#Action | users#create
--[ Route 3 ]----------------------------------------------------
Prefix            | new_user
Verb              | GET
URI               | /users/new(.:format)
Controller#Action | users#new
--[ Route 4 ]----------------------------------------------------
Prefix            | edit_user
Verb              | GET
URI               | /users/:id/edit(.:format)
Controller#Action | users#edit
```



### [5.2. Searching Routes](https://edgeguides.rubyonrails.org/routing.html#searching-routes)

You can search through your routes with the grep option: `-g`. This outputs any routes that partially match the URL helper method name, the HTTP verb, or the URL path.

```
$ bin/rails routes -g new_comment
$ bin/rails routes -g POST
$ bin/rails routes -g admin
```



If you only want to see the routes that map to a specific controller, there's the controller option: `-c`.

```
$ bin/rails routes -c users
$ bin/rails routes -c admin/users
$ bin/rails routes -c Comments
$ bin/rails routes -c Articles::CommentsController
```



The output from `bin/rails routes` is easier to read if you widen your terminal window until the output lines don't wrap or use the `--expanded` option.

### [5.3. Listing Unused Routes](https://edgeguides.rubyonrails.org/routing.html#listing-unused-routes)

You can scan your application for unused routes with the `--unused` option. An "unused" route in Rails is a route that is defined in the config/routes.rb file but is not referenced by any controller action or view in your application. For example:

```
$ bin/rails routes --unused
Found 8 unused routes:

     Prefix Verb   URI Pattern                Controller#Action
     people GET    /people(.:format)          people#index
            POST   /people(.:format)          people#create
 new_person GET    /people/new(.:format)      people#new
edit_person GET    /people/:id/edit(.:format) people#edit
     person GET    /people/:id(.:format)      people#show
            PATCH  /people/:id(.:format)      people#update
            PUT    /people/:id(.:format)      people#update
            DELETE /people/:id(.:format)      people#destroy
```



### [5.4. Routes in Rails Console](https://edgeguides.rubyonrails.org/routing.html#routes-in-rails-console)

You can access route helpers using `Rails.application.routes.url_helpers` within the [Rails Console](https://edgeguides.rubyonrails.org/command_line.html#bin-rails-console). They are also available via the [app](https://edgeguides.rubyonrails.org/command_line.html#the-app-and-helper-objects) object. For example:

```
irb> Rails.application.routes.url_helpers.users_path
=> "/users"

irb> user = User.first
=> #<User:0x00007fc1eab81628
irb> app.edit_user_path(user)
=> "/users/1/edit"
```



## Testing Routes

Rails offers three built-in assertions designed to make testing routes simpler:

- [`assert_generates`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_generates)
- [`assert_recognizes`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_recognizes)
- [`assert_routing`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_routing)

### [6.1. The `assert_generates` Assertion](https://edgeguides.rubyonrails.org/routing.html#the-assert-generates-assertion)

[`assert_generates`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_generates) asserts that a particular set of options generate a particular path and can be used with default routes or custom routes. For example:

```
assert_generates "/photos/1", { controller: "photos", action: "show", id: "1" }
assert_generates "/about", controller: "pages", action: "about"
```



### [6.2. The `assert_recognizes` Assertion](https://edgeguides.rubyonrails.org/routing.html#the-assert-recognizes-assertion)

[`assert_recognizes`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_recognizes) is the inverse of `assert_generates`. It asserts that a given path is recognized and routes it to a particular spot in your application. For example:

```
assert_recognizes({ controller: "photos", action: "show", id: "1" }, "/photos/1")
```



You can supply a `:method` argument to specify the HTTP verb:

```
assert_recognizes({ controller: "photos", action: "create" }, { path: "photos", method: :post })
```



### [6.3. The `assert_routing` Assertion](https://edgeguides.rubyonrails.org/routing.html#the-assert-routing-assertion)

The [`assert_routing`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_routing) assertion checks the route both ways. It combines the functionality of both `assert_generates` and `assert_recognizes`. It tests that the path generates the options, and that the options generate the path:

```
assert_routing({ path: "photos", method: :post }, { controller: "photos", action: "create" })
```



## Breaking Up a Large Route File With `draw`

In a large application with thousands of routes, a single `config/routes.rb` file can become cumbersome and hard to read. Rails offers a way to break up a single `routes.rb` file into multiple small ones using the [`draw`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Resources.html#method-i-draw) macro.

For example, you could add an `admin.rb` file that contains all the routes related to the admin area, another `api.rb` file for API related resources, etc.

```ruby
# config/routes.rb

Rails.application.routes.draw do
	get "foo", to: "foo#bar"
	draw(:admin) # Will load another route file located in `config/routes/admin.rb`
end
```

```ruby
# config/routes/admin.rb

namespace :admin do
	resources :comments
end
```

Calling `draw(:admin)` inside the `Rails.application.routes.draw` block itself will try to load a route file that has the same name as the argument given (`admin.rb` in this example). The file needs to be located inside the `config/routes` directory or any sub-directory (i.e. `config/routes/admin.rb` or`config/routes/external/admin.rb`).

You can use the normal routing DSL inside a secondary routing file such as `admin.rb`, but *do not* surround it with the `Rails.application.routes.draw` block. That should be used in the main `config/routes.rb` file only.

Don't use this feature unless you really need it. Having multiple routing files make it harder to discover routes in one place. For most applications - even those with a few hundred routes - it's easier for developers to have a single routing file. The Rails routing DSL already offers a way to break routes in an organized manner with `namespace` and `scope`.
