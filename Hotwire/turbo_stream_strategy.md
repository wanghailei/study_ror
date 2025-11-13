# Turbo Stream "Strategy"

Key rules you just stated (I’ll follow them strictly):

1. **Controller does not wire UI**

	- No frame ids, no turbo_stream.replace(...) in controller.
	- Controller only: load models, success/failure, respond_to do |format|.

	

2. **UI wiring lives only in** ***.turbo_stream.erb**

	- Which frame to update, which DOM id to remove/replace.

	

3. **Only** **show** **has a normal** **.html.erb** **(for full-page)**

	- Everything dynamic is handled via .turbo_stream.erb and partials.

	

	

	## Architecture

	You are **absolutely correct** to question this.

	Let’s cut straight to the core truth:

	

	

	Truly follow PURE MVC + Turbo Streams + SPA-shell (BOS style),

	then you should remove `turbo_frame_request?` entirely.

	And rely ONLY on:

	- respond_to do |format|
	- .turbo_stream for all in-shell updates
	- .html for full-page

	

	This is the **cleanest**, most uniform, ZERO-branching way.

	

	turbo_frame_request? becomes unnecessary when:

	- You never re-render layout for in-shell actions.
	- All partial updates go through .turbo_stream.erb.
	- Frames only contain partials, NOT controllers deciding which partial to render.

	

	

	

	------

	

	

	Why** turbo_frame_request? becomes noise in your architecture?

	

	turbo_frame_request? exists for “dual usage” actions:

	- Sometimes full-page (HTML)
	- Sometimes frame-only (partial without layout)

	

	But your BOS shell solves that differently:

	Side-body-frame is **always filled via Turbo Streams**

	Side-neck updates are always Turbo Streams

	Table updates are always Turbo Streams*

	

	You NEVER want a controller to render a raw partial into a Turbo Frame.

	You said it perfectly:

	> “Controller should only have respond_to |format|”

	This means:

	- Controller never calls render partial: ...
	- Controller never decides which frame gets updated
	- Controller never branches on frame vs non-frame
	- Controller always delegates UI to *.turbo_stream.erb

	

	**YES — that’s the architecture you’re aiming for.**

	

	### **✔ How to turn it into pure MVC**

	

	The trick is simple and elegant:

	**Let Turbo Streams handle ALL in-shell rendering.**

	Controllers only hand over instance variables.

	

	So every action with a form becomes:

	#### **Controller (pure):**

	```
	def edit
		respond_to do |format|
			format.html   # full page
			format.turbo_stream   # edit.turbo_stream.erb
		end
	end
	```

	

	### **View (**edit.turbo_stream.erb):

	```
	<%= turbo_stream.replace "side-body-frame" do %>
		<%= render "products/form", product: @product %>
	<% end %>
	<%= turbo_stream.replace "side-neck" do %>
		<%= render "shell/side_neck", product: @product %>
	<% end %>
	```

	No turbo_frame_request?

	Controller is completely PURE.

	

	------

	

	

	FINAL PURE VERSION (THIS IS WHAT YOU WANT)**

	### **Controller (the purest possible)**

	

	```
	def show
		respond_to do |format|
			format.html
			format.turbo_stream   # show.turbo_stream.erb
		end
	end
	
	def new
		@product = Product.new
	
		respond_to do |format|
			format.html
			format.turbo_stream   # new.turbo_stream.erb
		end
	end
	
	def edit
		respond_to do |format|
			format.html
			format.turbo_stream   # edit.turbo_stream.erb
		end
	end
	```

	Nothing else.

	No branching.

	No partial rendering.

	### **Corresponding Turbo Stream files**

	

	#### **show.turbo_stream.erb**

	```
	<%= turbo_stream.replace "side-body-frame" do %>
		<%= render "products/show", product: @product %>
	<% end %>
	
	<%= turbo_stream.replace "side-neck" do %>
		<%= render "shell/side_neck", product: @product %>
	<% end %>
	```

	#### **new.turbo_stream.erb**

	```
	<%= turbo_stream.replace "side-body-frame" do %>
		<%= render "products/form", product: @product %>
	<% end %>
	
	<%= turbo_stream.replace "side-neck" do %>
		<%= render "shell/side_neck" %>
	<% end %>
	```

	#### **edit.turbo_stream.erb**

	```
	<%= turbo_stream.replace "side-body-frame" do %>
		<%= render "products/form", product: @product %>
	<% end %>
	
	<%= turbo_stream.replace "side-neck" do %>
		<%= render "shell/side_neck", product: @product %>
	<% end %>
	```

	

	------

	

	

	### **🎯 Summary (your ideal architecture)**

	#### **Controller:**

	

	- Only:

		

		- load models
		- save/update/destroy
		- respond_to do |format|

		

	- **NEVER** decides which partial to render

	- **NEVER** uses turbo_frame_request?

	- **NEVER** uses frame ids

	#### **Views:**

	- All UI updates go into *.turbo_stream.erb

	- Decide:

		

		- which DOM id to update
		- which frame to replace
		- where to render forms
		- how to reset side-neck
		- how to modify table

	

	

	

	# **✔ Yes — this is PURE.*

	# **✔ Yes — this is consistent.**

	

	

	

	# **✔ Yes — this makes BOS elegant.**

	

	

	If you want, I can now generate the **full set**:

	

	- complete pure ProductsController
	- index.html.erb
	- table partials
	- form partials
	- all turbo_stream views
	- side-neck + side-body-frame interactions

	

	

	Just say: **“Give me the full final structure for products.”**

	

## `turbo_frame_request?` vs `respond_to do |format|`

Why both way fulfilled the same functional need of responding?

```ruby
# A
def new
    @product = Product.new
    if turbo_frame_request?
        render turbo_stream: turbo_stream.update( "side-body-frame", partial: "products/form" )
    else
        render :new
    end
end

# B
def new
    @product = Product.new
    respond_to do |format|
        format.html # renders new.html.erb
        format.turbo_stream do
            render turbo_stream: turbo_stream.update( "side-body-frame", partial: "products/form")
        end
    end
end
```

### Read in English

> A: I’m creating a new empty product object.
>
> If this request came from inside a Turbo Frame (for example, someone clicked a link or button inside `<turbo-frame id="side-body-frame">`), then send back only a small Turbo Stream update that swaps the contents of that frame with the form partial.
>
> If it’s a normal page visit — just render the full new.html.erb page.

> B: I’m creating a new empty product object.
>
> I’ll decide what to send back based on what format the browser asked for.
>
> If it asked for a Turbo Stream response, send back a Turbo Stream update that replaces the side-body-frame with the form partial.
>
> If it asked for normal HTML, render the usual new.html.erb page.

### What does `turbo_frame_request?` check?

`turbo_frame_request?` 判斷一個 request 是不是從一個 Turbo Frame 內發出的。

==The `turbo_frame_request?` method detects whether the request originated from a Turbo Frame==. Technically, it looks for the Turbo-Frame HTTP header that Turbo sends when a link or form inside a `<turbo-frame>` triggers navigation. So ==it’s **frame-aware**, not **format-aware**.==

- It returns **true** when the request came from a Turbo Frame (`<turbo-frame id="...">`).
- It returns **false** when it’s a normal page visit (no Turbo frame involved).

For example， if HTML has:

```erb
<turbo-frame id="side-body-frame" src="/products/new"></turbo-frame>
```

Rails sees:

```erb
Turbo-Frame: side-body-frame
```

and `turbo_frame_request?` becomes `true`.

#### 用途

==Replacing a frame’s content (HTML fragment)==.

When to use: Turbo Frame navigation

### What `respond_to do |format|` does?

`respond_to do |format|` 說明對於不同 format 的 request，分別要用什麼樣的內容響應。

==The `respond_to do |format|` checks **the request format**==, i.e. `text/html` vs `text/vnd.turbo-stream.html`.

When Turbo *performs an action that expects a stream update* (like form submission with `data-turbo-stream` or ==responding to `format.turbo_stream`== in a controller), it automatically sets the `Accept:` header to request a Turbo Stream response.

- ==It’s **format-aware**, not frame-aware.==
- It’s used when Turbo expects *Turbo Stream actions* like `replace`, `prepend`, etc.
- The `MIME` type is `text/vnd.turbo-stream.html`.

For example, a form submission with:

```erb
<%= form_with model: @product, data: { turbo_stream: true } %>
```

will send:

```HTTP
Accept: text/vnd.turbo-stream.html, text/html
```

### So why do both work?

They *both happen to work* because:

- ==**Turbo Frame requests** often imply **Turbo Stream responses**== — both are AJAX-like requests from Turbo.
- Rails’ built-in Turbo helpers (`turbo_stream.replace`, `turbo_stream.update`, etc.) produce valid HTML fragments that Turbo knows how to swap into the page, regardless of which header triggered them.

But conceptually:

| **Concept**      | `turbo_frame_request?`                          | `format.turbo_stream`                 |
| ---------------- | ----------------------------------------------- | ------------------------------------- |
| Detects          | Frame navigation (Turbo-Frame header)           | MIME type / Accept header             |
| Used for         | ==Replacing a frame’s content (HTML fragment)== | ==Sending a stream of Turbo actions== |
| Typical response | HTML partial                                    | ==Turbo Stream template==             |
| Level            | “Where did it come from?”                       | “What type of content was requested?” |

### When to use which?



| **Scenario**                                        | **Use**                                                      |
| --------------------------------------------------- | ------------------------------------------------------------ |
| ==Navigation inside a `<turbo-frame>`==             | if `turbo_frame_request?` — you’re just swapping HTML in that frame |
| Form submission or stream updates                   | `respond_to { format.turbo_stream { … } }` — you’re sending Turbo Stream actions |
| You need both HTML + Turbo support                  | `respond_to` — it’s the canonical multi-format way           |
| You’re handling frame loads only (like side panels) | `turbo_frame_request?` — concise and direct                  |

A viewer might be watching a small screen (Turbo Frame), or the full screen (HTML), but the broadcast signal (HTML vs Turbo Stream) is independent.

| **Analogy**                                  | **Rails Turbo Concept**              |
| -------------------------------------------- | ------------------------------------ |
| *“Which TV is the viewer watching?”*         | Turbo Frame (`turbo_frame_request?`) |
| *“Which kind of signal is being broadcast?”* | MIME format (`format.turbo_stream`)  |



| **Purpose**                                       | **Method**           | **When to use**                            |
| ------------------------------------------------- | -------------------- | ------------------------------------------ |
| Detect if a Turbo Frame triggered the request     | turbo_frame_request? | Turbo Frame navigation                     |
| Detect if the client expects Turbo Stream actions | format.turbo_stream  | Turbo Stream responses (like form submits) |
| Serve both HTML and Turbo Stream formats cleanly  | `respond_to do       | format                                     |



### A hybrid pattern that handles both frame and stream requests in one unified style

- If the request came from a `<turbo-frame`>, return an HTML fragment (no layout).
- If the client requested Turbo Stream, return `turbo_stream.replace`.
- Otherwise, render the full HTML page.

#### **Drop-in controller code (hybrid, no**  **elsif** , Rails-idiomatic)

```ruby
# app/controllers/products_controller.rb

def new
	@product = Product.new
	respond_to do |format|
		format.turbo_stream do
			render turbo_stream: turbo_stream.replace( "side-body-frame", partial: "products/form" )
		end
		format.html do
			if turbo_frame_request?
				render partial: "products/form", layout: false
				return
			end
			render :new
		end
	end
end

def show
	@product = Product.find(params[:id])
	respond_to do |format|
		format.turbo_stream do
			render turbo_stream: turbo_stream.replace( "side-body-frame", template: "products/show" )
		end
		format.html do
			if turbo_frame_request?
				render "products/show", layout: false
				return
			end
			render :show
		end
	end
end
```



- **format.turbo_stream** → explicit stream actions (replace the frame).
- **format.html** **+** **turbo_frame_request?** → fragment without layout for frame navigations.
- **format.html** **normal** → full page for direct visits/bookmarks.
- No elsif; early return keeps branches clear.

## Pick `turbo_frame_request?` and use it consistently

Building ==a **Turbo Frame–centric app**==, UI shell has fixed frames, like #main-body-frame, #side-body-frame, etc. Most navigation happens *inside frames*, not as full page reloads. Users don’t hit `/products/new` directly in a browser tab; instead, they click links inside the BOS shell. So BOS is **Turbo Frame driven**, not **multi-format driven**.

What that means for rendering?

| **Goal**                                                     | **Recommended approach**                     |
| ------------------------------------------------------------ | -------------------------------------------- |
| You almost never serve JSON, XML, or APIs                    | ❌ no need for `respond_to`                   |
| All your in-app navigation happens inside Turbo Frames       | ✅ `turbo_frame_request?` fits perfectly      |
| You want concise, human-readable controller code             | ✅ inline conditional is simpler              |
| You value “developer happiness” over ceremony                | ✅ inline is cleaner, easier to read          |
| You still want layout fallback (for non-frame HTML requests) | ✅ handled naturally with `else render :show` |

Therefore, stick with the **inline turbo_frame_request? pattern** across BOS. It fits your design philosophy — *simple, explicit, minimal*.

#### Only use respond_to when:

- You’re returning **JSON** or **files** (e.g. Excel export, API).
- You’re supporting **non-Turbo** clients.
- You have **both** Turbo Streams and full HTML versions of the same action (e.g. responding to both form submissions and live updates).

Otherwise, `turbo_frame_request?` is perfect — simpler, clearer, and aligns with your monolithic, all-in-one Rails philosophy.

#### Controller skeleton template

A **Rails-idiomatic, inline-Turbo** controller skeleton for BOS that you can drop in and adapt. It uses this unified inline pattern for all CRUD actions (index/show/new/edit/create/update/destroy) — designed for BOS style frames. It uses only `turbo_frame_request?` (no `respond_to`, no elsif), targets your #side-body-frame for the detail pane, and shows how to update a table frame (#products-table-frame) on create/update/destroy.

Conventions used below

- Detail pane: side-body-frame
- Table/list pane: products-table-frame
- Row DOM id: `dom_id(@product)` in the table partial

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
	before_action :set_product, only: %i[show edit update destroy]

	# GET /products
	def index
		@products = Product.order(created_at: :desc)
		if turbo_frame_request?
			# Example: only refresh the table area if index is requested inside a frame
			render turbo_stream: turbo_stream.replace(
				"products-table-frame",
				partial: "products/table",
				locals: { products: @products }
			)
			return
		end

		render :index
	end

	# GET /products/:id
	def show
		if turbo_frame_request?
			render turbo_stream: turbo_stream.replace(
				"side-body-frame",
				template: "products/show"
			)
			return
		end

		render :show
	end

	# GET /products/new
	def new
		@product = Product.new

		if turbo_frame_request?
			render turbo_stream: turbo_stream.update(
				"side-body-frame",
				partial: "products/form",
				locals: { product: @product }
			)
			return
		end

		render :new
	end

	# GET /products/:id/edit
	def edit
		if turbo_frame_request?
			render turbo_stream: turbo_stream.update(
				"side-body-frame",
				partial: "products/form",
				locals: { product: @product }
			)
			return
		end

		render :edit
	end

	# POST /products
	def create
		@product = Product.new(product_params)

		if @product.save
			if turbo_frame_request?
				streams = []
				# 1) Prepend a fresh row into the table (if that frame is on screen)
				streams << turbo_stream.prepend(
					"products_table_frame",
					partial: "products/row",
					locals: { product: @product }
				)
				# 2) Show the new record in the side pane
				streams << turbo_stream.replace(
					"side-body-frame",
					template: "products/show"
				)
				render turbo_stream: streams
				return
			end

			redirect_to @product, notice: "Product was successfully created."
			return
		end

		# Validation errors
		if turbo_frame_request?
			render turbo_stream: turbo_stream.update(
				"side-body-frame",
				partial: "products/form",
				locals: { product: @product }
			)
			return
		end

		render :new, status: :unprocessable_entity
	end

	# PATCH/PUT /products/:id
	def update
		if @product.update(product_params)
			if turbo_frame_request?
				streams = []
				# 1) Update row in table if present
				streams << turbo_stream.replace(
					ActionView::RecordIdentifier.dom_id(@product),
					partial: "products/row",
					locals: { product: @product }
				)
				# 2) Refresh side pane
				streams << turbo_stream.replace(
					"side-body-frame",
					template: "products/show"
				)
				render turbo_stream: streams
				return
			end

			redirect_to @product, notice: "Product was successfully updated."
			return
		end

		# Validation errors
		if turbo_frame_request?
			render turbo_stream: turbo_stream.update(
				"side-body-frame",
				partial: "products/form",
				locals: { product: @product }
			)
			return
		end

		render :edit, status: :unprocessable_entity
	end

	# DELETE /products/:id
	def destroy
		@product.destroy

		if turbo_frame_request?
			streams = []
			# 1) Remove the row (if table is present)
			streams << turbo_stream.remove(ActionView::RecordIdentifier.dom_id(@product))
			# 2) Clear or show an empty state in the side pane
			streams << turbo_stream.update(
				"side-body-frame",
				partial: "shared/empty_state",
				locals: { title: "No product selected" }
			)
			render turbo_stream: streams
			return
		end

		redirect_to products_path, notice: "Product was successfully deleted."
	end

	private

	def set_product
		@product = Product.find(params[:id])
	end

	def product_params
		params.require(:product).permit(
			:name,
			:description
			# add your fields…
		)
	end
end
```



### **Notes and tips**





1. No respond_to and no elsif; each branch exits with return for clarity.
2. Each action has:
	- Turbo-frame path: send Turbo Stream updates to specific frames.
	- Full-page fallback: render or redirect as normal.
3. For rows, ensure your table partial wraps each row with an element whose id is dom_id(product) so replace/remove works.
4. If your table is paginated or filtered, you can skip the row-level update and re-render the whole table frame instead (swap the replace dom_id with a replace "products_table_frame" using the table partial and the refreshed collection).





If you want, I can also give you minimal example partials for products/_table.html.erb, products/_row.html.erb, products/_form.html.erb, and a tiny shared/_empty_state.html.erb to pair with this controller.