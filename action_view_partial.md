# Partials

Partial templates - usually just called "partials" - are another device for breaking the rendering process into more manageable chunks. With a partial, you can move the code for rendering a particular piece of a response to its own file.

To render a partial as part of a view, you use the [`render`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/RenderingHelper.html#method-i-render) method within the view:

```erb
<%= render "menu" %>
```

This will render a file named `_menu.html.erb` at that point within the view being rendered. Note the leading underscore character: partials are named with a leading underscore to distinguish them from regular views, even though they are referred to without the underscore. This holds true even when you're pulling in a partial from another folder:

```erb
<%= render "application/menu" %>
```

Since view partials rely on the same Template Inheritance as templates and layouts, that code will pull in the partial from `app/views/application/_menu.html.erb`.

## Using Partials to Simplify Views

One way to use partials is to treat them as the equivalent of subroutines: as a way to move details out of a view so that you can grasp what's going on more easily. For example, you might have a view that looked like this:

```erb
<%= render "application/ad_banner" %>
<h1>Products</h1>
<p>Here are a few of our fine products:</p>
<%# ... %>
<%= render "application/footer" %>
```

Here, the `_ad_banner.html.erb` and `_footer.html.erb` partials could contain content that is shared by many pages in your application. You don't need to see the details of these sections when you're concentrating on a particular page.

As seen in the previous sections of this guide, `yield` is a very powerful tool for cleaning up your layouts. Keep in mind that it's pure Ruby, so you can use it almost everywhere. For example, we can use it to DRY up form layout definitions for several similar resources:

- `users/index.html.erb`

	```erb
	<%= render "application/search_filters", search: @q do |form| %>
		<p>Name contains: <%= form.text_field :name_contains %></p>
	<% end %>
	```

	

- `roles/index.html.erb`

	```erb
	<%= render "application/search_filters", search: @q do |form| %>
		<p>Title contains: <%= form.text_field :title_contains %></p>
	<% end %>
	```

	

- `application/_search_filters.html.erb`

	```erb
	<%= form_with model: search do |form| %>
	    <h1>Search form:</h1>
	    <fieldset><%= yield form %></fieldset>
	    <p><%= form.submit "Search" %></p>
	<% end %>
	```

	For content that is shared among all pages in your application, you can use partials directly from layouts.

## The most explicit way

```erb
<%= render "formulas/show", formula: @formula %>
```

This is the most robust, precise and explicit way to render a specific named partial. It tells Rails exactly **what partial** to use and **what data** to give it. 

1. The File Lookup (`"formulas/show"`)

- **Absolute Path:** It ignores your current location and looks strictly inside `app/views/formulas/`.
- **Partial Convention:** It automatically looks for the underscore prefix: `_show.html.erb`.
- **Target:** `app/views/formulas/_show.html.erb`

2. The Data Contract (`formula: @formula`)

- **Injection:** ==It takes the value of `@formula` (from your controller) and injects it into the partial as a **local variable** named `formula`.==
- **Decoupling:** Inside the partial, you simply use `formula`. You do **not** use `@formula`.
	- *Why this is "Beauty in Order":* It breaks the dependency on the controller. You can now render this partial anywhere (e.g., inside a "Category" page) by passing a different formula object, and it will still work perfectly.

### Equivalnant styles

These are exact semantic equivalent. the "hash style":

```erb
<%= render partial: "customers/customer", locals: { customer: @customer } %>
```

or the "string style":

```erb
<%= render "customers/customer", customer: @customer %>
```

1. File Path: app/views/customers/_customer.html.erb

	(Note: Plural folder, singular filename.)

2. Local Variable: It passes the object into the partial as `customer`.

### Data binding

```erb
<%= render "customers/customer", customer: @customer %>
```

> Is the "`customer: @customer`" part necessary? when to use, when not to? or should it be explicit at all?

Yes, the "`customer: @customer`" part is **absolutely necessary** if you are using the string path syntax, or `locals: { customer: @customer }` necesary if using the hash style.

==When you use the string path `<%= render "customers/customer" %>`, Rails performs a **file lookup only**. It does **not** perform any data binding magic.==

If `customer: @customer` is omitted, inside _customer.html.erb, the local variable `customer` will be `undefined`. The partial would only work if it accessed the instance variable `@customer` directly. 

#### When to use it

You use the explicit `customer: @customer` when you need to **control the interface**.

- **Scenario A: Mismatched Names**

	Your partial expects customer, but your data is named differently.

	```erb
	<%# Rendering a supplier's contact using the customer card %>
	<%= render "customers/customer", customer: @supplier.contact_person %>
	```

- **Scenario B: Contextual Overrides**

	You are rendering the standard card, but with a flag.

	```erb
	<%= render "customers/customer", customer: @customer, show_balance: false %>
	```

#### When NOT to use it (The "Better" Way)

If you are just passing the standard object to its standard partial, **do not use the string path at all**.

Instead of this verbose redundancy:

```erb
<%= render "customers/customer", customer: @customer %>
```

Use the "Beauty in Order" object syntax:

```erb
<%= render @customer %>
```

### Reusable Case

To reuse a partial for a generic user or a nested object (like `supplier.contact_person`), use `<%= render "path/to/partial", local_name: @data %>`.

我想，_side_head.html.erb 可能會用到這種模式。

## Rendering a model

If you have an instance of a model to render into a partial:

```erb
<%= render @customer %>
```

Assuming that the `@customer` instance variable contains an instance of the `Customer` model, this will use `_customer.html.erb` to render it and will pass the local variable `customer` into the partial which will refer to the `@customer` instance variable in the parent view.

It is a shorthand and equivalent to the hash style and the string style mentioned previously.

### This is the "Purest" Form

For someone who believes "Beauty is Order" (美就是秩序), this is the most beautiful line in Rails. It enforces a strict **Entity-to-View binding**. It implies that `_customer.html.erb` is the canonical, standard visual representation of a Customer in your system.

Since you are building **BOS** (likely with namespaced models for different domains like `Bakery` or `Factory`), this shorthand is even smarter. If `@customer` is actually an instance of `Factory::Customer` (a namespaced model):

1. **File Path:** `app/views/factory/customers/_customer.html.erb`
2. **Local Variable:** It passes the variable as `customer` (or `factory_customer` depending on exact Rails configuration, usually the underscored class name).

This keeps your view structure exactly aligned with your Ruby class hierarchy automatically.

### What if `@customer` is nil?

If `@customer` happens to be `nil`, `<%= render @customer %>` renders **nothing**. It fails silently and gracefully, saving you from writing `if @customer` checks in your view.

## Rendering a collection

When pass a collection to a partial via the `:collection` option, the partial will be inserted once for each member in the collection.

- `index.html.erb`

```erb
<%= render partial: "product", collection: @products %>
```

- _product.html.erb

	```erb
	<p>Product Name: <%= product.name %></p>
	```

	==When a partial is called with a pluralized collection, then the individual instances of the partial have access to the member of the collection being rendered via a variable named after the partial.== In this case, the partial is `_product`, and within the `_product` partial, you can refer to `product` to get the instance that is being rendered.

	

	Assuming `@products` is a collection of `Product` instances, you can simply write in a shorthand way. Rails determines the name of the partial to use by looking at the model name in the collection. 

	```erb
	<%= render @products %>
	```
	
	### Even for a heterogeneous collection!

	In fact, you can even create a heterogeneous collection and render it this way, and Rails will choose the proper partial for each member of the collection:

	- `index.html.erb`

		```erb
		<h1>Contacts</h1>
		<%= render [customer1, employee1, customer2, employee2] %>
		```
	
- `customers/_customer.html.erb`

```erb
<p>Customer: <%= customer.name %></p>
```


- `employees/_employee.html.erb`

	```erb
	<p>Employee: <%= employee.name %></p>
	```

In this case, Rails will use the customer or employee partials as appropriate for each member of the collection.

In the event that the collection is empty, `render` will return `nil`, so it should be fairly simple to provide alternative content.

```erb
<h1>Products</h1>
<%= render(@products) || "There are no products available." %>
```

### Counter Variables

Rails also makes a counter variable available within a partial called by the collection. The variable is named after the title of the partial followed by `_counter`. 

For example, when rendering a collection `@products`, the partial `_product.html.erb` can access the variable `product_counter`. The variable indexes the number of times the partial has been rendered within the enclosing view, starting with a value of `0` on the first render.

- index.html.erb

```erb
<%= render partial: "product", collection: @products %>
```

- _product.html.erb

```erb
<%= product_counter %> 
```

0 for the first product, 1 for the second product....

This also works when the local variable name is changed using the `as:` option. So if you did `as: :item`, the counter variable would be `item_counter`.

## Relative path vs Explicit path of a partial

The difference between `<%= render "form" %>` and `<%= render "products/form" %>` lies entirely in **lookup context**.

When you use `<%= render "form" %>`, Rails performs a **relative lookup**. It assumes the partial exists in the same directory as the view currently being rendered. If you are in `app/views/products/new.html.erb`, Rails looks strictly for `app/views/products/_form.html.erb`.

When you use `<%= render "products/form" %>`, you are providing an **explicit path** from the `app/views` root. Rails ignores the current directory context and goes straight to `app/views/products/_form.html.erb`.

#### Recommendation for BOS

For an enterprise application like BOS, where clarity and order are paramount, you should follow a contextual rule.

==**Use `<%= render "form" %>` (Relative)** whenever you are inside the `products` views (e.g., `new`, `edit`, or `show`).== This is superior for two reasons:

1. **Simplicity:** It reduces noise. When you are reading code inside the `products` folder, you know you are dealing with products. ==Repeating the folder name adds redundancy, not clarity.==
2. **Portability:** If you ever refactor—say, renaming `ProductsController` to `ItemsController`—you simply rename the folder. You won't have to grep through the files to update string literals inside the views.

Use `<%= render "products/form" %>` (Explicit) only when you are rendering that partial from a **different** context. For example, if your `Admin::Dashboard` needs to render a product creation form, or if you are embedding it in a modal on a `Ingredients` page. In those cases, the absolute path is necessary to tell Rails to "reach over" into the products directory.

## Local Variables

### Partial's local variable

==Every partial also has a local variable with the same name as the partial== (minus the leading underscore). You can pass an object in to this local variable via the `:object` option:

```erb
<%= render partial: "customer", object: @new_customer %>
```

Within the `customer` partial, the `customer` variable will refer to `@new_customer` from the parent view.

To use a custom local variable name within the partial, specify the `:as` option in the call to the partial:

```erb
<%= render partial: "product", collection: @products, as: :item %>
```

With this change, you can access an instance of the `@products` collection as the `item` local variable within the partial.

### `locals: {}`

You can also pass in arbitrary local variables to any partial you are rendering with the `locals: {}` option:

```erb
<%= render partial: "product", collection: @products, as: :item, locals: {title: "Products Page"} %>
```

In this case, the partial will have access to a local variable `title` with the value "Products Page".

For example, you can use this technique to reduce duplication between `new` and edit` `pages, while still keeping a bit of distinct content:

- `new.html.erb`

	```erb
	<h1>New zone</h1>
	<%= render partial: "form", locals: {zone: @zone} %>
	```

- `edit.html.erb`

	```erb
	<h1>Editing zone</h1>
	<%= render partial: "form", locals: {zone: @zone} %>
	```

- `_form.html.erb`

	```erb
	<%= form_with model: zone do |form| %>
	    <p><b>Zone name</b>:<%= form.text_field :name %></p>
	    <%= form.submit %>
	<% end %>
	```

	Although the same partial will be rendered into both views, Action View's submit helper will return "Create Zone" for the new action and "Update Zone" for the edit action.

	### `local_assigns`

==To pass a local variable to a partial in only specific cases use the `local_assigns`.==

- `index.html.erb`

	```erb
	<%= render user.articles %>
	```

- `show.html.erb`

	```erb
	<%= render article, full: true %>
	```

- `_article.html.erb`

	```erb
	<h2><%= article.title %></h2>
	<% if local_assigns[:full] %>
		<%= simple_format article.body %>
	<% else %>
		<%= truncate article.body %>
	<% end %>
	```

	This way it is possible to use the partial without the need to declare all local variables.



### Strict Locals

If you want to enforce rigorous structure, focus on defining **what data the partial expects** rather than hardcoding where it lives. ==Using strict locals ensures that whether you render it relatively or absolutely, the interface remains solid==:

```erb
<%# app/views/products/_form.html.erb %>
<%# locals: (product:) %>

<%= form_with model: product do |f| %>
<% end %>
```

This enforces order far better than the file path syntax does.

==Strict locals is a feature that allows you to define a **required interface** for your partials.== It transforms a partial from a loose bag of variables into a component with a strict signature. It forces the code to be explicit about its dependencies.

#### The Syntax

You place a magic comment at the very top of your partial file.

```erb
<%# app/views/products/_product.html.erb %>
<%# locals: (product:, show_price: true) %>

<div class="product-card">
	<h2><%= product.name %></h2>
	<% if show_price %>
		<span class="price"><%= product.price %></span>
	<% end %>
</div>
```

#### Why this is superior

==Without strict locals, partials blindly accept whatever is passed (or implicitly available).== With strict locals:

1. **Validation:** If you call `<%= render "product", product: @item %>` and forget `show_price`, it uses the default (`true`).
2. **Safety:** If you call it **without** the required `product:` key, Rails raises an `ArgumentError` immediately. It prevents `undefined method for nil:NilClass` errors deep inside the view.
3. **Clarity:** You can open any partial and immediately see exactly what data it needs to function. You don't have to hunt for instances of `local_assigns` or variables used throughout the file.
4. **Scope Isolation:** It isolates the partial. It will *only* accept variables defined in the `locals:` collection, preventing variable leakage from the outer view scope.

#### Recommendation

Adopt this for all **reusable components** in BOS (like your ingredient cards, formula rows, or generic modal wrappers). It makes your view code self-documenting.

## Partial Layouts

A partial can use its own layout file, just as a view can use a layout. For example, you might call a partial like this:

```erb
<%= render partial: "link_area", layout: "graybar" %>
```

This would look for a partial named `_link_area.html.erb` and render it using the layout `_graybar.html.erb`. Note that layouts for partials follow the same leading-underscore naming as regular partials, and are placed in the same folder with the partial that they belong to (not in the master `layouts` folder). Also note that explicitly specifying `:partial` is required when passing additional options such as `:layout`.

#### [3.4.8. Spacer Templates](https://guides.rubyonrails.org/layouts_and_rendering.html#spacer-templates)

You can also specify a second partial to be rendered between instances of the main partial by using the `:spacer_template` option:

```
<%= render partial: @products, spacer_template: "product_ruler" %>
```

Rails will render the `_product_ruler` partial (with no data passed in to it) between each pair of `_product` partials.