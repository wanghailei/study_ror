# Turbo for BOS - products

A **pure** structure for products that matches your philosophy:

**`Controller`**:

- Only knows “what happened” (save success, validation failure, destroyed, etc.).
- Only does respond_to do |format| format.html / format.turbo_stream end.
- only data + respond_to do |format|
- No turbo_frame_request? in controllers
- No dom_id in controllers
- All UI wiring (which frame / which id) lives in *.turbo_stream.erb
- No UI ids, no frame names, no partial wiring.

**`*.turbo_stream.erb`**:

- Knows the DOM structure and frames.
- Decides which element (products-table, product-123, side-body-frame, side-neck) gets updated.
- Branches on @product.errors.any? to choose error vs success UI.

Below is a complete structure with comments in every file so Future-You remembers why each thing exists.



## The core mental model

> I kind of understand the relationship and roles. *.turbo_stream.erbs are for wiring, and *.html.erb are still the concrete building blocks.

Turbo Stream = “Wiring Layer”

HTML/Partials = “Building Blocks Layer”

## **1. Controller**

### **app/controllers/products_controller.rb**



```ruby
class ProductsController < ApplicationController
	before_action :set_product, only: [ :show, :edit, :update, :destroy ]
	before_action :load_categories, only: [ :new, :edit, :create, :update ]

	# ============================================================
	# index
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Load all products for listing.
	#   - HTML:
	#       Used for the initial full-page load (main + side shell).
	#   - TURBO_STREAM:
	#       Used when you filter / sort / search and want to
	#       update the products table without full-page reload.
	#
	# No UI wiring here:
	#   - Controller does NOT decide which frame to update.
	#   - index.turbo_stream.erb decides that.
	# ============================================================
	def index
		@products = Product.order(created_at: :desc)

		respond_to do |format|
			format.html              # renders app/views/products/index.html.erb
			format.turbo_stream      # renders app/views/products/index.turbo_stream.erb
		end
	end

	# ============================================================
	# show
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Present a single product.
	#   - HTML:
	#       Full-page show, if you ever navigate directly.
	#   - TURBO_STREAM:
	#       Used for BOS shell:
	#       "load this product into side-body-frame and update side-neck".
	#
	# Note:
	#   - No turbo_frame_request? here.
	#   - The turbo_stream view decides which frames to touch.
	# ============================================================
	def show
		respond_to do |format|
			format.html              # app/views/products/show.html.erb
			format.turbo_stream      # app/views/products/show.turbo_stream.erb
		end
	end

	# ============================================================
	# new
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Provide a blank product instance for form.
	#   - HTML:
	#       Full-page new (if ever needed).
	#   - TURBO_STREAM:
	#       Show the "new product" form in side-body-frame.
	#
	# Again:
	#   - Controller does not say "render this into frame X".
	#   - new.turbo_stream.erb handles that.
	# ============================================================
	def new
		@product = Product.new

		respond_to do |format|
			format.html              # app/views/products/new.html.erb (optional)
			format.turbo_stream      # app/views/products/new.turbo_stream.erb
		end
	end

	# ============================================================
	# edit
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Provide an existing product for editing.
	#   - HTML:
	#       Full-page edit (optional).
	#   - TURBO_STREAM:
	#       Show the edit form in side-body-frame.
	# ============================================================
	def edit
		respond_to do |format|
			format.html              # app/views/products/edit.html.erb (optional)
			format.turbo_stream      # app/views/products/edit.turbo_stream.erb
		end
	end

	# ============================================================
	# create
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Persist a new product.
	#   - On success:
	#       - TURBO_STREAM: update table, clear side, etc.
	#       - HTML: redirect as fallback.
	#   - On failure:
	#       - TURBO_STREAM: re-render form with errors.
	#       - HTML: render :new with errors.
	#
	# UI decisions (which ids, which frames) live in:
	#   - app/views/products/create.turbo_stream.erb
	# ============================================================
	def create
		@product = Product.new(product_params)

		if @product.save
			respond_to do |format|
				format.turbo_stream   # success branch: create.turbo_stream.erb
				format.html do
					redirect_to products_path, notice: "Product created."
				end
			end
		else
			respond_to do |format|
				format.turbo_stream   # error branch: create.turbo_stream.erb
				format.html do
					render :new, status: :unprocessable_entity
				end
			end
		end
	end

	# ============================================================
	# update
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Update an existing product.
	#   - On success:
	#       - TURBO_STREAM: update row, clear side, etc.
	#       - HTML: redirect as fallback.
	#   - On failure:
	#       - TURBO_STREAM: re-render form with errors.
	#       - HTML: render :edit with errors.
	#
	# UI wiring is in:
	#   - app/views/products/update.turbo_stream.erb
	# ============================================================
	def update
		if @product.update(product_params)
			respond_to do |format|
				format.turbo_stream   # success branch: update.turbo_stream.erb
				format.html do
					redirect_to products_path, notice: "Product updated."
				end
			end
		else
			respond_to do |format|
				format.turbo_stream   # error branch: update.turbo_stream.erb
				format.html do
					render :edit, status: :unprocessable_entity
				end
			end
		end
	end

	# ============================================================
	# destroy
	# ------------------------------------------------------------
	# Responsibilities:
	#   - Delete a product.
	#   - TURBO_STREAM:
	#       - remove the table row.
	#       - clear/reset side panel.
	#   - HTML: optional redirect path if needed.
	#
	# UI wiring lives in:
	#   - app/views/products/destroy.turbo_stream.erb
	# ============================================================
	def destroy
		@product.destroy!

		respond_to do |format|
			format.turbo_stream     # destroy.turbo_stream.erb
			# Optionally:
			# format.html { redirect_to products_path, notice: "Product deleted." }
		end
	end

	private

	def set_product
		@product = Product.find(params[:id])
	end

	def load_categories
		@categories = Category.order(:name)
	end

	def product_params
		params.require(:product).permit(
			:name,
			:code,
			:description,
			:category_id
		)
	end
end
```

## **2. Main index view (initial full-page)**

### **app/views/products/index.html.erb**



```erb
<%# ------------------------------------------------------------
    Products index page
    - Rendered on full-page visit (format.html).
    - The layout already defines #main, #side, #side-frame, etc.
    - Here we only care about main body content.
    - The products table itself is in a partial for reuse.
   ------------------------------------------------------------ %>

<section id="products-main">
	<h1 class="page-title">Products</h1>

	<%= render "products/table", products: @products %>
</section>
```

## **3. Table partials**

### **app/views/products/_table.html.erb**



```erb
<%# ------------------------------------------------------------
    Products table
    - Wrapped in a turbo-frame so the entire table area can be
      replaced by index.turbo_stream.erb (e.g. after filter).
    - DOM ids use dashes, not underscores.
    - Inner <cds-table-body> has its own id so we can prepend
      or replace rows without touching header.
   ------------------------------------------------------------ %>

<turbo-frame id="products-table-frame">
	<cds-table size="md">
		<cds-table-head>
			<cds-table-header-row>
				<cds-table-header-cell>Name</cds-table-header-cell>
				<cds-table-header-cell>Code</cds-table-header-cell>
				<cds-table-header-cell>Description</cds-table-header-cell>
				<cds-table-header-cell>Actions</cds-table-header-cell>
			</cds-table-header-row>
		</cds-table-head>

		<cds-table-body id="products-table">
			<%= render @products %> <%# renders _product.html.erb for each %>
		</cds-table-body>
	</cds-table>
</turbo-frame>
```

### **app/views/products/_product.html.erb**

```
<%# ------------------------------------------------------------
    Single product row
    - Rendered by "render @products".
    - Id uses dash naming: product-123.
    - This id is referenced in update.turbo_stream.erb and
      destroy.turbo_stream.erb to update/remove the row.
   ------------------------------------------------------------ %>

<tr id="product-<%= product.id %>">
	<td><%= product.name %></td>
	<td><%= product.code %></td>
	<td><%= product.description %></td>
	<td>
		<%# Show link that triggers show.turbo_stream.erb.
		    Using format: :turbo_stream so the controller will
		    respond with the turbo_stream view.
		    Table itself is not inside a frame for this; instead
		    the Turbo Stream response will update side-body-frame. %>
		<%= link_to "Show",
			product_path(product, format: :turbo_stream) %>
	</td>
</tr>
```

## **4. Form partial**

### **app/views/products/_form.html.erb**



```erb
<%# ------------------------------------------------------------
    Shared form partial for new/edit.
    - This markup is rendered inside side-body-frame by:
        new.turbo_stream.erb
        edit.turbo_stream.erb
        create.turbo_stream.erb (errors)
        update.turbo_stream.erb (errors)
    - The form submits as TURBO_STREAM by default when used
      in BOS shell.
   ------------------------------------------------------------ %>

<%= form_with model: product, local: false do |f| %>
	<div class="field">
		<%= f.label :name %>
		<%= f.text_field :name %>
		<%= f.error_message_on :name if f.object.errors[:name].any? rescue nil %>
	</div>

	<div class="field">
		<%= f.label :code %>
		<%= f.text_field :code %>
	</div>

	<div class="field">
		<%= f.label :description %>
		<%= f.text_area :description %>
	</div>

	<div class="field">
		<%= f.label :category_id, "Category" %>
		<%= f.collection_select :category_id, @categories, :id, :name, include_blank: true %>
	</div>

	<div class="actions">
		<%= f.submit %>
	</div>
<% end %>
```

(You can replace error_message_on with your own implementation; this is just a placeholder.)

## **5. Side neck partial**

### **app/views/shell/_side_neck.html.erb**



```erb
<%# ------------------------------------------------------------
    Side neck bar
    - Used for actions related to the current thing in side-body.
    - It is rendered in 2 modes:
        1) Without product (default from layout) -> no delete button.
        2) With product local -> show delete button.
    - "product" local is passed only when a product is loaded.
   ------------------------------------------------------------ %>

<div></div>

<div>
	<%# Only show delete if product local is present and persisted %>
	<% if local_assigns[:product]&.persisted? %>
		<%= button_to product_path(product, format: :turbo_stream),
			method: :delete,
			form: { data: { turbo_confirm: "Delete this product?" } },
			class: "cds--btn cds--btn--danger" do %>
			删除
		<% end %>
	<% end %>
</div>

<div></div>
```

Note: format: :turbo_stream ensures the destroy action renders destroy.turbo_stream.erb.

## **6. Turbo Stream views**

Now the important part: all the UI wiring lives here.

### **app/views/products/index.turbo_stream.erb**



```erb
<%# ------------------------------------------------------------
    index.turbo_stream.erb
    - Called when products#index responds as turbo_stream.
    - Responsibility:
        Refresh the products table area.
    - Controller only called "format.turbo_stream"; the choice
      to hit "products-table-frame" is here.
   ------------------------------------------------------------ %>

<%= turbo_stream.replace "products-table-frame" do %>
	<%= render "products/table", products: @products %>
<% end %>
```

### **app/views/products/show.turbo_stream.erb**



```erb
<%# ------------------------------------------------------------
    show.turbo_stream.erb
    - Called when link_to product_path(..., format: :turbo_stream)
      is clicked.
    - Responsibilities:
        1) Render product details into side-body-frame.
        2) Update side-neck to reflect current product
           (so delete button appears).
   ------------------------------------------------------------ %>

<%= turbo_stream.replace "side-body-frame" do %>
	<%= render "products/show", product: @product %>
<% end %>

<%= turbo_stream.replace "side-neck" do %>
	<%= render "shell/side_neck", product: @product %>
<% end %>
```

You will need a products/show partial (below) for the contents.

### **app/views/products/new.turbo_stream.erb**



```erb
<%# ------------------------------------------------------------
    new.turbo_stream.erb
    - Called when /products/new is requested as turbo_stream.
    - Responsibilities:
        1) Render blank form into side-body-frame.
        2) Optionally reset side-neck (no delete button).
   ------------------------------------------------------------ %>

<%= turbo_stream.replace "side-body-frame" do %>
	<%= render "products/form", product: @product %>
<% end %>

<%= turbo_stream.replace "side-neck" do %>
	<%= render "shell/side_neck" %> <%# no product local => no delete %>
<% end %>
```

### **app/views/products/edit.turbo_stream.erb**



```
<%# ------------------------------------------------------------
    edit.turbo_stream.erb
    - Called when /products/:id/edit is requested as turbo_stream.
    - Responsibilities:
        1) Render edit form into side-body-frame.
        2) Show delete button in side-neck for this product.
   ------------------------------------------------------------ %>

<%= turbo_stream.replace "side-body-frame" do %>
	<%= render "products/form", product: @product %>
<% end %>

<%= turbo_stream.replace "side-neck" do %>
	<%= render "shell/side_neck", product: @product %>
<% end %>
```

### **app/views/products/create.turbo_stream.erb**



```
<%# ------------------------------------------------------------
    create.turbo_stream.erb
    - Used for both success and failure of ProductsController#create.
    - Controller has already decided:
        - @product.save succeeded or failed.
    - Here we decide:
        - which frames to update.
        - how to treat error vs success.
   ------------------------------------------------------------ %>

<% if @product.errors.any? %>

	<%# Validation failed:
	    - Re-render the form with errors in side-body-frame.
	    - Keep side-neck as-is (it will likely have no product).
	%>
	<%= turbo_stream.replace "side-body-frame" do %>
		<%= render "products/form", product: @product %>
	<% end %>

<% else %>

	<%# Success:
	    1) Prepend new row to products-table.
	    2) Clear side-body-frame (close form).
	    3) Reset side-neck to default (no product).
	%>

	<%= turbo_stream.prepend "products-table" do %>
		<%= render "products/product", product: @product %>
	<% end %>

	<%= turbo_stream.replace "side-body-frame" do %>
		<%# empty => side body visually cleared %>
	<% end %>

	<%= turbo_stream.replace "side-neck" do %>
		<%= render "shell/side_neck" %>
	<% end %>

<% end %>
```

### **app/views/products/update.turbo_stream.erb**



```
<%# ------------------------------------------------------------
    update.turbo_stream.erb
    - Used for both success and failure of ProductsController#update.
   ------------------------------------------------------------ %>

<% if @product.errors.any? %>

	<%# Validation failed:
	    - Re-render the form with errors.
	%>
	<%= turbo_stream.replace "side-body-frame" do %>
		<%= render "products/form", product: @product %>
	<% end %>

<% else %>

	<%# Success:
	    1) Update the row for this product in the table.
	    2) Clear side-body-frame.
	    3) Reset side-neck.
	%>

	<%= turbo_stream.replace "product-#{@product.id}" do %>
		<%= render "products/product", product: @product %>
	<% end %>

	<%= turbo_stream.replace "side-body-frame" do %>
		<%# empty %>
	<% end %>

	<%= turbo_stream.replace "side-neck" do %>
		<%= render "shell/side_neck" %>
	<% end %>

<% end %>
```

### **app/views/products/destroy.turbo_stream.erb**



```
<%# ------------------------------------------------------------
    destroy.turbo_stream.erb
    - Called after ProductsController#destroy successfully deletes.
    - Responsibilities:
        1) Remove the product row from the table.
        2) Clear side-body-frame.
        3) Reset side-neck.
   ------------------------------------------------------------ %>

<%= turbo_stream.remove "product-#{@product.id}" %>

<%= turbo_stream.replace "side-body-frame" do %>
	<%# empty %>
<% end %>

<%= turbo_stream.replace "side-neck" do %>
	<%= render "shell/side_neck" %>
<% end %>
```

## **7. Product show content partial**

### **app/views/products/show.html.erb (full-page version)



```
<%# Full-page show; rarely used in BOS shell, but nice to have. %>

<h1><%= @product.name %></h1>
<p><strong>Code:</strong> <%= @product.code %></p>
<p><strong>Description:</strong> <%= @product.description %></p>
<p><strong>Category:</strong> <%= @product.category&.name %></p>
```



### **app/views/products/_show.html.erb **(for side-body-frame)

If you prefer a separate partial:

```
<%# ------------------------------------------------------------
    _show.html.erb
    - Used inside side-body-frame for show.turbo_stream.erb.
    - You can style this specifically for side panel.
   ------------------------------------------------------------ %>

<div class="product-show">
	<h2><%= product.name %></h2>

	<dl>
		<dt>Code</dt>
		<dd><%= product.code %></dd>

		<dt>Description</dt>
		<dd><%= product.description %></dd>

		<dt>Category</dt>
		<dd><%= product.category&.name %></dd>
	</dl>
</div>
```

Then update show.turbo_stream.erb to:

```
<%= turbo_stream.replace "side-body-frame" do %>
	<%= render "products/show", product: @product %>
<% end %>

<%= turbo_stream.replace "side-neck" do %>
	<%= render "shell/side_neck", product: @product %>
<% end %>
```











Now we answer your exact question:



------





# **⭐** 

# **What does a \*.turbo_stream.erb wire?**





A **Turbo Stream template wires:**



> **“Which DOM element (A) is updated with which HTML fragment (B).”**



In other words:



- It chooses **the target** (a DOM id)
- It specifies **the operation** (replace, remove, prepend, etc.)
- It renders **the partial** that becomes the new content





The controller does **NOT** decide these things — only the turbo_stream template does.



------





# **🔌** 

# **Turbo Stream Wires = (Operation) + (Target DOM ID) + (New HTML)**





Example:

```
<%= turbo_stream.replace "side-body-frame" do %>
	<%= render "products/form", product: @product %>
<% end %>
```

This line alone wires 3 things:





### **1.** 

### **Operation:**





replace





### **2.** 

### **Target DOM element:**





"side-body-frame"

(JavaScript: document.getElementById(“side-body-frame”))





### **3.** 

### **New HTML inserted:**





render "products/form"



Your form partial is the “building block”.

The turbo_stream file is the “electrician” connecting the block to the correct socket.



------





# **⚡** 

# **The 6 Wiring Operations in Turbo Streams**





Turbo Streams have the following “wires”:





### **1.** 

### **replace**





👉 Replace the entire element.





### **2.** 

### **append**





👉 Insert new content at the end of a list.





### **3.** 

### **prepend**





👉 Insert new content at the top of a list.





### **4.** 

### **remove**





👉 Delete the element (<div id="product-123">).





### **5.** 

### **update**





👉 Replace *inside* the element, but keep the element itself.





### **6.** 

### **before**

### **,** 

### **after**





👉 Insert sibling content next to target.



You almost always use:



- replace
- append / prepend
- remove





------





# **📌** 

# **Turbo Stream Files Describe UI Flow, Not HTML**





For example:





### **destroy.turbo_stream.erb**



```
<%= turbo_stream.remove "product-#{@product.id}" %>
<%= turbo_stream.replace "side-body-frame" do %> <% end %>
<%= turbo_stream.replace "side-neck" do %> <%= render "shell/side_neck" %> <% end %>
```

This file wires:

| **Operation** | **Target DOM ID** | **Replacement Content**    |
| ------------- | ----------------- | -------------------------- |
| remove        | "product-123"     | — (nothing)                |
| replace       | "side-body-frame" | empty (clear panel)        |
| replace       | "side-neck"       | render _side_neck.html.erb |

This is **pure UI logic**.

The controller is completely free from UI decisions.



------





# **🧱** 

# **HTML partials (form, table, row, show)**





These are dumb building blocks.



Examples:



- _product.html.erb — a single <tr>
- _table.html.erb — full table
- _form.html.erb — product form
- _show.html.erb — product info
- _side_neck.html.erb — delete button bar





They contain HTML (contents), but **no wiring**.

They don’t know *where* they get inserted.



------





# **🔌** 

# **Turbo Streams = “flow control between blocks”**





*.turbo_stream.erb files connect:





### **HTML building block → DOM target**





For example, edit.turbo_stream.erb says:



> “When editing a product, insert the _form partial into side-body-frame,

> AND update side-neck with the delete button.”

```
<%= turbo_stream.replace "side-body-frame" do %>
	<%= render "products/form", product: @product %>
<% end %>

<%= turbo_stream.replace "side-neck" do %>
	<%= render "shell/side_neck", product: @product %>
<% end %>
```

Controllers DO NOT say:



- “put this in side-body-frame”
- “replace this frame”
- “clear this”
- or even mention DOM ids





That is wiring.



------





# **🧠** 

# **The beautiful conceptual model**







### **1. HTML partials**





= bricks, furniture, components

(no idea where they go)





### **2. Turbo Stream templates**





= electricians

(connect components → frames)





### **3. Controller**





= foreman

(doesn’t touch wires or furniture; just passes data around)





### **4. Layout**





= house architecture (rooms and containers)



------





# **🎯** 

# **Answer in one sentence**





> **A \*.turbo_stream.erb wires which DOM element (via ID) should receive which partial or HTML fragment, using operations like replace/prepend/remove.**



That’s it.

Once you fully adopt this model, your BOS UI stays clean, modular, predictable, and future-proof.



------



If you want, I can draw a “map diagram” of:



- DOM regions (#main, #side-frame → side-body-frame → side-neck)
- actions (new/edit/destroy)
- turbo_stream wiring flows





Just ask:

**“Give me the BOS wiring map.”**