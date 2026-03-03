# Resourceful Routing

Resource Routing is the Rails Default.

Resource routing allows you to quickly declare all of the common routes for a given resource controller. For example, ==a single call to method `resources` declares all of the necessary routes for the `index`, `show`, `new`, `edit`, `create`, `update`, and `destroy` actions==, without you having to declare each route separately.

## Resources on the Web

Browsers request pages from Rails by making a request for a URL using a specific HTTP verb, such as `GET`, `POST`, `PATCH`, `PUT`, and `DELETE`. Each HTTP verb is a request to perform an operation on the resource. A resource route maps related requests to actions in a single controller.

When your Rails application receives an incoming request for:

```ruby
DELETE /photos/17
```

it asks the router to map it to a controller action. If the first matching route is:

```ruby
resources :photos
```

Rails would dispatch that request to the `destroy` action on the `PhotosController` with `{ id: '17' }`in `params`.

### CRUD, Verbs, and Actions

In Rails, resourceful routes provide a mapping from incoming requests (a combination of HTTP verb + URL) to controller actions. By convention, each action generally maps to a specific [CRUD](https://edgeguides.rubyonrails.org/active_record_basics.html#crud-reading-and-writing-data) operation on your data. A single entry in the routing file, such as:

```
resources :photos
```

creates seven different routes in your application, all mapping to the `PhotosController` actions:

| HTTP Verb | Path             | Controller#Action | Used to                                      |
| :-------- | :--------------- | :---------------- | :------------------------------------------- |
| GET       | /photos          | photos#index      | display a list of all photos                 |
| GET       | /photos/new      | photos#new        | return an HTML form for creating a new photo |
| POST      | /photos          | photos#create     | create a new photo                           |
| GET       | /photos/:id      | photos#show       | display a specific photo                     |
| GET       | /photos/:id/edit | photos#edit       | return an HTML form for editing a photo      |
| PATCH/PUT | /photos/:id      | photos#update     | update a specific photo                      |
| DELETE    | /photos/:id      | photos#destroy    | delete a specific photo                      |

Since the router uses the HTTP verb *and* path to match inbound requests, four URLs can map to seven different controller actions. For example, the same `photos/` path matches to `photos#index` when the verb is `GET` and `photos#create` when the verb is `POST`.

Order matters in the `routes.rb` file. Rails routes are matched in the order they are specified. For example, if you have a `resources :photos` above a `get 'photos/poll'` the `show` action's route for the `resources` line will be matched before the `get` line. If you want the `photos/poll` route to match first, you'll need to move the `get` line **above** the `resources` line.

### Path and URL Helpers

Creating a resourceful route will also expose a number of helpers to controllers and views in your application.

For example, adding `resources :photos` to the route file will generate these `_path` helpers:

| Path Helper            | Returns URL       |
| :--------------------- | :---------------- |
| `photos_path`          | /photos           |
| `new_photo_path`       | /photos/new       |
| `edit_photo_path(:id)` | /photos/:id/edit` |
| `photo_path(:id)`      | /photos/:id       |

Parameters to the path helpers, such as `:id` above, are passed to the generated URL, such that `edit_photo_path(10)` will return `/photos/10/edit`.

Each of these `_path` helpers also have a corresponding `_url` helper (such as `photos_url`) which returns the same path prefixed with the current host, port, and path prefix.

Yes, you are exactly right. Rails path helpers are designed to be "smart" about arguments.

When you call `new_formulation_path(product_id: 123)`, Rails looks at your route definition (`/formulations/new`) and sees that there are no "slots" (like `:id` or `:product_id`) in the path itself.

Therefore, it automatically moves that argument to the **Query String**.

#### Path Helpers' arguments

- **Code:** `new_formulation_path(product_id: 5)`
- **Resulting URL:** `/formulations/new?product_id=5`

Technically, ==the path helper method accepts a Hash of options as its final argument. Any key in that hash that **does not** match a dynamic segment in the route definition is automatically converted into a URL query parameter.==

##### Why this is perfect for your "Library Model"

This is the standard, orderly way to pass "Context" without enforcing "Structure."

1. **The Link (Product View):**

	```ruby
	<%# "I am creating a formulation, and for your info, it is for Product 5" %>
	<%= link_to "New Formulation", new_formulation_path(product_id: @product.id) %>
	```

2. **The Controller (FormulationsController):**

	```ruby
	def new
		@formulation = Formulation.new
	
		# "Oh, you passed a product_id? Let me set that up for you."
		if params[:product_id]
			@product = Product.find(params[:product_id])
			@formulation.product_formulations.build(product: @product)
		end
	end
	```

This keeps your routes flat (`resources :formulations`) while preserving the user's flow.

### Defining Multiple Resources at the Same Time

If you need to create routes for more than one resource, you can save a bit of typing by defining them all with a single call to `resources`:

```
resources :photos, :books, :videos
```

The above is a shortcut for:

```
resources :photos
resources :books
resources :videos
```

### Singular Resources

Sometimes, you have a resource that users expect to have only one (i.e. it does not make sense to have an `index` action to list all values of that resource). In that case, you can use `resource` (singular) instead of `resources`.

The below resourceful route creates six routes in your application, all mapping to the `Geocoders` controller:

```ruby
resource :geocoder
resolve("Geocoder") { [:geocoder] }
```

The call to `resolve` is necessary for converting instances of the `Geocoder` to singular routes through [record identification](https://edgeguides.rubyonrails.org/form_helpers.html#relying-on-record-identification).

Here are all of the routes created for a singular resource:

| HTTP Verb | Path           | Controller#Action | Used to                                       |
| :-------- | :------------- | :---------------- | :-------------------------------------------- |
| GET       | /geocoder/new  | geocoders#new     | return an HTML form for creating the geocoder |
| POST      | /geocoder      | geocoders#create  | create the new geocoder                       |
| GET       | /geocoder      | geocoders#show    | display the one and only geocoder resource    |
| GET       | /geocoder/edit | geocoders#edit    | return an HTML form for editing the geocoder  |
| PATCH/PUT | /geocoder      | geocoders#update  | update the one and only geocoder resource     |
| DELETE    | /geocoder      | geocoders#destroy | delete the geocoder resource                  |

Singular resources map to plural controllers. For example, the `geocoder` resource maps to the `GeocodersController`.

A singular resourceful route generates these helpers:

- `new_geocoder_path` returns `/geocoder/new`
- `edit_geocoder_path` returns `/geocoder/edit`
- `geocoder_path` returns `/geocoder`

As with plural resources, the same helpers ending in `_url` will also include the host, port, and path prefix.

### Controller Namespaces and Routing

In large applications, you may wish to organize groups of controllers under a namespace. For example, you may have a number of controllers under an `Admin::` namespace, which are inside the `app/controllers/admin` directory. You can route to such a group by using a [`namespace`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Scoping.html#method-i-namespace) block:

```ruby
namespace :admin do
	resources :articles
end
```

For `Admin::ArticlesController`, Rails will create the following routes:

| HTTP Verb | Path                     | Controller#Action      | Named Route Helper           |
| :-------- | :----------------------- | :--------------------- | :--------------------------- |
| GET       | /admin/articles          | admin/articles#index   | admin_articles_path          |
| GET       | /admin/articles/new      | admin/articles#new     | new_admin_article_path       |
| POST      | /admin/articles          | admin/articles#create  | admin_articles_path          |
| GET       | /admin/articles/:id      | admin/articles#show    | admin_article_path(:id)      |
| GET       | /admin/articles/:id/edit | admin/articles#edit    | edit_admin_article_path(:id) |
| PATCH/PUT | /admin/articles/:id      | admin/articles#update  | admin_article_path(:id)      |
| DELETE    | /admin/articles/:id      | admin/articles#destroy | admin_article_path(:id)      |

Note that in the above example all of the paths have a `/admin` prefix per the default convention for `namespace`.

#### 2.6.1. Using Module

If you want to route `/articles` (without the prefix `/admin`) to `Admin::ArticlesController`, you can specify the module with a [`scope`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Scoping.html#method-i-scope) block:

```ruby
scope module: "admin" do
	resources :articles
end
```

Another way to write the above:

```ruby
resources :articles, module: "admin"
```

#### Using Scope

Alternatively, you can also route `/admin/articles` to `ArticlesController` (without the `Admin::`module prefix). You can specify the path with a `scope` block:

```ruby
scope "/admin" do
	resources :articles
end
```

Another way to write the above:

```ruby
resources :articles, path: "/admin/articles"
```

For these alternatives (without `/admin` in path and without `Admin::` in module prefix), the named route helpers remain the same as if you did not use `scope`.

In the last case, the following paths map to `ArticlesController`:

| HTTP Verb | Path                     | Controller#Action | Named Route Helper     |
| :-------- | :----------------------- | :---------------- | :--------------------- |
| GET       | /admin/articles          | articles#index    | articles_path          |
| GET       | /admin/articles/new      | articles#new      | new_article_path       |
| POST      | /admin/articles          | articles#create   | articles_path          |
| GET       | /admin/articles/:id      | articles#show     | article_path(:id)      |
| GET       | /admin/articles/:id/edit | articles#edit     | edit_article_path(:id) |
| PATCH/PUT | /admin/articles/:id      | articles#update   | article_path(:id)      |
| DELETE    | /admin/articles/:id      | articles#destroy  | article_path(:id)      |

If you need to use a different controller namespace inside a `namespace` block you can specify an absolute controller path, e.g: `get '/foo', to: '/foo#index'`.

### Routing Concerns

Routing concerns allow you to declare common routes that can be reused inside other resources. To define a concern, use a [`concern`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Concerns.html#method-i-concern) block:

```ruby
concern :commentable do
  resources :comments
end

concern :image_attachable do
  resources :images, only: :index
end
```

These concerns can be used in resources to avoid code duplication and share behavior across routes:

```
resources :messages, concerns: :commentable

resources :articles, concerns: [:commentable, :image_attachable]
```



The above is equivalent to:

```
resources :messages do
  resources :comments
end

resources :articles do
  resources :comments
  resources :images, only: :index
end
```



You can also call [`concerns`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Concerns.html#method-i-concerns) in a `scope` or `namespace` block to get the same result as above. For example:

```
namespace :messages do
  concerns :commentable
end

namespace :articles do
  concerns :commentable
  concerns :image_attachable
end
```



### Creating Paths and URLs from Objects

In addition to using the routing helpers, Rails can also create paths and URLs from an array of parameters. For example, suppose you have this set of routes:

```ruby
resources :magazines do
	resources :ads
end
```

When using `magazine_ad_path`, you can pass in instances of `Magazine` and `Ad` instead of the numeric IDs:

```erb
<%= link_to 'Ad details', magazine_ad_path(@magazine, @ad) %>
```

The generated path will be something like `/magazines/5/ads/42`.

You can also use [`url_for`](https://edgeapi.rubyonrails.org/classes/ActionView/RoutingUrlFor.html#method-i-url_for) with an array of objects to get the above path, like this:

```
<%= link_to 'Ad details', url_for([@magazine, @ad]) %>
```

In this case, Rails will see that `@magazine` is a `Magazine` and `@ad` is an `Ad` and will therefore use the `magazine_ad_path` helper. An even shorter way to write that [`link_to`](https://edgeapi.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to) is to specify just the object instead of the full [`url_for`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/UrlFor.html) call:

```
<%= link_to 'Ad details', [@magazine, @ad] %>
```

If you wanted to link to just a magazine:

```
<%= link_to 'Magazine details', @magazine %>
```

For other actions, you need to insert the action name as the first element of the array, for `edit_magazine_ad_path`:

```
<%= link_to 'Edit Ad', [:edit, @magazine, @ad] %>
```

This allows you to treat instances of your models as URLs, and is a key advantage to using the resourceful style.

In order to automatically derive paths and URLs from objects such as `[@magazine, @ad]`, Rails uses methods from [`ActiveModel::Naming`](https://edgeapi.rubyonrails.org/classes/ActiveModel/Naming.html) and [`ActiveModel::Conversion`](https://edgeapi.rubyonrails.org/classes/ActiveModel/Conversion.html) modules. Specifically, the `@magazine.model_name.route_key` returns `magazines` and `@magazine.to_param` returns a string representation of the model's `id`. So the generated path may be something like `/magazines/1/ads/42` for the objects `[@magazine, @ad]`.

### [2.10. Adding More RESTful Routes](https://edgeguides.rubyonrails.org/routing.html#adding-more-restful-routes)

You are not limited to the [seven routes](https://edgeguides.rubyonrails.org/routing.html#crud-verbs-and-actions) that RESTful routing creates by default. You can add additional routes that apply to the collection or individual members of the collection.

The below sections describe adding member routes and collection routes. The term `member` refers to routes acting on a single element, such as `show`, `update`, or `destroy`. The term `collection` refers to routes acting on multiple, or a collection of, elements, such as the `index` route.

#### [2.10.1. Adding Member Routes](https://edgeguides.rubyonrails.org/routing.html#adding-member-routes)

You can add a [`member`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Resources.html#method-i-member) block into the resource block like this:

```
resources :photos do
  member do
    get "preview"
  end
end
```



An incoming GET request to `/photos/1/preview` will route to the `preview` action of `PhotosController`. The resource id value will be available in `params[:id]`. It will also create the `preview_photo_url` and `preview_photo_path` helpers.

Within the `member` block, each route definition specifies the HTTP verb (`get` in the above example with `get 'preview'`). In addition to [`get`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-get), you can use [`patch`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-patch), [`put`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-put), [`post`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-post), or [`delete`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/HttpHelpers.html#method-i-delete).

If you don't have multiple `member` routes, you can also pass `:on` to a route, eliminating the block:

```
resources :photos do
  get "preview", on: :member
end
```



You can also leave out the `:on` option, this will create the same member route except that the resource id value will be available in `params[:photo_id]` instead of `params[:id]`. Route helpers will also be renamed from `preview_photo_url` and `preview_photo_path` to `photo_preview_url` and `photo_preview_path`.

#### [2.10.2. Adding Collection Routes](https://edgeguides.rubyonrails.org/routing.html#adding-collection-routes)

To add a route to the collection, use a [`collection`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Resources.html#method-i-collection) block:

```
resources :photos do
  collection do
    get "search"
  end
end
```



This will enable Rails to recognize paths such as `/photos/search` with GET, and route to the `search`action of `PhotosController`. It will also create the `search_photos_url` and `search_photos_path`route helpers.

Just as with member routes, you can pass `:on` to a route:

```
resources :photos do
  get "search", on: :collection
end
```



If you're defining additional resource routes with a symbol as the first positional argument, be mindful that it is not equivalent to using a string. Symbols infer controller actions while strings infer paths.

#### [2.10.3. Adding Routes for Additional New Actions](https://edgeguides.rubyonrails.org/routing.html#adding-routes-for-additional-new-actions)

To add an alternate new action using the `:on` shortcut:

```
resources :comments do
  get "preview", on: :new
end
```



This will enable Rails to recognize paths such as `/comments/new/preview` with GET, and route to the `preview` action of `CommentsController`. It will also create the `preview_new_comment_url` and `preview_new_comment_path` route helpers.

If you find yourself adding many extra actions to a resourceful route, it's time to stop and ask yourself whether you're disguising the presence of another resource.

It is possible to customise the default routes and helpers generated by `resources`, see [customizing resourceful routes section](https://edgeguides.rubyonrails.org/routing.html#customizing-resourceful-routes) for more.

## Customising Resourceful Routes

While the default routes and helpers generated by [`resources`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Resources.html#method-i-resources) will usually serve you well, you may need to customise them in some way. Rails allows for several different ways to customise the resourceful routes and helpers. This section will detail the available options.

### Specifying a Controller to Use

The `:controller` option lets you explicitly specify a controller to use for the resource. For example:

```
resources :photos, controller: "images"
```

will recognize incoming paths beginning with `/photos` but route to the `Images` controller:

| HTTP Verb | Path             | Controller#Action | Named Route Helper   |
| :-------- | :--------------- | :---------------- | :------------------- |
| GET       | /photos          | images#index      | photos_path          |
| GET       | /photos/new      | images#new        | new_photo_path       |
| POST      | /photos          | images#create     | photos_path          |
| GET       | /photos/:id      | images#show       | photo_path(:id)      |
| GET       | /photos/:id/edit | images#edit       | edit_photo_path(:id) |
| PATCH/PUT | /photos/:id      | images#update     | photo_path(:id)      |
| DELETE    | /photos/:id      | images#destroy    | photo_path(:id)      |

For namespaced controllers you can use the directory notation. For example:

```
resources :user_permissions, controller: "admin/user_permissions"
```

This will route to the `Admin::UserPermissionsController` instance.

Only the directory notation is supported. Specifying the controller with Ruby constant notation (e.g. `controller: 'Admin::UserPermissions'`) is not supported.

### Specifying Constraints on `id`

You can use the `:constraints` option to specify a required format on the implicit `id`. For example:

```ruby
resources :photos, constraints: { id: /[A-Z][A-Z][0-9]+/ }
```

This declaration constrains the `:id` parameter to match the given regular expression. The router would no longer match `/photos/1` to this route. Instead, `/photos/RR27` would match.

You can specify a single constraint to apply to a number of routes by using the block form:

```
constraints(id: /[A-Z][A-Z][0-9]+/) do
  resources :photos
  resources :accounts
end
```

You can use the more [advanced constraints](https://edgeguides.rubyonrails.org/routing.html#advanced-constraints) available in non-resourceful routes section in this context as well.

By default the `:id` parameter doesn't accept dots - this is because the dot is used as a separator for formatted routes. If you need to use a dot within an `:id` add a constraint which overrides this - for example `id: /[^\/]+/` allows anything except a slash.

### Renaming the `new` and `edit` Path Names

The `:path_names` option lets you override the default `new` and `edit` segment in paths. For example:

```ruby
resources :photos, path_names: { new: "make", edit: "change" }
```

This would allow paths such as `/photos/make` and `/photos/1/change` instead of `/photos/new` and `/photos/1/edit`.

The route helpers and controller action names aren't changed by this option. The two paths shown would have `new_photo_path` and `edit_photo_path` helpers and still route to the `new`and `edit` actions.

It is also possible to change this option uniformly for all of your routes by using a `scope` block:

```ruby
scope path_names: { new: "make" } do
	# rest of your routes
end
```

### Overriding the Named Route Helpers

The `:as` option lets you override the default naming for the route helpers. For example:

```ruby
resources :photos, as: "images"
```

This will match `/photos` and route the requests to `PhotosController` as usual, *but* use the value of the `:as` option to name the helpers `images_path` etc., as shown:

| HTTP Verb | Path             | Controller#Action | Named Route Helper   |
| :-------- | :--------------- | :---------------- | :------------------- |
| GET       | /photos          | photos#index      | images_path          |
| GET       | /photos/new      | photos#new        | new_image_path       |
| POST      | /photos          | photos#create     | images_path          |
| GET       | /photos/:id      | photos#show       | image_path(:id)      |
| GET       | /photos/:id/edit | photos#edit       | edit_image_path(:id) |
| PATCH/PUT | /photos/:id      | photos#update     | image_path(:id)      |
| DELETE    | /photos/:id      | photos#destroy    | image_path(:id)      |

### Prefixing the Named Route Helpers with `:as`

You can use the `:as` option to prefix the named route helpers that Rails generates for a route. Use this option to prevent name collisions between routes using a path scope. For example:

```ruby
scope "admin" do
  resources :photos, as: "admin_photos"
end

resources :photos
```

This changes the route helpers for `/admin/photos` from `photos_path`, `new_photos_path`, etc. to `admin_photos_path`, `new_admin_photo_path`, etc. Without the addition of `as: 'admin_photos'` on the scoped `resources :photos`, the non-scoped `resources :photos` will not have any route helpers.

To prefix a group of route helpers, use `:as` with `scope`:

```ruby
scope "admin", as: "admin" do
	resources :photos, :accounts
end

resources :photos, :accounts
```

Just as before, this changes the `/admin` scoped resource helpers to `admin_photos_path` and `admin_accounts_path`, and allows the non-scoped resources to use `photos_path` and `accounts_path`.

The `namespace` scope will automatically add `:as` as well as `:module` and `:path` prefixes.

### Using `:as` in Nested Resources

The `:as` option can override routing helper names for resources in nested routes as well. For example:

```ruby
resources :magazines do
	resources :ads, as: "periodical_ads"
end
```

This will create routing helpers such as `magazine_periodical_ads_url` and `edit_magazine_periodical_ad_path` instead of the default `magazine_ads_url` and `edit_magazine_ad_path`.

### [4.7. Parametric Scopes](https://edgeguides.rubyonrails.org/routing.html#parametric-scopes)

You can prefix routes with a named parameter:

```
scope ":account_id", as: "account", constraints: { account_id: /\d+/ } do
  resources :articles
end
```

This will provide you with paths such as `/1/articles/9` and will allow you to reference the `account_id` part of the path as `params[:account_id]` in controllers, helpers, and views.

It will also generate path and URL helpers prefixed with `account_`, into which you can pass your objects as expected:

```
account_article_path(@account, @article) # => /1/article/9
url_for([@account, @article])            # => /1/article/9
form_with(model: [@account, @article])   # => <form action="/1/article/9" ...>
```

The `:as` option is also not mandatory, but without it, Rails will raise an error when evaluating `url_for([@account, @article])` or other helpers that rely on `url_for`, such as [`form_with`](https://edgeapi.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_with).

### Restricting the Routes Created

By default, using `resources` creates routes for the seven default actions (`index`, `show`, `new`, `create`, `edit`, `update`, and `destroy`). You can use the `:only` and `:except` options to limit which routes are created.

==The `:only` option tells Rails to create only the specified routes==.

```
resources :photos, only: [:index, :show]
```

Now, a `GET` request to `/photos` or `/photos/:id` would succeed, but a `POST` request to `/photos` will fail to match.

The `:except` option specifies a route or list of routes that Rails should *not* create:

```
resources :photos, except: :destroy
```

In this case, Rails will create all of the normal routes except the route for `destroy` (a `DELETE` request to `/photos/:id`).

If your application has many RESTful routes, using `:only` and `:except` to generate only the routes that you actually need can cut down on memory use and speed up the routing process by eliminating [unused routes](https://edgeguides.rubyonrails.org/routing.html#listing-unused-routes).

### [4.9. Translated Paths](https://edgeguides.rubyonrails.org/routing.html#translated-paths)

Using `scope`, we can alter path names generated by `resources`:

```
scope(path_names: { new: "neu", edit: "bearbeiten" }) do
  resources :categories, path: "kategorien"
end
```

Rails now creates routes to the `CategoriesController`.

| HTTP Verb | Path                       | Controller#Action  | Named Route Helper      |
| :-------- | :------------------------- | :----------------- | :---------------------- |
| GET       | /kategorien                | categories#index   | categories_path         |
| GET       | /kategorien/neu            | categories#new     | new_category_path       |
| POST      | /kategorien                | categories#create  | categories_path         |
| GET       | /kategorien/:id            | categories#show    | category_path(:id)      |
| GET       | /kategorien/:id/bearbeiten | categories#edit    | edit_category_path(:id) |
| PATCH/PUT | /kategorien/:id            | categories#update  | category_path(:id)      |
| DELETE    | /kategorien/:id            | categories#destroy | category_path(:id)      |

### [4.10. Specifying the Singular Form of a Resource](https://edgeguides.rubyonrails.org/routing.html#specifying-the-singular-form-of-a-resource)

If you need to override the singular form of a resource, you can add a rule to Active Support Inflector via [`inflections`](https://edgeapi.rubyonrails.org/classes/ActiveSupport/Inflector.html#method-i-inflections):

```
ActiveSupport::Inflector.inflections do |inflect|
  inflect.irregular "tooth", "teeth"
end
```

### [4.11. Renaming Default Route Parameter `id`](https://edgeguides.rubyonrails.org/routing.html#renaming-default-route-parameter-id)

It is possible to rename the default parameter name `id` with the `:param` option. For example:

```
resources :videos, param: :identifier
```

Will now use `params[:identifier]` instead of `params[:id]`.

```
    videos GET  /videos(.:format)                  videos#index
           POST /videos(.:format)                  videos#create
 new_video GET  /videos/new(.:format)              videos#new
edit_video GET  /videos/:identifier/edit(.:format) videos#edit
```



```
Video.find_by(id: params[:identifier])

# Instead of
Video.find_by(id: params[:id])
```



You can override [`ActiveRecord::Base#to_param`](https://edgeapi.rubyonrails.org/classes/ActiveRecord/Integration.html#method-i-to_param) of the associated model to construct a URL:

```
class Video < ApplicationRecord
  def to_param
    identifier
  end
end
```



```
irb> video = Video.find_by(identifier: "Roman-Holiday")
irb> edit_video_path(video)
=> "/videos/Roman-Holiday/edit"
```

