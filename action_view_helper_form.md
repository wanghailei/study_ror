# Form Helpers

Rails provides view helpers to simplify form generation. These helpers are part of the `ActionView` module, with `FormHelper` and `FormTagHelper` serving distinct roles in form creation. ==`ActionView::Helpers::FormHelper` is for **model-based forms**, automatically handling URLs and parameter nesting==. ==`ActionView::Helpers::FormTagHelper` is for **standalone forms without models**, requiring manual URL and parameter specification==.

% Model-based Form, MBF sounds like a cool concept. 我應該建立一個工具，就是從 Form View 來建立 Model 以及數據庫，而非要開發這麼多 view、controller、model、migration。我應該把 Rails 再往前推進一步，建立一個 No Code 工具。我想，無論是創建、修改還是顯示一個 resource，都可以用 form 來實現————無外乎顯示是 read-only 或 disabled 的狀態而已。直接用一個準備好了的 form view 來 20250501 %

## `FormHelper` vs `FormTagHelper` vs `FormBuilder`



% FormHelper just renders raw HTML tags, and FormBuilder binds the tags to the model. %

==FormHelper => tags rendering**, **FormBuilder => model binding.==

FormHelper renders raw HTML form tags using methods like text_field, label, etc. ==FormHelper focuses purely on generating the HTML markup without any model-specific context.==

FormBuilder binds those tags to a model’s attributes, and calls FormHelper to do the tag rendering.



Based on the documentation and analysis:

| Aspect                | FormTagHelper                                                | FormHelper                                                   | FormBuilder                                                  |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Purpose               | Provides methods f==or creating standalone form tags without model association==, e.g., form_tag, text_field_tag. | Offers higher-level methods for generating forms, supporting both model and non-model forms, e.g., form_with, form_for. | A class instantiated by FormHelper methods for generating form fields associated with a model, e.g., text_field, checkbox. |
| Model Association     | No model association; used for standalone forms like search forms. | Supports both model-bound (e.g., editing resources) and standalone forms. | Requires a model object, used within form_with or form_for for model-aware field generation. |
| Example Methods       | form_tag, text_field_tag, checkbox_tag, select_tag, etc.     | form_with, form_for, fields_for, etc.                        | text_field, checkbox, select, file_field, etc.               |
| URL Inference         | Requires manual specification, e.g., url: articles_path.     | ==Infers URLs from the model when used with models==, e.g., article_path(@article) for updates. | Automatically handles URL inference based on the model, used within FormHelper methods. |
| Parameter Nesting     | Parameters are flat unless manually nested, e.g., name="article[title]". | Automatically nests parameters based on model name, e.g., params[:article][:title]. | Ensures proper parameter nesting based on the model, wrapping FormHelper methods. |
| Use Case Example      | Search form: <%= form_tag search_path, method: :get do %> <%= text_field_tag :query %> <% end %>. | Editing an article: `<%= form_with model: @article do        | form                                                         |
| Customization         | Limited; can create custom helpers in application_helper.rb, e.g., text_field_with_label. | Supports custom FormBuilder classes via builder: option, e.g., `<%= form_with model: @article, builder: LabellingFormBuilder do | form                                                         |
| Relation to form_with | Predecessor functionality, now discouraged in favor of form_with since Rails 5.1. | Includes form_with, the unified helper since Rails 5.1, replacing form_tag and form_for for most cases. | Integral part, object yielded by form_with for model-bound form generation. |



## `FormTagHelper`

`ActionView::Helpers::FormTagHelper` is intended for scenarios where no model is involved, offering methods like `form_tag`, `text_field_tag`, and `checkbox_tag`. It requires manual specification of URLs (e.g., `url: articles_path`) and does not automatically nest parameters, making it ==suitable for custom forms like search forms.== It provides flexibility by allowing manual specification of form attributes.

```erb
<%= form_tag articles_path do %>
	<%= text_field_tag :title %>
<% end %>
```

## `FormHelper`

`ActionView::Helpers::FormHelper` is designed for forms tied to Active Record models. It uses methods like `form_for`, `form_with`, and `fields_for` ==to generate form fields based on the model's attributes, automatically inferring the form's action URL and nesting parameters== (e.g., `params[:article][:title]`). This is ideal for creating or editing resources, such as editing an article with:

```erb
<%= form_with model: @article do |form| %>
	<%= form.text_field :title %>
<% end %>
```

==Form helpers are designed to make working with resources much easier== compared to using vanilla HTML.

Typically, a form designed to create or update a resource reflects the identity of the resource in several ways: 

- (i) the URL that the form is sent to (the form element's `action` attribute) should result in a request being routed to the appropriate controller action (with the appropriate `:id` parameter in the case of an existing resource), 
- (ii) `input` fields should be named in such a way that in the controller their values appear in the appropriate places within the `params` hash, and 
- (iii) for an existing record, when the form is initially displayed, `input` fields corresponding to attributes of the resource should show the current values of those attributes.

In Rails, this is usually achieved by creating the form using either `#form_with` or `#form_for` and a number of related helper methods. These methods generate an appropriate `form` tag and yield a form builder object that knows the model the form is about. ==Input fields are created by calling methods defined on the form builder==, which means they are able to generate the appropriate names and default values corresponding to the model attributes, as well as convenient IDs, etc. Conventions in the generated field names allow controllers to receive form data nicely structured in `params` with no effort on your side.

For example, to create a new person you typically set up a new instance of `Person` in the `PeopleController#new` action, `@person`, and in the view template pass that object to `#form_with` or `#form_for`:

```erb
<%= form_with model: @person do |f| %>
    <%= f.label :first_name %>:
    <%= f.text_field :first_name %><br />

    <%= f.label :last_name %>:
    <%= f.text_field :last_name %><br />

    <%= f.submit %>
 <% end %>
```

The HTML generated for this would be (modulus formatting):

```html
<form action="/people" class="new_person" id="new_person" method="post">
    <input name="authenticity_token" type="hidden" value="NrOp5bsjoLRuK8IW5+dQEYjKGUJDe7TQoZVvq95Wteg=" />
    <label for="person_first_name">First name</label>:
    <input id="person_first_name" name="person[first_name]" type="text" /><br />
    <label for="person_last_name">Last name</label>:
    <input id="person_last_name" name="person[last_name]" type="text" /><br />

    <input name="commit" type="submit" value="Create Person" />
</form>   
```

As you see, the HTML reflects knowledge about the resource in several spots, like the path the form should be submitted to, or the names of the input fields.

In particular, thanks to the conventions followed in the generated field names, the controller gets a nested hash `params[:person]` with the person attributes set in the form. That hash is ready to be passed to Person.new:

```ruby
@person = Person.new(params[:person])
if @person.save
	# success
else
	# error handling
end
```

Interestingly, the exact same view code in the previous example can be used to edit a person. If `@person` is an existing record with name `“John Smith”` and ID `256`, the code above as is would yield instead:

```html
<form action="/people/256" class="edit_person" id="edit_person_256" method="post">
    <input name="_method" type="hidden" value="patch" />
    <input name="authenticity_token" type="hidden" value="NrOp5bsjoLRuK8IW5+dQEYjKGUJDe7TQoZVvq95Wteg=" />

    <label for="person_first_name">First name</label>:
    <input id="person_first_name" name="person[first_name]" type="text" value="John" /><br />
    <label for="person_last_name">Last name</label>:
    <input id="person_last_name" name="person[last_name]" type="text" value="Smith" /><br />

    <input name="commit" type="submit" value="Update Person" />
 </form>
```

Note that the endpoint, default values, and submit button label are tailored for `@person`. That works that way because the involved helpers know whether the resource is a new record or not, and generate HTML accordingly.

The controller would receive the form data again in `params[:person]`, ready to be passed to `Person#update`:

```ruby
if @person.update(params[:person])
	# success
else
	# error handling
end
```

That's how you typically work with resources.

`FormHelper` methods, such as `text_field` and `checkbox`, are ==called within the context of a form builder (e.g., `|form|` in a block)==. These methods are optimized for model-based forms, providing a streamlined interface for field generation.

## form\_with

`form_with()` sets up a Ruby block environment, within which, the block’s parameter `f` is used to reference a `form` context. 

Before `form_with` was introduced in Rails 5.1, its functionality used to be split between `form_tag` and `form_for`. Both are now soft-deprecated.



% ==`form_with model: @anything` is beautiful!== %

### `form_with model:` Binding a Form to an Object

==The `form_with model:` helper sets up an HTML form that knows about Rails routes and models.==

```ruby
<%= form_with model: @order do |f| %>
    <%= f.label :name, "Name:" %>
    <%= f.text_field :name, size: 40 %>
<% end %>
```

==The `:model` argument of `form_with` binds the form builder object to a model object.== This association means that: 1) the form will be scoped to that model object, 2)the form's fields will be populated with values from that model object, and 3) submitting the form will set the right names and values in the data available to the controller.

### form\_with and RESTful resources

When dealing with RESTful resources, calls to `form_with` can get significantly easier if you rely on record identification. In short, you can just pass ==the model instance== and have Rails figure out model name and the rest.

The long and short style of editing an existing article have the same outcome:

```ruby
# long-style:
form_with( model: @article, url: article_path(@article), method: "patch" )

# short-style:
form_with model: @article
```

### Parameter Naming Conventions

Values from forms can be at the top level of the `params` hash or nested in another hash. For example, in a standard `create` action for a `Person` model, `params[:person]` would usually be a hash of all the attributes for the person to create. The `params` hash can also contain arrays, arrays of hashes, and so on.

Fundamentally HTML forms don't know about any sort of structured data, all they generate is name-value pairs, where pairs are just plain strings. The arrays and hashes you see in your application are the result of some parameter naming conventions that Rails uses.

## Creates a form tag based on mixing URLs, scopes, or models.


The parameters in the forms are accessible in controllers according to their name nesting. So inputs named `title` and `article[title]` are accessible as `params[:title]` and `params[:article][:title]` respectively.

### Resource-oriented style

In many of the examples just shown, ==the `:model` passed to `form_with` is a _resource_.== It corresponds to a set of RESTful routes, most likely defined via `resources` in `config/routes.rb`. So when passing such a model record, Rails infers the URL and method.

```erb
<%= form_with model: @article do |form| %>
<% end %> 
```

is then equivalent to something like:

```erb
<%= form_with scope: :article, url: article_path(@article), method: :patch do |form| %>
<% end %>
```

And for a new record,

```erb
<%= form_with model: Article.new do |form| %>
<% end %> 
```

is equivalent to something like:

```erb
<%= form_with scope: :article, url: articles_path do |form| %>
<% end %>
```

### form_with options

==`:url` - The URL the form submits to.== Akin to values passed to `url_for` or `link_to`. For example, you may use a named route directly. When a `:scope` is passed without a `:url`, the form just submits to the current URL.

`:method` - The method to use when submitting the form, usually either “`get`” or "`post`". If "`patch`", "`put`", "`delete`", or another verb is used, a hidden `input` named `_method` is added to simulate the verb over post.

`:format` - The format of the route the form submits to. Useful when submitting to another resource type, like `:json`. Skipped if a `:url` is passed.

`:scope` - The scope ==to prefix input field names== with and thereby how the submitted parameters are grouped in controllers.

`:namespace` - A namespace for your form to ensure uniqueness of `id` attributes on form elements. The `namespace` attribute will be prefixed with underscore on the generated HTML id.

`:model` - A model object to infer the `:url` and `:scope` by, plus fill out input field values. So if a *title* attribute is set to “Ahoy!” then a *title* input field's value would be "Ahoy!". If the model is a new record **a create form** is generated, if an existing record, however, **an update form** is generated. Pass `:scope` or `:url` to override the defaults. E.g. turn `params[:article]` into `params[:blog]`.

`:authenticity_token` - Authenticity token to use in the form. Override with a custom authenticity token or pass false to skip the authenticity token field altogether. Useful when submitting to an external resource like a payment gateway that might limit the valid fields. Remote forms may omit the embedded authenticity token by setting `config.action_view.embed_authenticity_token_in_remote_forms = false`. This is helpful when fragment-caching the form. Remote forms get the authenticity token from the `meta` tag, so embedding is unnecessary unless you support browsers without JavaScript.

`:local` - Whether to use standard HTTP form submission. When set to `true`, the form is submitted via standard HTTP. ==When set to `false`, the form is submitted as a "remote form"==, which is handled by Rails UJS as an XHR. When unspecified, the behavior is derived from `config.action_view.form_with_generates_remote_forms` where the config's value is actually the inverse of what local's value would be. As of Rails 6.1, that configuration option defaults to false (which has the equivalent effect of passing `local: true`). In previous versions of Rails, that configuration option defaults to true (the equivalent of passing local: false).

`:skip_enforcing_utf8` - If set to `true`, a hidden input with name utf8 is not output.

`:builder` - Override the object used to build the form.

==`:id` - Optional HTML id attribute.==

`:class` - Optional HTML class attribute.

`:data` - Optional HTML data attributes.

`:html` - Other optional HTML attributes for the form tag.

### Mixing with other form helpers

While `form_with` uses a `ActionView::Helpers::FormBuilder` object, it's possible to mix and match the stand-alone `ActionView::Helpers::FormHelper` methods and methods from `ActionView::Helpers::FormTagHelper`:

```erb
<%= form_with scope: :person do |form| %>
    <%= form.text_field :first_name %>
    <%= form.text_field :last_name %>

    <%= textarea :person, :biography %>
    <%= checkbox_tag "person[admin]", "1", @person.company.admin? %>

    <%= form.submit %>
<% end %>
```

Same goes for the methods in `ActionView::Helpers::FormOptionsHelper` and `ActionView::Helpers::DateHelper` designed to work with an object as a base, like `ActionView::Helpers::FormOptionsHelper#collection_select` and `DateHelper#datetime_select`.

### Setting the method

You can force the form to use the full array of HTTP verbs by setting `method: (:get|:post|:patch|:put|:delete)` in the `options` hash. If the verb is not `GET` or `POST`, which are natively supported by HTML forms, the form will be set to `POST` and a hidden input `called _method` will carry the intended verb for the server to interpret.

### Setting HTML options

You can set `data` attributes directly in a data hash, but ==HTML options besides `id` and `class` must be wrapped in an HTML key==:

```erb
<%= form_with(model: @article, data: { behavior: "autosave" }, html: { name: "go" }) do |form| %>
<% end %> 
```

generates

```html
<form action="/articles/123" method="post" data-behavior="autosave" name="go">
	<input name="_method" type="hidden" value="patch" />
</form>
```

### Removing hidden model id's

==The `form_with` method automatically includes the model id as a hidden field in the form.== This is used to maintain the correlation between the form data and its associated model. Some ORM systems do not use IDs on nested models so in this case you want to be able to disable the hidden id.

### Customized form builders

You can also build forms using a customized `ActionView::Helpers::FormBuilder` class. Subclass `ActionView::Helpers::FormBuilder` and override or define some more helpers, then ==use your custom builder==. For example, let's say you made a helper to automatically add labels to form inputs.

```erb
<%= form_with model: @person, url: { action: "create" }, builder: LabellingFormBuilder do |form| %>
    <%= form.text_field :first_name %>
    <%= form.text_field :last_name %>
    <%= form.textarea :biography %>
    <%= form.checkbox :admin %>
    <%= form.submit %>
<% end %> 
```

In this case, if you use `<%= render form %>`, the rendered template is `people/_labelling_form` and the local variable referencing the form builder is called `labelling_form`.

The custom `ActionView::Helpers::FormBuilder` class is automatically merged with the options of a nested fields call, unless it's explicitly set. In many cases you will want to wrap the above in another helper, so you could do something like the following:

```ruby
def labelled_form_with(**options, &block)
	form_with(**options.merge(builder: LabellingFormBuilder), &block)
end
```

#### When not passing a block

`form_with` just generates an opening form tag.

```erb
<%= form_with(model: @article, url: super_articles_path) %>
<%= form_with(model: @article, scope: :blog) %>
<%= form_with(model: @article, format: :json) %>
<%= form_with(model: @article, authenticity_token: false) %> # Disables the token. 
```

#### For namespaced routes, 

like `admin_article_url`:

```erb
<%= form_with(model: [ :admin, @article ]) do |form| %>
<% end %>
```

#### If the resource has associations defined, 

for example, you want to add comments to the document given that the routes are set correctly:

```erb
<%= form_with model: [ @document, Comment.new ] do |form| %>
<% end %>
```

Where `@document = Document.find(params[:id])`.

#### Using just a URL:

```erb
<%= form_with url: articles_path do |form| %>
	<%= form.text_field :title %>
<% end %>

# =>

<form action="/articles" method="post">
	<input type="text" name="title" />
</form>
```

#### With an intentionally empty URL:

```erb
<%= form_with url: false do |form| %>
	<%= form.text_field :title %>
<% end %>

# =>

<form method="post">
	<input type="text" name="title" />
</form>
```

#### Adding a scope prefixes the input field names:

```erb
<%= form_with scope: :article, url: articles_path do |form| %>
	<%= form.text_field :title %>
<% end %>

# =>

<form action="/articles" method="post">
	<input type="text" name="article[title]" />
</form>
```

#### Using a model infers both the URL and scope:

```erb
<%= form_with model: Article.new do |form| %>
	<%= form.text_field :title %>
<% end %>

# =>

<form action="/articles" method="post">
	<input type="text" name="article[title]" />
</form>
```

#### An existing model makes an update form and fills out field values:

```erb
<%= form_with model: Article.first do |form| %>
	<%= form.text_field :title %>
<% end %>

# =>

<form action="/articles/1" method="post">
	<input type="hidden" name="_method" value="patch" />
	<input type="text" name="article[title]" value="<the title of the article>" />
</form>
```

#### Though the fields don't have to correspond to model attributes:

```erb
<%= form_with model: Cat.new do |form| %>
	<%= form.text_field :cats_dont_have_gills %>
	<%= form.text_field :but_in_forms_they_can %>
<% end %>

# =>

<form action="/cats" method="post">
	<input type="text" name="cat[cats_dont_have_gills]" />
	<input type="text" name="cat[but_in_forms_they_can]" />
</form>
```



## Form Builder

==A `FormBuilder` object is associated with a particular model object and allows you to generate fields associated with the model object.== The `FormBuilder` object is yielded when using `#form_with` or `#fields_for`. For example:

```erb
<%= form_with model: @person do |person_form| %>
    Name: <%= person_form.text_field :name %>
    Admin: <%= person_form.checkbox :admin %>
<% end %>
```

In the above block, a `FormBuilder` object is yielded as the `person_form` variable. This allows you to generate the `text_field` and `checkbox` fields by specifying their eponymous methods, which modify the underlying template and associates the `@person` model object with the form.

The `FormBuilder` object can be thought of as serving as a proxy for the methods in the FormHelper `module`. This class, however, allows you to call methods with the model object you are building the form for.

You can create your own custom `ActionView::Helpers::FormBuilder` templates by subclassing this class. For example:

```ruby
class MyFormBuilder < ActionView::Helpers::FormBuilder
    def div_radio_button(method, tag_value, options = {})
        @template.content_tag(:div,
            @template.radio_button(
            	@object_name, method, tag_value, objectify_options(options)
            )
        )
    end
end
```

The above code creates a new method div_radio_button which wraps a div around the new radio button. Note that when options are passed in, you must call objectify_options in order for the model object to get correctly passed to the method. If objectify_options is not called, then the newly created helper will not be linked back to the model.

The div_radio_button code from above can now be used as follows:

 <%= form_with model: @person, :builder => MyFormBuilder do |f| %>
   I am a child: <%= f.div_radio_button(:admin, "child") %>
   I am an adult: <%= f.div_radio_button(:admin, "adult") %>
 <% end -%>

The standard set of helper methods for form building are located in the `field_helpers` class attribute.

### `FormBuilder#id`

```ruby
# Generate an HTML id attribute value.
def id
	options.dig(:html, :id) || options[:id]
end
```

It returns the `<form>` element's id attribute.

```erb
<%= form_with model: @article do |f| %>
    <% content_for :sticky_footer do %>
	    <%= form.button(form: f.id) %>
    <% end %>
<% end %>
```

In the example above, the `:sticky_footer` content area will exist outside of the `<form>` element. By declaring the form HTML attribute, we hint to the browser that the generated `<button>` element should be treated as the `<form>` element's submit button, regardless of where it exists in the DOM.



## `submit` and `button`



### `FormBuilder#submit`

Add the submit button for the given form. When no value is given, it checks if the object is a new resource or not to create the proper label:

```erb
<%= form_with model: @article do |f| %>
	<%= f.submit %>
<% end %>
```

In the example above, if `@article` is a new record, it will use “Create Article” as submit button label; otherwise, it uses "Update Article".

% `f.sumbit` 事實上生成一個 `<input type="submit">`，而不是一個 `<button>`。 %

### `FormBuilder#button`

Add **the submit button** for the given form. When no value is given, it checks if the object is a new resource or not to create the proper label:

```erb
<%= form_with model: @article do |f| %>
	<%= f.button %>
<% end %>
```

In the example above, if @article is a new record, it will use “Create Article” as button label; otherwise, it uses "Update Article".

```ruby
button("Create article")
# => <button name='button' type='submit'>Create article</button>

button(:draft, value: true)
# => <button id="article_draft" name="article[draft]" value="true" type="submit">Create article</button>

button do
	content_tag(:strong, 'Ask me!')
end
# => 
# <button name='button' type='submit'>
# 	  <strong>Ask me!</strong>
# </button>

button do |text|
	content_tag(:strong, text)
end
# => 
# <button name='button' type='submit'>
#     <strong>Create article</strong>
# </button>

button(:draft, value: true) do
	content_tag(:strong, "Save as draft")
end
# => 
# <button id="article_draft" name="article[draft]" value="true" type="submit">
#     <strong>Save as draft</strong>
# </button>
```



