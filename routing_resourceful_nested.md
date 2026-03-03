# Nested Resources

It's common to have resources that are logically children of other resources. For example, suppose your application includes these models:

```ruby
class Magazine < ApplicationRecord
	has_many :ads
end

class Ad < ApplicationRecord
	belongs_to :magazine
end
```

Nested route declarations allow you to capture this relationship in your routing:

```ruby
resources :magazines do
	resources :ads
end
```

In addition to the routes for magazines, this declaration will also route ads to an `AdsController`. Here are all of the routes for the nested `ads` resource:

| HTTP Verb | Path                                 | Controller#Action | Used to                                                      |
| :-------- | :----------------------------------- | :---------------- | :----------------------------------------------------------- |
| GET       | /magazines/:magazine_id/ads          | ads#index         | display a list of all ads for a specific magazine            |
| GET       | /magazines/:magazine_id/ads/new      | ads#new           | return an HTML form for creating a new ad belonging to a specific magazine |
| POST      | /magazines/:magazine_id/ads          | ads#create        | create a new ad belonging to a specific magazine             |
| GET       | /magazines/:magazine_id/ads/:id      | ads#show          | display a specific ad belonging to a specific magazine       |
| GET       | /magazines/:magazine_id/ads/:id/edit | ads#edit          | return an HTML form for editing an ad belonging to a specific magazine |
| PATCH/PUT | /magazines/:magazine_id/ads/:id      | ads#update        | update a specific ad belonging to a specific magazine        |
| DELETE    | /magazines/:magazine_id/ads/:id      | ads#destroy       | delete a specific ad belonging to a specific magazine        |

This will also create the usual path and url routing helpers such as `magazine_ads_url` and `edit_magazine_ad_path`. Since the `ads` resource is nested below `magazines`, The ad URLs require a magazine. The helpers can take an instance of `Magazine` as the first parameter (`edit_magazine_ad_path(@magazine, @ad)`).

### Limits to Nesting

You can nest resources within other nested resources if you like. For example:

```ruby
resources :publishers do
    resources :magazines do
	    resources :photos
    end
end
```

In the above example, the application would recognize paths such as:

```
/publishers/1/magazines/2/photos/3
```

The corresponding route helper would be `publisher_magazine_photo_url`, requiring you to specify objects at all three levels. As you can see, ==deeply nested resources can become overly complex and cumbersome to maintain.==

**==The general rule of thumb is to only nest resources 1 level deep.==**





This is a profound question that touches on the core philosophy of REST and Rails resource design. It challenges the default assumption that "nesting is good."

In a strict sense, **you never \*need\* nesting for identification.** Every record in your database has a unique ID. Therefore, `/ingredients/100` is always sufficient to find the record.

So, why do we nest? The principle is not about **Identification**, it is about **Context** and **Authorization**.



### The 3 Principles of Nesting

##### 1. Contextual Creation (The "Birth" Principle)

This is the single strongest argument for nesting. You cannot create a child without knowing its parent. An orphan ingredient is invalid data.

- **Need:** To create an `ingredient`, the server *must* know the `formula_id`.
- **The Nested Solution:** `POST /formulas/1/ingredients`
- **Why:** The URL itself carries the required foreign key (`1`). It guarantees that any data created at this endpoint belongs to Formula #1.

##### 2. Contextual Security (The "Scope" Principle)

Nesting acts as a natural firewall.

- **Scenario:** You have a multi-tenant app (e.g., different bakeries).
- **The Flat Risk:** `GET /formulas/500`. Does the current user own Formula #500? You have to manually check `current_user.bakery.formulas.find(500)`.
- **The Nested Guard:** `GET /bakeries/1/formulas/500`.
	- ==The URL implies a hierarchy.==
	- Your controller naturally loads the parent first: `@bakery = Bakery.find(1)`.
	- Then it scopes the child: `@bakery.formulas.find(500)`.
	- If Formula 500 belongs to Bakery 2, this query fails naturally (404 Not Found), providing secure isolation by default.

##### 3. Visual/UI Hierarchy (The "Breadcrumb" Principle)

Sometimes, the URL is part of the user interface.

- **URL:** `example.com/products/croissant/formulas/v1`
- **Benefit:** The URL tells a story. It helps the user understand where they are ("I am inside the Croissant product, looking at version 1").
- **Counter-argument:** In modern Single Page Applications (SPA) or Turbo-driven apps (like BOS), the URL is less visible or less critical for navigation state, weakening this argument.

### The "Zero Nesting" Philosophy

There is a school of thought (which aligns with your "Order" and "Simplicity" values) that argues **Nesting is an Anti-Pattern** for everything except creation.

**The "Zero Nesting" Rules:**

1. **Creation is Nested:** `POST /formulas/1/ingredients` (Because you need the ID).
2. **Everything Else is Flat:** `GET /ingredients/50`, `PATCH /ingredients/50`, `DELETE /ingredients/50`.
3. **Use Query Params for Lists:** Instead of `GET /formulas/1/ingredients`, use `GET /ingredients?formula_id=1`.
	- *Why:* It treats "filtering by formula" just like "filtering by name" or "filtering by date." It standardizes the interface.

### Recommendation for BOS

Given your preference for **Simplicity**, **Order**, and **Shallow Routes**:

1. ==**Keep Nesting for Creation Only:** You strictly need `POST /formulas/:id/ingredients` to ensure data integrity.==
2. **Use Shallow for Member Actions:** Always use `shallow: true`. Never nest `edit`, `update`, `destroy`, or `show` for child resources.
3. **Avoid Deep Nesting (3+ levels):** As you discovered, `/products/1/formulas/2/ingredients` is chaos. Flatter is better.

==The "Order" is: **Nest to Give Life (Create), Flatten to Live Life (Interact).**==

### Shallow Nesting

One way to avoid deep nesting is to generate the collection actions scoped under the parent - so as to get a sense of the hierarchy, ==but to not nest the member actions==. In other words, ==to only build routes with the minimal amount of information to uniquely identify the resource.==

The "member" actions are the ones that apply to an individual resource and require an ID to identify the specific resource they are acting upon, such as `show`, `edit`, etc. The "collection" actions are the ones that act on the entire set of the resource, such as `index`.

For example:

```ruby
resources :articles do
	resources :comments, only: [:index, :new, :create]
end
resources :comments, only: [:show, :edit, :update, :destroy]
```

Above we use the `:only` option which tells Rails to create only the specified routes. This idea strikes a balance between descriptive routes and deep nesting. There is a shorthand syntax to achieve just that, via the `:shallow` option:

```ruby
resources :articles do
	resources :comments, shallow: true
end
```

This will generate the exact same routes as the first example. ==You can also specify the `:shallow` option in the parent resource, in which case all of the nested resources will be shallow.==

```ruby
resources :articles, shallow: true do
    resources :comments
    resources :quotes
end
```

The articles resource above will generate the following routes:

| HTTP Verb | Path                                         | Controller#Action | Named Route Helper       |
| :-------- | :------------------------------------------- | :---------------- | :----------------------- |
| GET       | /articles/:article_id/comments(.:format)     | comments#index    | article_comments_path    |
| POST      | /articles/:article_id/comments(.:format)     | comments#create   | article_comments_path    |
| GET       | /articles/:article_id/comments/new(.:format) | comments#new      | new_article_comment_path |
| GET       | /comments/:id/edit(.:format)                 | comments#edit     | edit_comment_path        |
| GET       | /comments/:id(.:format)                      | comments#show     | comment_path             |
| PATCH/PUT | /comments/:id(.:format)                      | comments#update   | comment_path             |
| DELETE    | /comments/:id(.:format)                      | comments#destroy  | comment_path             |
| GET       | /articles/:article_id/quotes(.:format)       | quotes#index      | article_quotes_path      |
| POST      | /articles/:article_id/quotes(.:format)       | quotes#create     | article_quotes_path      |
| GET       | /articles/:article_id/quotes/new(.:format)   | quotes#new        | new_article_quote_path   |
| GET       | /quotes/:id/edit(.:format)                   | quotes#edit       | edit_quote_path          |
| GET       | /quotes/:id(.:format)                        | quotes#show       | quote_path               |
| PATCH/PUT | /quotes/:id(.:format)                        | quotes#update     | quote_path               |
| DELETE    | /quotes/:id(.:format)                        | quotes#destroy    | quote_path               |
| GET       | /articles(.:format)                          | articles#index    | articles_path            |
| POST      | /articles(.:format)                          | articles#create   | articles_path            |
| GET       | /articles/new(.:format)                      | articles#new      | new_article_path         |
| GET       | /articles/:id/edit(.:format)                 | articles#edit     | edit_article_path        |
| GET       | /articles/:id(.:format)                      | articles#show     | article_path             |
| PATCH/PUT | /articles/:id(.:format)                      | articles#update   | article_path             |
| DELETE    | /articles/:id(.:format)                      | articles#destroy  | article_path             |

The [`shallow`](https://edgeapi.rubyonrails.org/classes/ActionDispatch/Routing/Mapper/Resources.html#method-i-shallow) method with a block creates a scope inside of which every nesting is shallow. This generates the same routes as the previous example:

```ruby
shallow do
    resources :articles do
        resources :comments
        resources :quotes
    end
end
```

#### Customise shallow routes

There are two options that can be used with `scope` to customise shallow routes - `:shallow_path` and `:shallow_prefix`.

The `shallow_path` option prefixes member paths with the given parameter:

```ruby
scope shallow_path: "sekret" do
    resources :articles do
    	resources :comments, shallow: true
    end
end
```

The comments resource here will have the following routes generated for it:

| HTTP Verb | Path                                         | Controller#Action | Named Route Helper       |
| :-------- | :------------------------------------------- | :---------------- | :----------------------- |
| GET       | /articles/:article_id/comments(.:format)     | comments#index    | article_comments_path    |
| POST      | /articles/:article_id/comments(.:format)     | comments#create   | article_comments_path    |
| GET       | /articles/:article_id/comments/new(.:format) | comments#new      | new_article_comment_path |
| GET       | /sekret/comments/:id/edit(.:format)          | comments#edit     | edit_comment_path        |
| GET       | /sekret/comments/:id(.:format)               | comments#show     | comment_path             |
| PATCH/PUT | /sekret/comments/:id(.:format)               | comments#update   | comment_path             |
| DELETE    | /sekret/comments/:id(.:format)               | comments#destroy  | comment_path             |

The `:shallow_prefix` option adds the specified parameter to the `_path` and `_url` route helpers:

```
scope shallow_prefix: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end
```



The comments resource here will have the following routes generated for it:

| HTTP Verb | Path                                         | Controller#Action | Named Route Helper       |
| :-------- | :------------------------------------------- | :---------------- | :----------------------- |
| GET       | /articles/:article_id/comments(.:format)     | comments#index    | article_comments_path    |
| POST      | /articles/:article_id/comments(.:format)     | comments#create   | article_comments_path    |
| GET       | /articles/:article_id/comments/new(.:format) | comments#new      | new_article_comment_path |
| GET       | /comments/:id/edit(.:format)                 | comments#edit     | edit_sekret_comment_path |
| GET       | /comments/:id(.:format)                      | comments#show     | sekret_comment_path      |
| PATCH/PUT | /comments/:id(.:format)                      | comments#update   | sekret_comment_path      |
| DELETE    | /comments/:id(.:format)                      | comments#destroy  | sekret_comment_path      |



















## Why formulations should be nested in products?

Short version:

==Nesting is just one way to express *“we usually manage this thing in the context of that thing”*, not a requirement.==



```
resources :products do
        resources :formulations
end
```

In Rails, this only means: The **URL** for a formulation is usually under a product, e.g. `/products/123/formulations/new`. In the controller you get `params[:product_id]`. It does **not** change database foreign keys, model associations or the fundamental domain meaning.

The *domain meaning* is set by:

```
class Formulation < ApplicationRecord
        belongs_to :product
        belongs_to :formula
end
```

Because a formulation literally has `product_id`, nesting routes under products is at least *consistent* with the DB: “formulation belongs to a product”.

So, unlike Formula (which we just carefully made product-agnostic), **Formulation really does belong_to Product**. Nesting does not lie here.

### Why you  might nest formulations under products

You start from how humans think:

> “I’m looking at a Product; I want to see or edit **its formulation(s)**.”

For BOS, that’s almost always true:

- On Product profile: “Which formula is this product using now?”
- Need to add/change the formula *for this specific product*
- Need to set valid_since / valid_until **per product**

So in the UI, the “primary context” is usually **the product**, not the formula.

Nesting under products gives you:

- URLs that read like the user’s intent:

	/products/123/formulations

	/products/123/formulations/new

- The product context is explicit and always present.

	- Simple controller code: `@product = Product.find(params[:product_id])``	`


In other words: ==we model the UX truth that *“you manage formulations while looking at a product”*.==

### Why you might not nest

You’re sensing a conceptual aesthetic issue:

> “We added Formulation to cleanly separate product assignment from Formula. Why are we ‘pushing it back under product’ again?”

Reasons **not** to nest:

- You want URLs that emphasise **Formulation** as a first-class resource:

	/formulations/123/edit

- You don’t want the mental shortcut “anything nested under product is just a sub-thing of product”.

- You might also want views like “all formulations that use Formula X” or “all formulations valid on a date” that are **not** anchored to a single product.

In that case you can do:

```
resources :formulations
```

and just pass product_id via params when needed:

```
<%= link_to "New formulation", new_formulation_path(product_id: @product.id) %>
```

and in the controller:

```
@product = Product.find(params[:product_id]) if params[:product_id]
```

Same behaviour, cleaner URLs from a “resource independence” perspective.

So… should we nest in BOS?

Honest answer:

- **We do NOT need to nest**.
- ==Formulations being nested under products is a UX/URL choice, not a domain requirement.==

Given your sensitivity to conceptual purity, a very reasonable structure for BOS is:

```
resources :formulas        # global, product-agnostic
resources :formulations    # global, but usually called from Product
resources :products
```

and then:

- From Product profile: `new_formulation_path(product_id: @product.id)`
- From some global admin screen: new_formulation_path without product, let the user choose product + formula in the form.

This keeps the story clean:

- **Formula**: global recipe definition
- **Formulation**: global resource that *links* product + formula + time
- **Product**: global

And we only use product_id (or formula_id) as context when needed, not as part of the URL hierarchy.

**5. Practical takeaway**

Nesting formulations under products is *not wrong* (Formulation really does belong_to Product).

- But we **added Formulation** to make the relationships explicit and flexible; we don’t need nested routes to express that.
- For BOS, with your taste, I’d pick:

```
resources :formulas
resources :formulations
resources :products
```

- and pass product_id when you start creation from a product view.

If you want, next I can show you the **non-nested** version of FormulationsController + links + form fields, wired the way above.