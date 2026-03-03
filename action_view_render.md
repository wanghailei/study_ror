# Rendering(展示) Responses

When it's time to send a response back to the user, the Controller hands things off to the View. In broad strokes, this involves deciding what should be sent as the response and calling an appropriate method to create that response. If the response is a full-blown view, Rails also does some extra work to wrap the view in a layout and possibly to pull in partial views.

From the controller's point of view, there are three ways to create an HTTP response:

- Call [`render`](https://api.rubyonrails.org/v8.1.1/classes/ActionController/Rendering.html#method-i-render) to create a full response to send back to the browser
- Call [`redirect_to`](https://api.rubyonrails.org/v8.1.1/classes/ActionController/Redirecting.html#method-i-redirect_to) to send an HTTP redirect status code to the browser
- Call [`head`](https://api.rubyonrails.org/v8.1.1/classes/ActionController/Head.html#method-i-head) to create a response consisting solely of HTTP headers to send back to the browser

By default, controllers in Rails automatically render views with names that correspond to valid routes. ==The rule is that if you do not explicitly render something at the end of a controller action, Rails will automatically look for the `action_name.html.erb` template in the controller's view path and render it.== The actual rendering is done by nested classes of the module [`ActionView::Template::Handlers`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Template/Handlers.html).

## Using `render`

In most cases, the controller's [`render`](https://api.rubyonrails.org/v8.1.1/classes/ActionController/Rendering.html#method-i-render) method does the heavy lifting of rendering your application's content for use by a browser. There are a variety of ways to customize the behavior of `render`. You can render the default view for a Rails template, or a specific template, or a file, or inline code, or nothing at all. You can render text, JSON, or XML. You can specify the content type or HTTP status of the rendered response as well.

==If you want to see the exact results of a call to `render` without needing to inspect it in a browser, you can call `render_to_string`.== This method takes exactly the same options as `render`, but it returns a string instead of sending a response back to the browser.

### Rendering an Action's View

If you want to render the view that corresponds to a different template within the same controller, you can use `render` with the name of the view:

==If you prefer, you can use a symbol instead of a string to specify the action to render==:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render :edit, status: :unprocessable_entity
  end
end
```

#### Rendering an Action's Template from Another Controller

What if you want to render a template from an entirely different controller from the one that contains the action code? You can also do that with `render`, which accepts the full path (relative to `app/views`) of the template to render. 

For example, if you're running code in an `AdminProductsController` that lives in `app/controllers/admin`, you can render the results of an action to a template in `app/views/products` this way:

```ruby
render "products/show"
```

Rails knows that this view belongs to a different controller because of **the embedded slash character** in the string. If you want to be explicit, you can use the `:template` option (which was required on Rails 2.2 and earlier):

```ruby
# 釋讀：展示 Products controller 的 show action 的模板。
render template: "products/show"
```

#### Wrapping it up

The above two ways of rendering (rendering the template of another action in the same controller, and rendering the template of another action in a different controller) are actually variants of the same operation.

In fact, in the `BooksController` class, inside of the `update` action where we want to render the edit template if the book does not update successfully, all of the following render calls would all render the `edit.html.erb` template in the `views/books` directory:

```ruby
render :edit
render action: :edit
render "edit"
render action: "edit"
render "books/edit"
render template: "books/edit"
```

Which one you use is really a matter of style and convention, but the rule of thumb is to use the simplest one that makes sense for the code you are writing.

## Using `render` with `:inline`

The `render` method can do without a view completely, if you're willing to use the `:inline` option to supply ERB as part of the method call. This is perfectly valid:

```ruby
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

There is seldom any good reason to use this option. ==Mixing ERB into your controllers defeats the MVC orientation of Rails== and will make it harder for other developers to follow the logic of your project. Use a separate erb view instead.

By default, inline rendering uses ERB. You can force it to use Builder instead with the `:type` option:

```
render inline: "xml.p {'Horrid coding practice!'}", type: :builder
```

## Options for `render`

Calls to the [`render`](https://api.rubyonrails.org/v8.1.1/classes/ActionController/Rendering.html#method-i-render) method generally accept six options:

- `:content_type`
- `:layout`
- `:location`
- `:status`
- `:formats`
- `:variants`

#### The `:content_type` Option

By default, Rails will serve the results of a rendering operation with the MIME content-type of `text/html`(or `application/json` if you use the `:json` option, or `application/xml` for the `:xml` option.). There are times when you might like to change this, and you can do so by setting the `:content_type` option:

```ruby
render template: "feed", content_type: "application/rss"
```

#### The `:layout` Option

With most of the options to `render`, the rendered content is displayed as part of the current layout. 

You can use the `:layout` option to tell Rails to use a specific file as the layout for the current action:

```ruby
render layout: "special_layout"
```

You can also tell Rails to render with no layout at all:

```ruby
render layout: false
```

#### [2.2.13.3. The `:location` Option](https://guides.rubyonrails.org/layouts_and_rendering.html#the-location-option)

You can use the `:location` option to set the HTTP `Location` header:

```
render xml: photo, location: photo_url(photo)
```

#### [2.2.13.4. The `:status` Option](https://guides.rubyonrails.org/layouts_and_rendering.html#the-status-option)

Rails will automatically generate a response with the correct HTTP status code (in most cases, this is `200 OK`). You can use the `:status` option to change this:

```
render status: 500
render status: :forbidden
```

Rails understands both numeric status codes and the corresponding symbols shown below.

| Response Class    | HTTP Status Code | Symbol                           |
| :---------------- | :--------------- | :------------------------------- |
| **Informational** | 100              | :continue                        |
|                   | 101              | :switching_protocols             |
|                   | 102              | :processing                      |
| **Success**       | 200              | ==:ok==                          |
|                   | 201              | :created                         |
|                   | 202              | :accepted                        |
|                   | 203              | :non_authoritative_information   |
|                   | 204              | :no_content                      |
|                   | 205              | :reset_content                   |
|                   | 206              | :partial_content                 |
|                   | 207              | :multi_status                    |
|                   | 208              | :already_reported                |
|                   | 226              | :im_used                         |
| **Redirection**   | 300              | :multiple_choices                |
|                   | 301              | :moved_permanently               |
|                   | 302              | :found                           |
|                   | 303              | :see_other                       |
|                   | 304              | :not_modified                    |
|                   | 305              | :use_proxy                       |
|                   | 307              | :temporary_redirect              |
|                   | 308              | :permanent_redirect              |
| **Client Error**  | 400              | :bad_request                     |
|                   | 401              | :unauthorized                    |
|                   | 402              | :payment_required                |
|                   | 403              | :forbidden                       |
|                   | 404              | :not_found                       |
|                   | 405              | :method_not_allowed              |
|                   | 406              | :not_acceptable                  |
|                   | 407              | :proxy_authentication_required   |
|                   | 408              | :request_timeout                 |
|                   | 409              | :conflict                        |
|                   | 410              | :gone                            |
|                   | 411              | :length_required                 |
|                   | 412              | :precondition_failed             |
|                   | 413              | :payload_too_large               |
|                   | 414              | :uri_too_long                    |
|                   | 415              | :unsupported_media_type          |
|                   | 416              | :range_not_satisfiable           |
|                   | 417              | :expectation_failed              |
|                   | 421              | :misdirected_request             |
|                   | 422              | :unprocessable_entity            |
|                   | 423              | :locked                          |
|                   | 424              | :failed_dependency               |
|                   | 426              | :upgrade_required                |
|                   | 428              | :precondition_required           |
|                   | 429              | :too_many_requests               |
|                   | 431              | :request_header_fields_too_large |
|                   | 451              | :unavailable_for_legal_reasons   |
| **Server Error**  | 500              | :internal_server_error           |
|                   | 501              | :not_implemented                 |
|                   | 502              | :bad_gateway                     |
|                   | 503              | :service_unavailable             |
|                   | 504              | :gateway_timeout                 |
|                   | 505              | :http_version_not_supported      |
|                   | 506              | :variant_also_negotiates         |
|                   | 507              | :insufficient_storage            |
|                   | 508              | :loop_detected                   |
|                   | 510              | :not_extended                    |
|                   | 511              | :network_authentication_required |

If you try to render content along with a non-content status code (100-199, 204, 205, or 304), it will be dropped from the response.

#### [2.2.13.5. The `:formats` Option](https://guides.rubyonrails.org/layouts_and_rendering.html#the-formats-option)

Rails uses the format specified in the request (or `:html` by default). You can change this passing the `:formats` option with a symbol or an array:

```
render formats: :xml
render formats: [:json, :xml]
```

If a template with the specified format does not exist an `ActionView::MissingTemplate` error is raised.

#### The `:variants` Option

This tells Rails to look for template variations of the same format. You can specify a list of variants by passing the `:variants` option with a symbol or an array. An example of use would be this.

```ruby
# called in HomeController#index
render variants: [:mobile, :desktop]
```

With this set of variants Rails will look for the following set of templates and use the first that exists.

- `app/views/home/index.html+mobile.erb`
- `app/views/home/index.html+desktop.erb`
- `app/views/home/index.html.erb`

If a template with the specified format does not exist an `ActionView::MissingTemplate` error is raised.

Instead of setting the variant on the render call you may also set [`request.variant`](https://api.rubyonrails.org/v8.1.1/classes/ActionDispatch/Http/MimeNegotiation.html#method-i-variant-3D) in your controller action. 

```
def index
  request.variant = determine_variant
end

private
  def determine_variant
    variant = nil
    # some code to determine the variant(s) to use
    variant = :mobile if session[:use_mobile]

    variant
  end
```

Adding many new variant templates with similarities to existing template files can make maintaining your view code more difficult.

## Render various formats

### Rendering Text

You can send plain text - with no markup at all - back to the browser by using the `:plain` option to `render`:

```ruby
render plain: "OK"
```

Rendering pure text is most useful when you're responding to Ajax or web service requests that are expecting something other than proper HTML.

By default, if you use the `:plain` option, the text is rendered without using the current layout. If you want Rails to put the text into the current layout, you need to add the `layout: true` option and use the `.text.erb` extension for the layout file.

### Rendering HTML

You can send an HTML string back to the browser by using the `:html` option to `render`:

```ruby
render html: helpers.tag.strong("Not Found")
```

This is useful when you're rendering a small snippet of HTML code. However, you might want to consider moving it to a template file if the markup is complex.

When using `html:` option, HTML entities will be escaped if the string is not composed with `html_safe`-aware APIs.

### Rendering JSON

JSON is a JavaScript data format used by many Ajax libraries. Rails has built-in support for converting objects to JSON and rendering that JSON back to the browser:

```ruby
render json: @product
```

You don't need to call `to_json` on the object that you want to render. If you use the `:json` option, `render` will automatically call `to_json` for you.

### [2.2.8. Rendering XML](https://guides.rubyonrails.org/layouts_and_rendering.html#rendering-xml)

Rails also has built-in support for converting objects to XML and rendering that XML back to the caller:

```
render xml: @product
```



You don't need to call `to_xml` on the object that you want to render. If you use the `:xml` option, `render` will automatically call `to_xml` for you.

### Rendering Vanilla JavaScript

Rails can render vanilla JavaScript:

```ruby
render js: "alert('Hello Rails');"
```

This will send the supplied string to the browser with a MIME type of `text/javascript`.

### [2.2.10. Rendering Raw Body](https://guides.rubyonrails.org/layouts_and_rendering.html#rendering-raw-body)

You can send a raw content back to the browser, without setting any content type, by using the `:body` option to `render`:

```
render body: "raw"
```

This option should be used only if you don't care about the content type of the response. Using `:plain` or `:html` might be more appropriate most of the time.

Unless overridden, your response returned from this render option will be `text/plain`, as that is the default content type of Action Dispatch response.

### Rendering Raw File

Rails can render a raw file from an absolute path. This is useful for conditionally rendering static files like error pages.

```ruby
render file: "#{Rails.root}/public/404.html", layout: false
```

This renders the raw file (it doesn't support ERB or other handlers). By default it is rendered within the current layout.

Using the `:file` option in combination with users input can lead to security problems since an attacker could use this action to access security sensitive files in your file system.

`send_file` is often a faster and better option if a layout isn't required.

### [2.2.12. Rendering Objects](https://guides.rubyonrails.org/layouts_and_rendering.html#rendering-objects)

Rails can render objects responding to `#render_in`. The format can be controlled by defining `#format` on the object.

```
class Greeting
  def render_in(view_context)
    view_context.render html: "Hello, World"
  end

  def format
    :html
  end
end

render Greeting.new
# => "Hello World"
```

This calls `render_in` on the provided object with the current view context. You can also provide the object by using the `:renderable` option to `render`:

```
render renderable: Greeting.new
# => "Hello World"
```

## Avoiding Double Render Errors

Sooner or later, most Rails developers will see the error message "Can only render or redirect once per action". While this is annoying, it's relatively easy to fix. Usually it happens because of a fundamental misunderstanding of the way that `render` works.

For example, here's some code that will trigger this error:

```
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
  render action: "regular_show"
end
```



If `@book.special?` evaluates to `true`, Rails will start the rendering process to dump the `@book` variable into the `special_show` view. But this will *not* stop the rest of the code in the `show` action from running, and when Rails hits the end of the action, it will start to render the `regular_show` view - and throw an error. The solution is simple: make sure that you have only one call to `render` or `redirect` in a single code path. One thing that can help is `return`. Here's a patched version of the method:

```
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
    return
  end
  render action: "regular_show"
end
```



Note that the implicit render done by ActionController detects if `render` has been called, so the following will work without errors:

```
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
end
```

This will render a book with `special?` set with the `special_show` template, while other books will render with the default `show` template.
