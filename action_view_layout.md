# Layouts

When Rails renders a view as a response, it does so by combining the view with the current layout, using the rules for finding the current layout.

## Finding Layouts

To find the current layout, Rails first looks for a file in `app/views/layouts` with the same base name as the controller. For example, rendering actions from the `PhotosController` class will use `app/views/layouts/photos.html.erb` (or `app/views/layouts/photos.builder`). If there is no such c**ontroller-specific layout**, Rails will use `app/views/layouts/application.html.erb` or `app/views/layouts/application.builder`. If there is no `.erb` layout, Rails will use a `.builder` layout if one exists. Rails also provides several ways to more precisely assign specific layouts to individual controllers and actions.

### Specifying Layouts for Controllers

You can override the default layout conventions in your controllers by using the [`layout`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Layouts/ClassMethods.html#method-i-layout) declaration. For example:

```ruby
class ProductsController < ApplicationController
	layout "inventory"
end
```

With this declaration, all of the views rendered by the `ProductsController` will use `app/views/layouts/inventory.html.erb` as their layout.

To assign a specific layout for the entire application, use a `layout` declaration in your `ApplicationController` class:

```ruby
class ApplicationController < ActionController::Base
	layout "main"
end
```

With this declaration, all of the views in the entire application will use `app/views/layouts/main.html.erb` for their layout.

### Choosing Layouts at Runtime

You can use a symbol to defer the choice of layout until a request is processed:

```ruby
class ProductsController < ApplicationController
    layout :products_layout

    def show
    	@product = Product.find(params[:id])
    end

private
    def products_layout
    	@current_user.special? ? "special" : "products"
    end
end
```

Now, if the current user is a special user, they'll get a special layout when viewing a product.

You can even use an inline method, such as a Proc, to determine the layout. For example, if you pass a Proc object, the block you give the Proc will be given the `controller` instance, so the layout can be determined based on the current request:

```
class ProductsController < ApplicationController
  layout Proc.new { |controller| controller.request.xhr? ? "popup" : "application" }
end
```

### Conditional Layouts

Layouts specified at the controller level support the `:only` and `:except` options. These options take either a method name, or an array of method names, corresponding to method names within the controller:

```ruby
class ProductsController < ApplicationController
	layout "product", except: [:index, :rss]
end
```

With this declaration, the `product` layout would be used for everything but the `rss` and `index` methods.

### Layout Inheritance

Layout declarations cascade downward in the hierarchy, and more specific layout declarations always override more general ones. For example:

- `application_controller.rb`

	```
	class ApplicationController < ActionController::Base
	  layout "main"
	end
	```

	

- `articles_controller.rb`

	```
	class ArticlesController < ApplicationController
	end
	```

	

- `special_articles_controller.rb`

	```
	class SpecialArticlesController < ArticlesController
	  layout "special"
	end
	```

	

- `old_articles_controller.rb`

	```
	class OldArticlesController < SpecialArticlesController
	  layout false
	
	  def show
	    @article = Article.find(params[:id])
	  end
	
	  def index
	    @old_articles = Article.older
	    render layout: "old"
	  end
	  # ...
	end
	```

	

In this application:

- In general, views will be rendered in the `main` layout
- `ArticlesController#index` will use the `main` layout
- `SpecialArticlesController#index` will use the `special` layout
- `OldArticlesController#show` will use no layout at all
- `OldArticlesController#index` will use the `old` layout

#### Template Inheritance

==If a template or partial is not found in the conventional path, the controller will look for a template or partial to render in its inheritance chain.== For example:

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
end

# app/controllers/admin_controller.rb
class AdminController < ApplicationController
end

# app/controllers/admin/products_controller.rb
class Admin::ProductsController < AdminController
    def index
    end
end
```

The lookup order for an `admin/products#index` action will be:

- `app/views/admin/products/`
- `app/views/admin/`
- `app/views/application/`

This makes `app/views/application/` a great place for your shared partials, which can then be rendered in your ERB as such:

```erb
<%# app/views/admin/products/index.html.erb %>
<%= render @products || "empty_list" %>

<%# app/views/application/_empty_list.html.erb %>
There are no items in this list <em>yet</em>.
```

## Understanding `yield`

Within the context of a layout, ==`yield` identifies a section where content from the view should be inserted==. ==一個yield是layout中的一個區域錨點，用來被插入一個view文件中的內容。==

The simplest way to use this is to have a single `yield`, into which the entire contents of the view currently being rendered is inserted:

```erb
<html>
<head>
</head>
<body>
	<%= yield %>
</body>
</html>
```

You can also create a layout with multiple yielding regions:

```erb
<html>
    <head>
    	<%= yield :head %>
    </head>
    <body>
    	<%= yield %>
    </body>
</html>
```

==The main body of the view will always render into the unnamed `yield`.== To render content into **a named `yield`**, call the `content_for` method with the same argument as the named `yield`.

==`<%= yield :head %>` 釋讀為：此為一個名字為head的yield錨點。==

Newly generated applications will include `<%= yield :head %>` within the `<head>` element of its `app/views/layouts/application.html.erb` template.

### Using the `content_for` Method

The [`content_for`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/CaptureHelper.html#method-i-content_for) method allows you to insert content into a named `yield` block in your layout. For example, this view would work with the layout that you just saw:

```erb
<% content_for :head do %>
      <title>A simple page</title>
<% end %>
<p>Hello, Rails!</p>
```

`<% content_for :head do %>` 釋讀為：此間為要插入名為head的yield的內容。 %>

The result of rendering this page into the supplied layout would be this HTML:

```
<html>
  <head>
    <title>A simple page</title>
  </head>
  <body>
    <p>Hello, Rails!</p>
  </body>
</html>
```

The `content_for` method is very helpful when your layout contains distinct regions such as sidebars and footers that should get their own blocks of content inserted. It's also useful for inserting page-specific JavaScript `<script>` elements, CSS `<link>` elements, context-specific `<meta>` elements, or any other elements into the `<head>` of an otherwise generic layout.

### `content_for( name, content = nil, options = {}, &block )`

==Calling `content_for` stores a block of markup in an identifier for later use.== In order to access this stored content in other templates, helper modules or the layout, you would pass the identifier as an argument to `content_for`.

```erb
<% content_for :not_authorized do %><%# 定義 %>
	alert('You are not authorized to do that!')
<% end %>
```

You can then use `content_for :not_authorized` anywhere in your templates.

```erb
<%= content_for :not_authorized if current_user.nil? %><%# 引用 %>
```

This is equivalent to: 這個看起來更自然、明白。

```erb
<%= yield :not_authorized if current_user.nil? %>
```

Note: `yield` can still be used to retrieve the stored content, but calling `yield` doesn’t work in helper modules, while `content_for` does.

```ruby
module StorageHelper
    def stored_content
    	content_for(:storage) || "Your storage is empty"
    end
end
```

This helper works just like normal helpers.

```
<%= stored_content %>
```

You can also use the `yield` syntax alongside an existing call to `yield` in a layout. For example:

```
<%# This is the layout %>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
  <title>My Website</title>
  <%= yield :script %>
</head>
<body>
  <%= yield %>
</body>
</html>
```

And now, we’ll create a view that has a [`content_for`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/CaptureHelper.html#method-i-content_for) call that creates the `script` identifier.

```
<%# This is our view %>
Please login!

<% content_for :script do %>
  <script>alert('You are not authorized to view this page!')</script>
<% end %>
```

Then, in another view, you could to do something like this:

```
<%= link_to 'Logout', action: 'logout', remote: true %>

<% content_for :script do %>
  <%= javascript_include_tag :defaults %>
<% end %>
```

That will place `script` tags for your default set of JavaScript files on the page; this technique is useful if you’ll only be using these scripts in a few views.

Note that [`content_for`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/CaptureHelper.html#method-i-content_for) concatenates (default) the blocks it is given for a particular identifier in order. For example:

```
<% content_for :navigation do %>
  <li><%= link_to 'Home', action: 'index' %></li>
<% end %>
```

And in another place:

```
<% content_for :navigation do %>
  <li><%= link_to 'Login', action: 'login' %></li>
<% end %>
```

Then, in another template or layout, this code would render both links in order:

```
<ul><%= content_for :navigation %></ul>
```

If the flush parameter is `true` [`content_for`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/CaptureHelper.html#method-i-content_for) replaces the blocks it is given. For example:

```
<% content_for :navigation do %>
  <li><%= link_to 'Home', action: 'index' %></li>
<% end %>

<%# Add some other content, or use a different template: %>

<% content_for :navigation, flush: true do %>
  <li><%= link_to 'Login', action: 'login' %></li>
<% end %>
```

Then, in another template or layout, this code would render only the last link:

```
<ul><%= content_for :navigation %></ul>
```

Lastly, simple content can be passed as a parameter:

```
<% content_for :script, javascript_include_tag(:defaults) %>
```

WARNING: [`content_for`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/CaptureHelper.html#method-i-content_for) is ignored in caches. So you shouldn’t use it for elements that will be fragment cached.



### [3.5. Using Nested Layouts](https://guides.rubyonrails.org/layouts_and_rendering.html#using-nested-layouts)

You may find that your application requires a layout that differs slightly from your regular application layout to support one particular controller. Rather than repeating the main layout and editing it, you can accomplish this by using nested layouts (sometimes called sub-templates). Here's an example:

Suppose you have the following `ApplicationController` layout:

- `app/views/layouts/application.html.erb`

	```
	<html>
	<head>
	  <title><%= @page_title or "Page Title" %></title>
	  <%= stylesheet_link_tag "layout" %>
	  <%= yield :head %>
	</head>
	<body>
	  <div id="top_menu">Top menu items here</div>
	  <div id="menu">Menu items here</div>
	  <div id="content"><%= content_for?(:content) ? yield(:content) : yield %></div>
	</body>
	</html>
	```

	

On pages generated by `NewsController`, you want to hide the top menu and add a right menu:

- `app/views/layouts/news.html.erb`

	```
	<% content_for :head do %>
	  <style>
	    #top_menu {display: none}
	    #right_menu {float: right; background-color: yellow; color: black}
	  </style>
	<% end %>
	<% content_for :content do %>
	  <div id="right_menu">Right menu items here</div>
	  <%= content_for?(:news_content) ? yield(:news_content) : yield %>
	<% end %>
	<%= render template: "layouts/application" %>
	```

	

That's it. The News views will use the new layout, hiding the top menu and adding a new right menu inside the "content" div.

There are several ways of getting similar results with different sub-templating schemes using this technique. Note that there is no limit in nesting levels. One can use the `ActionView::render` method via `render template: 'layouts/news'` to base a new layout on the News layout. If you are sure you will not subtemplate the `News` layout, you can replace the `content_for?(:news_content) ? yield(:news_content) : yield` with simply `yield`.