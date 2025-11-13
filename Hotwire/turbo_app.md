# Turbo app - a new generation of Rails app

==Turbo changes how views and controllers are structured in Rails==, using frames for partial updates and streams for dynamic DOM changes.

Turbo enables Rails applications to exhibit single-page application (SPA)-like behavior without relying on heavy client-side JavaScript frameworks like React or Vue. This is achieved through features like ==Turbo Drive for fast navigation==, ==Turbo Frames for updating blocks of components==, and ==Turbo Streams for pinpoint content changes==, all operating with server-delivered HTML.

This evolution aligns with the trend toward interactive, responsive web applications, allowing Rails to compete with modern front-end frameworks while maintaining its server-rendered HTML approach. Turbo 8's introduction of DOM morphing and page transitions, further enhancing user experience and positioning Turbo as a forward-looking component of Rails. While not explicitly termed a "new generation," ==Turbo's capabilities suggest it represents a significant modernization==, enabling developers to build applications that feel like SPAs but are rooted in Rails' server-side rendering paradigm.

However, it is important to note that Turbo does not constitute a completely new framework; it is an enhancement within the existing Rails ecosystem. It builds on Rails' conventions, such as MVC architecture and Active Record, while introducing new patterns for interactivity. This positions Turbo-based Rails apps as an evolved, modern iteration rather than a distinct generational shift. 

## Classic Rails Views and Controllers Structure

In a classic Rails architecture, each CRUD action corresponds to an ERB template, and user interactions often trigger full-page updates:

### View Structure

Pages are composed of ERB templates, with reusable partials for shared components (e.g. form fields or list items). For example, an index.html.erb might render a list of records via a partial, and a separate new.html.erb or edit.html.erb holds the form for creating or editing a record. There is a clear separation: listing happens on one page, and forms appear on dedicated pages. If the user clicks “New” or “Edit,” the browser navigates to a new page to show the form. After form submission, the user is redirected back to a show or index page to see the results. This means ==users **frequently switch pages** and context during CRUD operations.==

### Controller Behavior 

Controllers are organized around RESTful actions (index, show, new, create, etc.). ==In each action, the controller either renders an HTML template or redirects to another action.== For instance, a typical `create` action attempts to save a record and then uses `redirect_to` on success or `render :new` on failure. ==Controllers primarily produce HTML and rely on full-page reloads for updates. Partial updates (without a full reload) are possible but require custom JavaScript==: for example, using Rails UJS (remote: true on forms/links) to trigger AJAX and respond with .js.erb templates that manually manipulate the DOM. This approach works but adds complexity – developers must write JavaScript or jQuery to update specific page elements with server data.

### User Interaction Flow

Each interaction typically unloads the current page and loads a new one. There is a clear **context switch**: navigating to a form page or reloading the list page after submission. Users experience a full-page refresh for most actions (unless the app explicitly uses AJAX for some parts). Keeping client-side state (like scroll position or form data) requires extra work because a full reload resets the page state. ==As a result, classic Rails apps sometimes felt less “app-like” in their responsiveness, prompting some teams to introduce front-end JS frameworks to avoid frequent reloads.==

In summary, a classic Rails app delivers HTML page-by-page. Views are full pages or static partials, and controllers don’t typically differentiate between partial or full updates – they render HTML for the entire page. Any richer interactivity (modal forms, inline updates, etc.) requires either full page transitions or custom JavaScript.

## Structuring Views: Traditional vs. Turbo

==In a traditional Rails application, views are primarily composed of full HTML pages rendered on each server request.== These views leverage layouts, such as application.html.erb, to provide a consistent structure across pages, and partials (e.g., _header.html.erb) to encapsulate reusable components. ==Each action in a controller typically results in a full page reload, with the browser receiving and rendering the entire HTML document.== This approach aligns with Rails' convention of server-side rendering, where the server generates complete HTML for each user interaction.

In contrast, a Rails app heavily based on ==Turbo introduces significant modifications to view structuring, primarily through the use of Turbo Frames and Turbo Streams==. 

Turbo Frames allow developers to define specific sections of a page that can be updated independently, encapsulated within `<turbo-frame>` tags. For instance, a form or a list of items can be wrapped in a frame, enabling the browser to update only that section without reloading the entire page. This is particularly evident in scenarios like rendering forms on a `Quotes#show` page, where Turbo Frames facilitate frame swapping by matching id attributes, such as `new_line_item_date`.

Turbo Streams, on the other hand, enable dynamic updates to the Document Object Model (DOM) by sending small HTML fragments from the server. These streams support nine actions, including append, prepend, replace, update, remove, before, after, morph, and refresh, allowing for precise manipulation of page content. For example, inserting a new LineItemDate into an ordered list on a Quotes#show page can be achieved using `turbo_stream.after` or `turbo_stream.prepend`, maintaining order without a full page reload. ==This approach leverages existing partial templates for both initial rendering and dynamic updates==, reducing redundancy and enhancing maintainability.

The integration of Turbo Frames and Streams means views are no longer static representations but are designed to support partial updates, aligning with modern web development practices that prioritize user experience and performance. This shift is supported by the turbo-rails gem, which facilitates seamless integration with Rails, ensuring views can handle both initial renders and dynamic updates efficiently.

## Structuring Controllers: Traditional vs. Turbo

In a typical Rails application, controllers are responsible for handling HTTP requests, processing data, and rendering responses, typically full views or redirects. They use methods like `render` or `redirect_to` to respond to requests, often within a `respond_to` block to handle different formats such as HTML or JSON. For example, a `create` action might render a view like `new.html.erb` if validation fails, resulting in a full page reload.

In a Turbo-based Rails app, controllers must adapt to handle Turbo Stream responses, which are essential for dynamic updates. This involves using `respond_to` with `format.turbo_stream` for actions like `create`, `update`, and `destroy`. For instance, a `create` action might include:

```ruby
respond_to do |format| 
    format.turbo_stream { flash.now[:notice] = "Date was successfully created." } 
end
```

This allows the controller ==to render Turbo Stream responses==, sending HTML fragments that Turbo can use to update specific parts of the page, such as appending a new item to a list. ==This is a departure from traditional Rails, where responses are typically full HTML pages.== Additionally, Turbo integration often involves leveraging Action Cable for real-time updates, enabling features like WebSocket-based broadcasts, which require controllers to interact with Active Job for asynchronous rendering.

The controller structure thus shifts to accommodate these partial updates, requiring developers to design actions that can respond in multiple formats, including Turbo Streams, while maintaining compatibility with traditional HTML responses. This dual-mode operation enhances flexibility but also introduces complexity, as controllers must handle both full-page renders and partial updates seamlessly.

## Comparative Table

To summarize the differences, the following table provides a structured comparison:

| Aspect          | Typical Rails App                                            | Turbo Rails App                                              |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Views**       | Full HTML pages, layouts, and partials; full page reloads on each request. | Includes `<turbo-frame>` for updatable sections and `<turbo-stream>` for dynamic DOM updates, reducing full reloads. |
| **Controllers** | Render full views or redirect; use respond_to for HTML/JSON formats. | Handle `turbo_stream` responses for partial updates; integrate with Action Cable for real-time features. |

 