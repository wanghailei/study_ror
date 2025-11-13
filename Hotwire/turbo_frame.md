# Decompose with Turbo Frames

Turbo Frames allow predefined parts of a page to be updated on request. Any links and forms inside a frame are captured, and the frame contents automatically update after receiving a response. Regardless of whether the server provides a full document, or just a fragment containing an updated version of the requested frame, only that particular frame will be extracted from the response to replace the existing content.

==Frames are created by wrapping a segment of the page in a `<turbo-frame>` element.== Each element must have a unique ID, which is used to match the content being replaced when requesting new pages from the server. A single page can have multiple frames, each establishing their own context:

```html
<body>
    <div id="navigation">Links targeting the entire page</div>
    <turbo-frame id="message_1">
        <h1>My message title</h1>
        <p>My message content</p>
        <a href="/messages/1/edit">Edit this message</a>
    </turbo-frame>
    <turbo-frame id="comments">
        <div id="comment_1">One comment</div>
        <div id="comment_2">Two comments</div>
    	<form action="/messages/comments">...</form>
    </turbo-frame>
</body>
```

This page has two frames: One to display the message itself, with a link to edit it. One to list all the comments, with a form to add another. Each create their own context for navigation, capturing both links and submitting forms.

When the link to edit the message is clicked, the response provided by `/messages/1/edit` has its `<turbo-frame id="message_1">` segment extracted, and the content replaces the frame from where the click originated. The edit response might look like this:

```html
<body>
    <h1>Editing message</h1>
    <turbo-frame id="message_1">
        <form action="/messages/1">
            <input name="message[name]" type="text" value="My message title">
            <textarea name="message[content]">My message content</textarea>
            <input type="submit">
        </form>
    </turbo-frame>
</body>
```

Notice how the `<h1>` isn’t inside the `<turbo-frame>`. This means it will remain unchanged when the form replaces the display of the message upon editing. Only content inside a matching `<turbo-frame>` is used when the frame is updated.

Thus your page can easily play dual purposes: Make edits in place within a frame or edits outside of a frame where the entire page is dedicated to the action.

Frames serve a specific purpose: to compartmentalize the content and navigation for a fragment of the document. Their presence has ramification on any `<a>` elements or `<form>` elements contained by their child content, and shouldn’t be introduced unnecessarily. ==Turbo Frames do not contribute support to the usage of Turbo Stream.== If your application utilizes `<turbo-frame>` elements for the sake of a `<turbo-stream>` element, change the `<turbo-frame>` into another built-in element.

## `<turbo-frame>` vs `<%= turbo_frame_tag %>`



| **Form**                 | **Origin**       | **Output**                    | **When to use**                                   |
| ------------------------ | ---------------- | ----------------------------- | ------------------------------------------------- |
| `<turbo-frame>`          | **HTML element** | Pure HTML                     | Used in **client-rendered** markup (static or JS) |
| `<%= turbo_frame_tag %>` | **Rails helper** | Renders <turbo-frame> in HTML | Used in **ERB templates** (server-rendered views) |

So:

```erb
<%= turbo_frame_tag "main-body-frame" do %>Hello<% end %>
```

is just a convenient Rails helper that produces:

```html
<turbo-frame id="main-body-frame">Hello</turbo-frame>
```

Both are functionally identical once rendered in the browser.

```erb
<%= turbo_frame_tag @product do %>
	…content…
<% end %>
```

Assuming `@product` is persisted via dom_id(`@product`), it will generate:

```html
<turbo-frame id="product_123">…</turbo-frame>
```

You may supply attributes:

```erb
<%= turbo_frame_tag "side-frame", src: new_product_path, target: "_top", class: "some-class" do %>
   …placeholder content…
<% end %>
```

generates:

```html
<turbo-frame id="side-frame" src="/products/new" target="_top" class="some-class">
  …placeholder content…
</turbo-frame>
```

==`<turbo-frame>` is a web component.== It’s part of the Turbo library (in `@hotwired/turbo`), not Rails. 

It defines a special DOM region that can:

- Load content dynamically (`src="/some/path"`).

- Be a target for frame navigations (`Turbo.visit(url, { frame: "..." })`).

- Be replaced when the server returns a matching frame in a response.

	

	==`<%= turbo_frame_tag %>` is just Rails sugar helper for that.== Rails provides the helper via the turbo-rails gem. It allows you to write Ruby code that produces the HTML version cleanly, including optional attributes and escaping.

The Rails helper adds convenience features. The helper automatically:

- escapes HTML-safe attributes (so no injection risk);
- eakes your code **consistent** with other Rails tag helpers (content_tag, etc.);
- works seamlessly with Turbo Streams and Turbo Frames responses.

So you almost always use the ERB helper in server-rendered views.

### Common usage patterns

#### a. Static frame region (layout or view)

```erb
<div id="main-body">
    <%= turbo_frame_tag "main-body-frame" do %>
	    <%= yield %>
    <% end %>
</div>
```

#### b. Frame-driven sub-page navigation

Link targeting a frame (native link).

```erb
<%= link_to "New Product", new_product_path, data: { turbo_frame: "side-frame" } %>
<%= link_to "Edit", edit_product_path(product), data: { turbo_frame: "side-frame" } %>
```

That causes Turbo to load /products/new and replace the content *inside* `<turbo-frame id="side-frame">`.

#### When to use which

| **Context**                           | **Use**                      |
| ------------------------------------- | ---------------------------- |
| Writing plain HTML or JS templates    | Use `<turbo-frame>`          |
| Writing ERB (server templates)        | Use `<%= turbo_frame_tag %>` |
| You need to interpolate Ruby (<%= %>) | Use the helper               |
| You’re debugging browser output       | Look for `<turbo-frame>`     |

#### Analogy

Think of it like this:

| **Rails helper**       | **What it produces** |
| ---------------------- | -------------------- |
| <%= form_with %>       | <form>               |
| <%= link_to %>         | <a>                  |
| <%= turbo_frame_tag %> | <turbo-frame>        |

They all follow the same pattern — Ruby helpers that generate HTML elements but give you all the benefits of Ruby context and escaping.



## `Turbo::Frames::FrameRequest`

### Overview

Turbo frame requests are requests made from within a turbo frame with the intention of replacing the content of just that frame, not the whole page. They are automatically tagged as such by the Turbo Frame JavaScript, which adds a `Turbo-Frame` header to the request.

When that header is detected by the controller, we substitute our own minimal layout in place of the application-supplied layout (since we’re only working on an in-page frame, thus can skip the weight of the layout). We use a minimal layout, rather than avoid the layout entirely, so that it’s still possible to render content into the `head`.

Accordingly, we ensure that the etag for the page is changed, such that a cache for a minimal-layout request isn’t served on a normal request and vice versa.

This is merely a rendering optimization. Everything would still work just fine if we rendered everything including the full layout. Turbo Frames knows how to fish out the relevant frame regardless.

The layout used is `turbo_rails/frame.html.erb`. If there’s a need to customize this layout, an application can supply its own (such as `app/views/layouts/turbo_rails/frame.html.erb`) which will be used instead.

This module is automatically included in `ActionController::Base`.

## Eager-Loading Frames

Frames don’t have to be populated when the page that contains them is loaded. ==If a `src` attribute is present on the `turbo-frame` tag, the referenced URL will automatically be loaded as soon as the tag appears on the page==:

```html
<body>
    <h1>Imbox</h1>
    <div id="emails"></div>
    <turbo-frame id="set_aside_tray" src="/emails/set_aside"></turbo-frame>
    <turbo-frame id="reply_later_tray" src="/emails/reply_later"></turbo-frame>
</body>
```

This page lists all the emails available in your imbox immediately upon loading the page, but then makes two subsequent requests to present small trays at the bottom of the page for emails that have been set aside or are waiting for a later reply. These trays are created out of separate HTTP requests made to the URLs referenced in the `src`.

In the example above, the trays start empty, but it’s also possible to populate the eager-loading frames with initial content, which is then overwritten when the content is fetched from the `src`:

```html
<turbo-frame id="set_aside_tray" src="/emails/set_aside">
	<img src="/icons/spinner.gif">
</turbo-frame>
```

Upon loading the imbox page, the set-aside tray is loaded from `/emails/set_aside`, and the response must contain a corresponding `<turbo-frame id="set_aside_tray">` element as in the original example:

```html
<body>
    <h1>Set Aside Emails</h1>
    <p>These are emails you've set aside</p>
    <turbo-frame id="set_aside_tray">
        <div id="emails">
            <div id="email_1">
	            <a href="/emails/1">My important email</a>
            </div>
        </div>
    </turbo-frame>
</body>
```

This page now works in both its minimized form, where only the `div` with the individual emails are loaded into the tray frame on the imbox page, but also as a direct destination where a header and a description is provided. Just like in the example with the edit message form.

Note that the `<turbo-frame>` on `/emails/set_aside` does not contain a `src` attribute. That attribute is only added to the frame that needs to lazily load the content, not to the rendered frame that provides the content.

During navigation, a Frame will set `[aria-busy="true"]` on the `<turbo-frame>` element when fetching the new contents. When the navigation completes, the Frame will remove the `[aria-busy]`attribute. When navigating the `<turbo-frame>` through a `<form>` submission, Turbo will toggle the Form’s `[aria-busy="true"]` attribute in tandem with the Frame’s.

After navigation finishes, a Frame will set the `[complete]` attribute on the `<turbo-frame>` element.

## Lazy-Loading Frames

Frames that aren’t visible when the page is first loaded can be marked with `loading="lazy"` such that they don’t start loading until they become visible. This works exactly like the `loading="lazy"` attribute on `img`. It’s a great way to delay loading of frames that sit inside `summary`/`detail` pairs or modals or anything else that starts out hidden and is then revealed.

## Cache Benefits to Loading Frames

==Turning page segments into frames can help make the page simpler to implement==, but an equally important reason for doing this is to improve cache dynamics. Complex pages with many segments are hard to cache efficiently, especially if they mix content shared by many with content specialized for an individual user. The more segments, the more dependent keys required for the cache look-up, the more frequently the cache will churn.

Frames are ideal for separating segments that change on different timescales and for different audiences. Sometimes it makes sense to turn the per-user element of a page into a frame, if the bulk of the rest of the page is then easily shared across all users. Other times, it makes sense to do the opposite, where a heavily personalized page turns the one shared segment into a frame to serve it from a shared cache.

While the overhead of fetching loading frames is generally very low, you should still be judicious in just how many you load, especially if these frames would create load-in jitter on the page. Frames are, however, essentially free if the content isn’t immediately visible upon loading the page. Either because they’re hidden behind modals or below the fold.

## Targeting Navigation Into or Out of a Frame

==By default, navigation within a frame will target just that frame.== This is true for both following links and submitting forms. But navigation can drive the entire page instead of the enclosing frame by setting the target to `_top`. Or it can drive another named frame by setting the target to the ID of that frame.

In the example with the set-aside tray, the links within the tray point to individual emails. You don’t want those links to look for frame tags that match the `set_aside_tray` ID. You want to navigate directly to that email. This is done by marking the tray frames with the `target` attribute:

```html
<body>
    <h1>Imbox</h1>
    ...
    <turbo-frame id="set_aside_tray" src="/emails/set_aside" target="_top">
    </turbo-frame>
</body>

<body>
    <h1>Set Aside Emails</h1>
    ...
    <turbo-frame id="set_aside_tray" target="_top">
    ...
    </turbo-frame>
</body>
```

Sometimes you want most links to operate within the frame context, but not others. This is also true of forms. You can add the `data-turbo-frame` attribute on non-frame elements to control this:

```html
<body>
    <turbo-frame id="message_1">
        <a href="/messages/1/edit">Edit this message (within the current frame)</a>
        <a href="/messages/1/permission" data-turbo-frame="_top">Change permissions (replace the whole page)</a>
    </turbo-frame>
    <form action="/messages/1/delete" data-turbo-frame="message_1">
        <a href="/messages/1/warning" data-turbo-frame="_self">Load warning within current frame</a>
        <input type="submit" value="Delete this message">
        (with a confirmation shown in a specific frame)
    </form>
</body>
```

## Promoting a Frame Navigation to a Page Visit

Navigating Frames provides applications with an opportunity to change part of the page’s contents while preserving the rest of the document’s state (for example, its current scroll position or focused element). There are times when we want changes to a Frame to also affect the browser’s history.

To promote a Frame navigation to a Visit, render the element with the `[data-turbo-action]` attribute. The attribute supports all [Visit](https://turbo.hotwired.dev/handbook/drive#page-navigation-basics) values, and can be declared on:

- the `<turbo-frame>` element
- any `<a>` elements that navigate the `<turbo-frame>`
- any `<form>` elements that navigate the `<turbo-frame>`
- any `<input type="submit">` or `<button>` elements contained within `<form>` elements that navigate the `<turbo-frame>`

For example, consider a Frame that renders a paginated list of articles and transforms navigations into “`advance`” Actions:

```html
<turbo-frame id="articles" data-turbo-action="advance">
	<a href="/articles?page=2" rel="next">Next page</a>
</turbo-frame>
```

Clicking the `<a rel="next">` element will set *both* the `<turbo-frame>` element’s `[src]` attribute *and* the browser’s path to `/articles?page=2`.

**Note:** when rendering the page after refreshing the browser, it is *the application’s* responsibility to render the *second* page of articles along with any other state derived from the URL path and search parameters.

## “Breaking out” from a Frame

In most cases, requests that originate from a `<turbo-frame>` are expected to fetch content for that frame (or for another part of the page, depending on the use of the `target` or `data-turbo-frame` attributes). This means the response should always contain the expected `<turbo-frame>` element. If a response is missing the `<turbo-frame>` element that Turbo expects, it’s considered an error; when it happens Turbo will write an informational message into the frame, and throw an exception.

In certain, specific cases, you might want the response to a `<turbo-frame>` request to be treated as a new, full-page navigation instead, effectively “breaking out” of the frame. The classic example of this is when a lost or expired session causes an application to redirect to a login page. In this case, it’s better for Turbo to display that login page rather than treat it as an error.

The simplest way to achieve this is to specify that the login page requires a full-page reload, by including the [`turbo-visit-control`](https://turbo.hotwired.dev/reference/attributes#meta-tags) meta tag:

```html
<head>
	<meta name="turbo-visit-control" content="reload">
</head>
```

If you’re using Turbo Rails, you can use the `turbo_page_requires_reload` helper to accomplish the same thing.

Pages that specify `turbo-visit-control` `reload` will always result in a full-page navigation, even if the request originated from inside a frame.

If your application needs to handle missing frames in some other way, you can intercept the[`turbo:frame-missing`](https://turbo.hotwired.dev/reference/events) event to, for example, transform the response or perform a visit to another location.

## [﹟](https://turbo.hotwired.dev/handbook/frames#anti-forgery-support-(csrf))Anti-Forgery Support (CSRF)

Turbo provides [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery) protection by checking the DOM for the existence of a `<meta>` tag with a `name` value of either `csrf-param` or `csrf-token`. For example:

```html
<meta name="csrf-token" content="[your-token]">
```

Upon form submissions, the token will be automatically added to the request’s headers as `X-CSRF-TOKEN`. Requests made with `data-turbo="false"` will skip adding the token to headers.

## [﹟](https://turbo.hotwired.dev/handbook/frames#custom-rendering)Custom Rendering

Turbo’s default `<turbo-frame>` rendering process replaces the contents of the requesting `<turbo-frame>` element with the contents of a matching `<turbo-frame>` element in the response. In practice, a `<turbo-frame>` element’s contents are rendered as if they operated on by (https://turbo.hotwired.dev/reference/streams#update) element. The underlying renderer extracts the contents of the `<turbo-frame>` in the response and uses them to replace the requesting `<turbo-frame>` element’s contents. The `<turbo-frame>` element itself remains unchanged, save for the [`[src\]`, `[busy]`, and `[complete]` attributes that Turbo Drive manages](https://turbo.hotwired.dev/reference/frames#html-attributes) throughout the stages of the element’s request-response lifecycle.

Applications can customize the `<turbo-frame>` rendering process by adding a `turbo:before-frame-render` event listener and overriding the `event.detail.render` property.

For example, you could merge the response `<turbo-frame>` element into the requesting `<turbo-frame>` element with [morphdom](https://github.com/patrick-steele-idem/morphdom):

```javascript
import morphdom from "morphdom"

addEventListener("turbo:before-frame-render", (event) => {
  event.detail.render = (currentElement, newElement) => {
    morphdom(currentElement, newElement, { childrenOnly: true })
  }
})
```

Since `turbo:before-frame-render` events bubble up the document, you can override one `<turbo-frame>` element’s rendering by attaching the event listener directly to the element, or override all `<turbo-frame>` elements’ rendering by attaching the listener to the `document`.

## [﹟](https://turbo.hotwired.dev/handbook/frames#pausing-rendering)Pausing Rendering

Applications can pause rendering and make additional preparations before continuing.

Listen for the `turbo:before-frame-render` event to be notified when rendering is about to start, and pause it using `event.preventDefault()`. Once the preparation is done continue rendering by calling `event.detail.resume()`.

An example use case is adding exit animation:

```javascript
document.addEventListener("turbo:before-frame-render", async (event) => {
  event.preventDefault()

  await animateOut()

  event.detail.resume()
})
```