# Turbo Frame Strategy

## Strategy for setting mulitple turbo_frames for an app

> For BOS's shell structure, what is a future-proof structure of setting turbo_frames? Set a) a turbo_frame for each of the 12 cells, or b) a frame for #main and #side. For example, when a link in the menu is clicked, say Products Catalogue, with a) main-head, main-neck, main-body and main-foot each needs change; with b) change once. But a) seems more flexible, e.g., if there is a filter bar in main-neck and a data table in main-body. When a operated filter, only the data-table needs update.

### Strategy

ChatGPT Recommended **b**

Start with **two primary frames**: main-frame, side-frame. Add **small ==nested frames==** only in `main-body` for hotspots (e.g., `products_table_frame`). Use **Turbo Streams or small Stimulus setters** for head/neck/foot text/counters. Avoid “one frame per cell” unless you truly need independent lifecycle + history. This gives BOS a durable, performant shell that scales with your enterprise UI without getting tangled in frame spaghetti.

#### Why this is future-proof:

- **Stable shell:** headers/footers/toolbars don’t get torn down, so Carbon WCs + Stimulus state stay alive.
- **Simple navigation:** menu clicks target **one** frame (main-frame), avoiding “content missing” headaches and layout drift.
- **Granular updates where it matters:** for filter/table interactions, **nest one more frame inside** **main-body** (e.g., products_table_frame) so only the table updates.
- **Streams for cross-cutting UI:** use Turbo Streams to update badges/flash/breadcrumbs (or a tiny Stimulus setter like you did for the side head title).

## Suggested structure

#### Layout (permanent shell + 2 primary frames)

\#shell is **permanent**. Only main-frame and side-frame are primary, page-changing frames. Head/neck/foot stay static (fast, stable, no re-init churn).

```erb
<div id="shell" data-controller="shell" data-turbo-permanent>
    <nav id="menu">
        <header id="menu-head"><%= render "shell/menu_head" %></header>
        <div id="menu-neck"><%= render "shell/menu_neck" %></div>
        <div id="menu-body"><%= render "shell/menu_body" %></div>
        <footer id="menu-foot"><%= render "shell/menu_foot" %></footer>
    </nav>
    <main id="main">
        <header id="main-head"><%= render "shell/main_head" %></header>
        <div id="main-neck"><%= render "shell/main_neck" %></div>
        <div id="main-body">
            <%= turbo_frame_tag "main-frame" do %><%= yield %><% end %>
        </div>
        <footer id="main-foot"><%= render "shell/main_foot" %></footer>
    </main>
    <aside id="side">
        <header id="side-head"><%= render "shell/side_head" %></header>
        <div id="side-body">
            <%= turbo_frame_tag "side-frame" %>
        </div>
        <div id="side-neck"><%= render "shell/main_neck" %></div>
        <footer id="side-foot"><%= render "shell/side_foot" %></footer>
    </aside>
</div>
```

#### A page inside main (adds 1 inner frame just for the table)

Menu click loads this whole view **into** **main-frame**. Filter submit targets **only** `products_table_frame`, updating the table without touching head/neck. You don’t need separate frames for main-head, main-neck, or main-foot unless they truly change independently.

```erb
<%# app/views/products/index.html.erb %>
<div class="filters">
    <%= form_with url: products_path, method: :get, data: { turbo_frame: "products_table_frame" } do |f| %>
        <%= f.label :q, "Filter" %>
        <%= f.text_field :q %>
        <%= f.submit "Apply" %>
    <% end %>
</div>
<%= turbo_frame_tag "products_table_frame" do %>
	<%= render "products/table", products: @products %>
<% end %>
```

#### When would you wrap `filters` in a frame?

Only if the *filter UI itself* can change dynamically. For example:

- You have a “Filter by Category” dropdown whose options depend on the selected department.

	Then that `<div class="filters">` might itself become a frame (`filters_frame`).

- Or you want to reset the filters entirely after some operation.

```erb
<%= turbo_frame_tag "filters_frame" do %>
	<%= form_with ... data: { turbo_frame: "products_table_frame" } %>
<% end %>

<%= turbo_frame_tag "products_table_frame" do %>
<% end %>
```

#### Controller pattern

No `format.turbo_stream` needed for the GET filter case; the nested frame returns HTML with the matching id.

```ruby
# app/controllers/products_controller.rb
def index
    @products = Product.search(params[:q])

    # Normal HTML render goes inside main-frame (from the layout)
    # The nested table frame is inside this view, so Turbo finds it automatically.
end
```

#### When to add more frames

- **Yes:** a big live region inside main-body (table, chart, paginated list) → 1 nested frame for that region.
- **Sometimes:** a *second* hotspot (e.g., a live preview panel alongside the table) → add a second nested frame in main-body.
- **No by default:** separate frames for main-head, main-neck, main-foot. Use Turbo Streams (or a tiny Stimulus setter) to tweak those when needed.

### **Why not “a frame per cell” (the 12-frame layout)**

- Hard to keep responses consistent (easy to hit “Content missing”).
- Cross-cell coordination becomes brittle (what updates first? who owns history?).
- Carbon WCs get re-initialized too often → lost state, flicker.
- Debuggability drops as frame count rises.

#### Menu wiring (Carbon → main-frame)

Your current Stimulus bridge is perfect. Keep it simple:

```js
navigateMain(event) {
	const href = event.detail?.href
		|| event.detail?.item?.getAttribute('href')
		|| event.composedPath?.().find(el => el?.tagName === 'CDS-TREE-NODE')?.getAttribute('href')
	if (!href) return
	Turbo.visit(href, { frame: "main-frame" })
}
```

#### Updating head/neck when the page changes

Two clean options:

1. **Include them inside** **main-frame** only on pages that truly own them (rare in your shell).
2. **Keep them static** and update text via:
	- Turbo Streams that update specific targets (e.g., #breadcrumb, #page_title), or
	- your existing “server-driven title” Stimulus pattern (drop an invisible `<div data-controller="set-title" data-set-title-text-value="Products">` inside the frame content).

### Nested Turbo Frame

Nested frames are one of the most powerful (and under-documented) ideas in Turbo. They give you the same composability you’d get from a modern reactive framework, but with zero JS reactivity overhead.

#### The nested-frame philosophy

Think of Turbo frames as **dynamic fragments of your UI hierarchy**. Each frame has its own:

- lifecycle (loads, replaces, caches independently),
- navigation scope (links/forms target a parent or child frame),
- and history (back/forward remember its state).

So when you *nest* them:

- The outer frame provides structural layout.
- The inner frame provides local reactivity (hot zones).

Result: You get a *progressive UI tree*, where parts update individually but still share a unified page.

#### The typical “nested trio” pattern

```erb
<%= turbo_frame_tag "main-frame" do %>
  <div id="filters">
    <%= form_with url: products_path, method: :get, data: { turbo_frame: "products_table_frame" } do |f| %>
      <%= f.text_field :q, placeholder: "Search…" %>
      <%= f.submit "Filter" %>
    <% end %>
  </div>
  <%= turbo_frame_tag "products_table_frame" do %>
    <%= render "products/table", products: @products %>
  <% end %>
<% end %>
```

**Outer main-frame:** responds to navigation (menu, section switch).

**Inner products_table_frame:** updates when the filter form submits.

All the logic remains server-driven — Turbo just swaps HTML fragments.

#### Deeper nesting examples

##### Multi-region dashboard

```
<%= turbo_frame_tag "dashboard-frame" do %>
  <%= turbo_frame_tag "metrics_frame" do %>
    <%= render "dashboard/metrics", metrics: @metrics %>
  <% end %>

  <%= turbo_frame_tag "tasks_frame" do %>
    <%= render "dashboard/tasks", tasks: @tasks %>
  <% end %>

  <%= turbo_frame_tag "activity_frame" do %>
    <%= render "dashboard/activity", events: @events %>
  <% end %>
<% end %>
```

Each subframe can stream updates independently.

You could even send Turbo Stream responses to only tasks_frame when a task is marked complete.

#### What makes nested frames safe

Turbo frames **don’t interfere** with one another as long as IDs are unique:

- A navigation inside products_table_frame replaces only its own contents.
- The parent frame remains intact.
- If a nested frame needs to update its parent, you can target that parent frame ID in the Turbo Stream response.

**Example:**

```
<!-- app/views/products/create.turbo_stream.erb -->
<turbo-stream action="replace" target="products_table_frame">
  <template>
    <%= render "products/table", products: @products %>
  </template>
</turbo-stream>

<turbo-stream action="update" target="notice">
  <template>
    Product created successfully.
  </template>
</turbo-stream>
```

#### Why it’s perfect for BOS

BOS’s layout is inherently modular — menu, main, side, head, neck, body, foot.

Nested frames let you mirror that hierarchy **without over-fragmenting**.

Here’s the ideal evolution path:

| **Layer** | **Example frame**                       | **Purpose**                    |
| --------- | --------------------------------------- | ------------------------------ |
| Global    | main-frame, side-frame                  | Section switching              |
| Section   | products_table_frame, orders_list_frame | Filtered tables, local updates |
| Inline    | edit_row_frame                          | Inline edit/preview            |

So instead of **12 fixed frames**, you’ll have **2 stable + N dynamic** (created where useful).

This keeps BOS fast, clean, and visually stable — the user never feels a “page reload.”

#### Debugging tip

When nesting, add a little border in dev mode:

```
turbo-frame { outline: 1px dotted rgba(0,0,0,0.2); }
```

You’ll literally *see* the hierarchy, and it’s enlightening.

You’ll start thinking of your UI as composable “frame islands” — a mental model that fits enterprise UI very naturally.

Would you like me to draw you a **visual map of BOS’s future Turbo frame hierarchy** (main, side, nested inside main-body for filters, tables, modals, etc.)?

It would make a perfect blueprint for how your Rails views should be structured going forward.