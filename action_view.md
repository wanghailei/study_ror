# Action View

Action View encapsulates all the functionality needed to render templates, most commonly generating HTML, XML, or JavaScript back to the user.

## Templates

The `render` method expects to find templates in the `app/views` directory of the current application.

Within this directory, the convention is to have a separate subdirectory for the views of each controller. Each subdirectory typically contains templates named after the actions in the corresponding controller.

Templates contain a mixture of fixed text and code. The code in the template adds dynamic content to the response. That code runs in an environment that gives it access to the information set up by the controller:



The path to the base directory of the templates is stored in the attribute `base_path`.

Jbuilder templates generate JSON responses.

In an ERB template, the ``debug`` method can be used to display the contents of the session, the details of the parameters, and the current response.

### Expose one instance variable per Action

==Rails uses instance variables to share data between controllers and views.== ==All instance variables of the controller are also available in the template.== This is how actions communicate data to the templates.

The way Rails makes data from controllers available to views is ==by copying the *instance variables of the controller* into the view code as *instance variables with the same name*==.

The way to get the most of Rails’ design without creating a problem is to adopt two conventions:

- ==Expose exactly one instance variable from any given action, ideally named for the resource or resources being manipulated by the route to that action.== For example, the `widget` `show` page should only expose `@widget`. ==The key situation to avoid is exposing multiple instance variables that collectively represent the resource rather than creating a single instance variable (and perhaps a new model class) to do so.==

- There are there exceptions: when a view requires access to reference data, like a list of country codes, when the view needs access to global context, like the currently logged-in user, or when there is UI state that is persisted across page refreshes, such as the currently selected tab in a tab navigation control.

## Layout

For a complex modular application, the best practice is to move beyond the single default `yield` and treat your primary layout as a "chassis" that defines all structural slots.

Use **one canonical layout** for global shell and **named regions** for module-specific stuff. Avoid fragmenting into many similar top-level layouts unless the shell truly differs.

### The "Chassis" Layout (`application.html.erb`)

**One Layout to Rule Them All:** For a standard application, strive to use a single `application.html.erb` that defines *all* possible structural regions using **named `yield`s**. Avoid nested layouts unless a module has a *fundamentally* different HTML structure (e.g., a full-screen "kiosk" mode vs. the standard app).

Think of your `application.html.erb` as the single source of truth for the *entire* UI structure. It should define *all* named modules (header, footer, sidebar, main) as distinct regions. ==The key is to use **named `yield`s** for every region that *can* change==, and to provide a default block for those regions. The main content area remains the default, unnamed `yield`.

Here is a robust `application.html.erb` example:

```erb
<!DOCTYPE html>
<html>
	<head>
		<%# This slot is for the <title> tag, set by each view %>
		<title><%= yield(:page_title).presence || "Default App Title" %></title>
		<%= csrf_meta_tags %>
		<%= csp_meta_tag %>
		<%= stylesheet_link_tag "application" %>
	</head>
	<body>
		<header class="app-header">
			<%# Headers are often static, but you could use a yield here too %>
			<%= render 'layouts/header' %>
		</header>
		<div class="app-container">
			<aside class="app-sidebar">
				<%# This is the slot for your main menu module %>
				<%= yield :menu do %>
					<%# This is the default content if no view provides a :menu %>
					<%= render 'layouts/default_menu' %>
				<% end %>
				<%# This is an optional, additional slot for context-specific widgets %>
				<%= yield :sidebar_extra %>
			</aside>
			<main class="app-main">
				<%# This is the primary, default yield for the view's main content %>
				<%= yield %>
			</main>
		</div>
		<footer class="app-footer">
			<%= render 'layouts/footer' %>
		</footer>
		<%# A special slot for page-specific javascript at the end of <body> %>
		<%= yield :page_scripts %>
	</body>
</html>
```

**`yield`** renders the unnamed “main” block of a layout (the view the controller action resolves to).

**Named yields** — `yield(:name)` are *slots* you can fill from views/partials via `content_for :name do ... end` or provide `:name, "..."`.

### Filling Slots with `content_for`

Now, your individual view files (e.g., `app/views/projects/show.html.erb`) become much cleaner. They have two responsibilities:

1. Provide the main content (which goes to the default `yield`).
2. ==Use `content_for` to "fill" any of the named slots defined in the layout.==

If a view provides `content_for :menu`, it *replaces* the default menu. If it doesn't, the layout's default is rendered. This gives you precise, per-view control over the entire page structure.

Here is a `show.html.erb` example:

```erb
<%# 1. Fill the :page_title slot %>
<% content_for :page_title, "Viewing Project: #{@project.name}" %>

<%# 2. Fill the :menu slot with a context-specific menu %>
<% content_for :menu do %>
	<%= render 'projects/context_menu', project: @project %>
<% end %>

<%# 3. Fill the :sidebar_extra slot with related items %>
<% content_for :sidebar_extra do %>
	<h3>Related Tasks</h3>
	<%= render @project.tasks %>
<% end %>

<%# 4. This is the main content, which goes to the default `yield` %>
<h1><%= @project.name %></h1>
<p>
	<%= @project.description %>
</p>
```

**`content_for`** appends by default. Call `content_for :name, flush: true` to replace prior content.

### Best Practices Summarized

1. 
2. **`yield` with Defaults:** Always provide a default block for your named `yield`s (e.g., `<%= yield :menu do ... end %>`). This ensures the layout is never "broken" and handles the default state gracefully.
3. **`content_for` is the "Provider":** ==Treat `content_for` as the way views *provide* content to the layout.== This is a very clear separation of concerns. ==The layout defines the *structure*, the view provides the *content*.==
4. **Use Partials for Content:** Inside both your `yield` blocks and your `content_for` blocks, you should almost always be rendering partials (e.g., `render 'layouts/default_menu'`). This keeps the layout and view files clean and focused on structure, not implementation details.

This pattern creates a system that is simple, clear, and "in-order," as the layout file becomes a perfect, readable map of your entire UI structure.

## Partial



***

