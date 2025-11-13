# Turbo

==% 有 Turbo，可以讓 UI 局部刷新，且不用寫 Javascript。20231129 %==

Turbo speeds up Rails applications, reduces the amount of JavaScript we have to write, and makes it easy to work with real-time features.

1\. All clicks on links and form submissions are now AJAX requests, thanks to Turbo Drive. We get this benefit for free as it does not require any work; we only have to import the library.

2\. It is now effortless, with just a few lines of code to build dynamic applications by slicing pages in different pieces with Turbo Frames. We develop our CRUD controllers just like we did before, and just by adding a few lines of code, we can replace or lazy-load independent parts of the page!

3\. It becomes trivial to add real-time features with the help of Turbo Streams. Want to add real-time notifications to your application, build a real-time multiplayer game, or a real-time bug monitoring system? The real-time part is just a few lines of code with Turbo Stream!

- *Turbo Drive* accelerates links and form submissions by negating the need for full page reloads.
- **Turbo Frames** decompose pages into independent contexts, which scope navigation and can be lazily loaded.
- *Turbo Streams* deliver page changes over WebSocket, SSE or in response to form submissions using just HTML and a set of CRUD-like actions.

Turbo focuses squarely on just updating the DOM, and then assumes you’ll connect any additional behavior using [Stimulus](https://stimulus.hotwired.dev/) actions and lifecycle callbacks.

## WHL

### One-way vs two-ways

From user interaction perspective, I think ==Turbo Stream is just about output== - responding or showing changes; yet ==Turbo Frame does both input and output==.

#### Turbo Frame: Full Interaction Unit (Input + Output)

- **Input**: Users click a link or submit a form *inside* a `<turbo-frame>`.
- **Output**: The server returns an HTML fragment (without full layout), and Turbo replaces just that frame.

**It’s a self-contained interaction cycle.** ==You can think of it like a **mini web browser inside the page**: navigation, form submission, back-forward caching — all scoped to that frame.==

#### Turbo Stream: One-Way DOM Update output only

- **No direct user input** involved. It doesn’t initiate anything — it just *reacts*.
- **Triggered indirectly** (e.g., form submitted, record saved).
- The server responds with **actions** like append, replace, remove, etc., and Turbo just *executes those actions* on the DOM.

#### Mental Model Analogy

| **Concept**   | **Turbo Frame**                     | **Turbo Stream**               |
| ------------- | ----------------------------------- | ------------------------------ |
| Comparable to | `<iframe>` with navigation          | JavaScript DOM patching        |
| Flow          | Input → Server → Output             | Trigger → Server → Output      |
| Role          | Mini page (component-level UI flow) | UI response (state reflection) |

### 3 Purposes

==Turbo Drive is for layout; Turbo Frame is for interactive parts; and Turbo Stream is for displaying parts. I think they can be aliased as Turbo Navigation, Turbo Interaction, and Turbo Display. ;-D==

#### Turbo Drive → Layout transitions

- Handles *whole-page* navigations elegantly.
- Optimises layout-level changes: switching sections, jumping across pages.
- Keeps `<head>`, JS, CSS alive = better performance + smoother UX.
- Great for top-level nav links and section switches

#### Turbo Frame → Interactive fragments

- Wraps self-contained UI areas (modals, sidebars, inline forms)
- Enables scoped navigation and inline form handling
- Keeps user context without reloading the whole page

#### Turbo Stream → Reactive updates elsewhere**

- Responds to model changes with append/replace/remove
- Used after server-side actions: append, replace, remove — anywhere.
- Often used *with* or *after* form submissions, broadcasts, or background jobs.

### 🧠 Summary Mental Model

| Layer              | Turbo Feature | Role                                |
| ------------------ | ------------- | ----------------------------------- |
| App-level Layout   | Turbo Drive   |                                     |
| Component-level UI | Turbo Frame   | In-place interaction                |
| System Feedback    | Turbo Stream  | Reflect model changes across the UI |

## Turbo Drive: Navigating

Turbo is a continuation of the ideas from the previous [Turbolinks](https://github.com/turbolinks/turbolinks) framework, and the heart of that past approach lives on as Turbo Drive. 

==When installed, Turbo automatically intercepts all clicks on `<a href>` links to the same domain.== When you click an eligible link, Turbo prevents the browser from following it. Instead, Turbo changes the browser’s URL using the History API, requests the new page using `fetch`, and then renders the HTML response.

==During rendering, Turbo Drive replaces the contents of the `<body>` element and merges the contents of the `<head>` element. The JavaScript window and document objects, and the `<html>` element, persist from one rendering to the next.==

## Turbo Frames: Decompose complex pages

Most web applications present pages that contain several independent segments. With Turbo Frames, you can place those independent segments inside frame elements that can scope their navigation and be lazily loaded. 

### Scoped navigation

==Scoped navigation means all interaction within a frame, like clicking links or submitting forms, happens within that frame, keeping the rest of the page from changing or reloading.==

To wrap an independent segment in its own navigation context, enclose it in a `<turbo-frame>` tag. For example:

```html
<turbo-frame id="new_message">
    <form action="/messages" method="post">
    </form>
</turbo-frame>
```

When you submit the form above, Turbo extracts the matching `<turbo-frame id="new_message">` element from the redirected HTML response and swaps its content into the existing `new_message` frame element. The rest of the page stays just as it was.

### Defer loading contents

Frames can also defer loading their contents in addition to scoping navigation. To defer loading a frame, add a `src` attribute whose value is the URL to be automatically loaded. As with scoped navigation, Turbo finds and extracts the matching frame from the resulting response and swaps its content into place:

```html
<turbo-frame id="messages" src="/messages">
	<p>This message will be replaced by the response from /messages.</p>
</turbo-frame>
```

This may sound a lot like old-school frames, or even `<iframe>`s, but Turbo Frames are part of the same DOM, so there’s none of the weirdness or compromises associated with “real” frames. Turbo Frames are styled by the same CSS, part of the same JavaScript context, and are not placed under any additional content security restrictions.

In addition to turning your segments into independent contexts, Turbo Frames affords:

### Efficient caching

In the discussion page example above, the related topics sidebar needs to expire whenever a new related topic appears, but the list of messages in the center does not. When everything is just one page, the whole cache expires as soon as any of the individual segments do. ==With turbo  frames, each segment is cached independently==, so you get longer-lived caches with fewer dependent keys.

### Parallelized execution

Each defer-loaded turbo frame is generated by its own HTTP request/response, which means it can be handled by a separate process. ==This allows for parallelized execution without having to manually manage the process.== A complicated composite page that takes 400ms to complete end-to-end can be broken up with frames where the initial request might only take 50ms, and each of three defer-loaded frames each take 50ms. Now the whole page is done in 100ms because the three frames each taking 50ms run concurrently rather than sequentially.

### `turbo_frame_tag` helper

This gem provides a `turbo_frame_tag` helper to create those frames.

```erb
<%# app/views/todos/show.html.erb %>
<%= turbo_frame_tag @todo do %>
    <p><%= @todo.description %></p>
    <%= link_to 'Edit this todo', edit_todo_path(@todo) %>
<% end %>

<%# app/views/todos/edit.html.erb %>
<%= turbo_frame_tag @todo do %>
    <%= render "form" %>
    <%= link_to 'Cancel', todo_path(@todo) %>
<% end %>
```



When the user clicks on the `Edit this todo` link, as a direct response to this user interaction, the turbo frame will be automatically replaced with the one in the `edit.html.erb` page.

## Turbo Streams: Deliver live page changes

Partial page updates that are **delivered asynchronously over a web socket connection** is the hallmark of modern, reactive web applications. Making partial page changes in response to asynchronous actions is how we make the application feel alive. With Turbo Streams, you can get all of that modern goodness using the existing server-side HTML you're already rendering to deliver the first page load. 

==While Turbo Frames give us such updates in response to direct interactions within a single frame, Turbo Streams let us change any part of the page in response to updates== sent over a WebSocket connection, SSE or other transport.

Turbo Streams introduces a `<turbo-stream>` element with nine basic actions: `append`, `prepend`, `replace`, `update`, `remove`, `before`, `after`, `morph`, and `refresh`. With these actions, along with the `target` attribute specifying the ID of the element you want to operate on, you can encode all the mutations needed to refresh the page. Simply include the HTML you’re interested in inserting or replacing in a [template tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template) and Turbo does the rest:

```html
<turbo-stream action="append" target="messages">
    <template>
    	<div id="message_1">My new message!</div>
    </template>
</turbo-stream>
```

This stream element will take the `div` with the new message and append it to the container with the ID `messages`. It’s just as simple to replace an existing element:

```html
<turbo-stream action="replace" target="message_1">
    <template>
    	<div id="message_1">This changes the existing message!</div>
    </template>
</turbo-stream>
```

You can even combine several stream elements in a single stream message. 

With a set of simple CRUD container tags, you can send HTML fragments over the web socket (or in response to direct interactions), and see the page change in response to new data. Again, **no need to construct an entirely separate API**, **no need to wrangle JSON**, **no need to reimplement the HTML construction in JavaScript**.

With this Rails integration, you can create these asynchronous updates directly in response to your model changes. ==Turbo uses Active Jobs to provide asynchronous partial rendering and Action Cable to deliver those updates to subscribers.==

This gem provides a `turbo_stream_from` helper to create a turbo stream.

```erb
<%# app/views/todos/show.html.erb %>
<%= turbo_stream_from dom_id(@todo) %>
```

Here’s a compact **Turbo (Drive/Frames/Streams) attribute cheat sheet** you can keep open while you build BOS. It’s grouped by where each attribute is valid.

## Turbo attributes

### `<turbo-frame>` element's core attributes

These are **native to the Turbo library** (defined in turbo.es2017.js), and work in all Rails 7/8 apps.

#### `id`

Uniquely identifies the frame and becomes its navigation target. Required.

Values: string

```html
<turbo-frame id="main-frame">
```

#### `src`

Automatically loads remote content into the frame, like an `iframe.

Values: URL

```html
<turbo-frame id="details" src="/products/1">
```

#### `target`

Sets default target for navigation triggered by **the links or forms inside this frame**.

Values: a frame's id or `_top` which means navigate the whole page.

```html    
<turbo-frame id="details" target="_top">
```

#### `disabled`

Disables Turbo behavior for this frame. Frame becomes static. Prevents any Turbo navigation or replacement for this frame.

Values: boolean  

```html
<turbo-frame id="details" disabled>
```

#### `loading`

Controls when Turbo loads a remote src. 

Values: `eager` (default), or `lazy` which means waits until visible.  

```html
<turbo-frame id="comments" src="/comments" loading="lazy">
    
<turbo-frame id="comments" src="/posts/1/comments" loading="lazy">Loading comments…</turbo-frame>
```

#### `data-turbo-cache="false"`

Disables Turbo’s cache for a specific frame or page.

Values: boolean

```html
<turbo-frame id="details" data-turbo-cache="false">
```

#### `data-turbo-temporary`

Marks a frame whose contents may be replaced without caching. Rarely used.

Values: boolean

```html
<turbo-frame id="preview" data-turbo-temporary>
```

### Automatically managed attributes

Turbo adds/removes these dynamically. Managed by Turbo JS.

#### `busy`

Added automatically while Turbo is fetching remote content.  

```html
<turbo-frame id="details" busy>
```

#### `complete`

Added when frame finishes loading (rare).

#### `reloading`

Indicates frame is being re-rendered via Turbo navigation.

#### `aria-busy`

Often mirrored from busy for accessibility.

### Link or form attributes (tell Turbo where/how to navigate)

These don’t belong to the `<turbo-frame>` tag itself, but control how Turbo Drive interacts with it.

#### `data-turbo`

Enables or disables Turbo behavior. 

Applies to any element  

Values: `true` | `false`

```erb
<form data-turbo="false">
```

#### ==`data-turbo-frame`==

Specifies which frame the navigation should target.

Values: frame id \| `_top`

```erb
<a href="/products/1" data-turbo-frame="main-frame">
```

#### `data-turbo-preserve-scroll`

Keep scroll position or prevent scroll reset after navigation. Applies to `<a>` or `<form>`. (boolean)

```erb
<a href="/feed" data-turbo-preserve-scroll>
```

#### `data-turbo-action`

Controls how Turbo updates history/scroll. 
Applies to `<a>` or `<form>`.

Values: `advance` or `replace`.

```erb
<a href="/products" data-turbo-action="replace">
```

#### `data-turbo-method`

Send non-GET over a link (Rails UJS replacement). Applies to `<a>`.

Values: post \| patch \| put \| delete  

### Lifecycle / progressive enhancement data attributes

These are advanced attributes used by the Turbo JS runtime.

#### `data-turbo-permanent`

Prevents element replacement between page loads (keeps same DOM node). Used for menus, shells, etc.

Applies to any element.

```html
<div id="shell" data-turbo-permanent>
<!-- Permanent shell layout -->
```

#### `data-turbo-track`

Tells Turbo to reload the page when asset changes.

Applies to `<link>` or `<script>`.

Values: `reload`

```html
<link rel="stylesheet" href="/app.css" data-turbo-track="reload">
```

#### `data-turbo-preload`

Used with `<link rel="preload">` for early Turbo resource loading.

```html
<link rel="modulepreload" href="/turbo" data-turbo-preload>
```

### Useful JS hooks (programmatic navigation)

Navigate whole page (Drive):

```js
Turbo.visit("/products")
```

Navigate a frame:

```js
Turbo.visit("/products", { frame: "main-body-frame" })
```

Set a frame source directly:

```js
document.getElementById("main-body-frame")?.setAttribute("src", "/products")
```

### Turbo Streams (for completeness)

| **Element**    | **Attribute**     | **Values**                                        | **What it does**                   |
| -------------- | ----------------- | ------------------------------------------------- | ---------------------------------- |
| <turbo-stream> | action            | append prepend replace update remove before after | What mutation to perform           |
| <turbo-stream> | target \| targets | id \| CSS selector                                | Where to apply the action          |
| <template>     | —                 | —                                                 | The HTML payload to insert/replace |
