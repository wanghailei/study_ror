# Routes in Fizzy

> How do you like the resourceful route design of Fizzy? What shall I learn from it for BOS. I actually don't understand the scope and namespace Fizzy uses.



Gemini: The `routes.rb` you uploaded (presumably from "Fizzy") is a masterclass in ==**RESTful purity** and **resource-oriented design**.== It aligns perfectly with your aesthetic of "being-in-order" (美就是秩序).

It eschews standard CRUD bucket controllers (giant controllers with custom actions) in favor of ==many small, single-purpose controllers==.

### `scope` vs `namespace`

This is the technical confusion you mentioned. Here is the concise difference, specifically regarding how Fizzy uses them:

1. **`namespace :admin`**
	- **URL:** Adds a prefix (`/admin/users`)
	- **Controller:** Adds a module (`Admin::UsersController`)
	- **Helper:** Adds a prefix (`admin_users_path`)
	- *Use when:* You want a separate area of the site, like a backend.
2. **`scope module: :users`** (Fizzy’s favorite pattern)
	- **URL:** No change (Keeps it clean).
	- **Controller:** Adds a module (`Users::EventsController`).
	- **Helper:** No change.
	- *Use when:* You want to organize your code files (folders) nicely without making the URLs ugly or deep.

**Example from your file:**

Ruby

```
resources :boards do
	scope module: :boards do
		resource :subscriptions
	end
end
```

- **URL:** `/boards/1/subscriptions` (Clean, intuitive)
- **File:** `app/controllers/boards/subscriptions_controller.rb` (Organized folder)
- **Class:** `Boards::SubscriptionsController`

If they had used `namespace :boards` inside `resources :boards`, the URL would likely have become redundant or required awkward overrides.

------

### What to learn for BOS

Fizzy’s routing design is highly recommended for your Bakery BOS. It turns complex business logic into simple HTTP verbs.

#### 1. Verbs as Resources (Action-based Resources)

Instead of adding custom actions to a controller (e.g., `def publish` inside `CardsController`), Fizzy creates a virtual resource.

- **Traditional:** `POST /cards/1/publish` -> `CardsController#publish`

- **Fizzy:** `POST /cards/1/publish` -> `Cards::PublishesController#create`

- **In `routes.rb`:**

	Ruby

	```
	resources :cards do
		scope module: :cards do
			resource :publish # Singular resource
		end
	end
	```

- **Benefit:** Your controllers stay tiny. The "Publishing" logic lives in its own file. For a complex system like BOS (e.g., "Baking a Batch"), this is crucial to prevent 1000-line controllers.

#### 2. Singular Resources (`resource` vs `resources`)

Fizzy uses `resource` (singular) extensively (e.g., `resource :not_now`, `resource :settings`).

- This implies there is only **one** of this thing relative to the parent.
- Example: A User has only one Avatar. `resource :avatar`.
- Result: Routes like `/users/1/avatar` (no ID required for the avatar itself).

#### 3. Deep Organization, Shallow URLs

By using `scope module:`, Fizzy groups related concepts in the codebase (e.g., `boards/`, `cards/`, `columns/` directories) but keeps the URLs relatively shallow and readable for the user.

### Summary for your BOS

For your Bakery Operating System, adopt this pattern to handle complexity:

1. **Don't build a `RecipesController` with 50 methods.**
2. **Break it down:**
	- `Recipes::IngredientsController` (managing the list)
	- `Recipes::CostsController` (calculating ABC)
	- `Recipes::VersionsController` (managing history)
3. **Use `scope module:`** to keep your file structure matching your domain understanding without making URLs weird.

Would you like me to draft a `routes.rb` snippet for your **Product & Recipe Management Module** using this specific Fizzy style?



## ChatGPT

I like Fizzy’s routes a lot: they’re very *resourceful* and quite disciplined. But they *are* dense, and the scope module: vs namespace layering is exactly the kind of thing that’s hard to read if you’re not the one who wrote it. Let me unpack what it’s doing and what you can steal for BOS.

### Quick mental model

In Rails routing, these three are the key players:

- `resources :things`

	- Adds RESTful routes with an **ID**: `/things`, `/things/:id`.

	- Controller: `ThingsController`.

		

- resource :thing

	- Adds RESTful routes **without an ID** (one per “owner”): `/thing`.
	- Controller: ThingsController (singular resource, plural controller).

	

- namespace :admin vs scope module: :admin

	

	- **namespace :admin**

		

		- Path prefix: /admin/...
		- Controller module: Admin::...
		- Route helper prefix: admin_..._path

		

	- **scope module: :admin**

		

		- **No path prefix**, only controller module changed to Admin::...
		- No helper prefix; paths look the same as outside.

		

	





So:



- Use namespace when you want **URL + controller + helper** prefixed.
- Use scope module: when you want **only controllers** to live in a sub-module, but URLs stay “clean”.





That’s 80% of the confusion.



------





## **2. How Fizzy structures things**







### **2.1 Account area (classic** 

### **namespace**

### **)**



```
namespace :account do
	resource :join_code
	resource :settings
	resource :entropy
	resources :exports, only: [ :create, :show ]
end
```

This means:



- URLs:

	

	- /account/join_code
	- /account/settings
	- /account/entropy
	- /account/exports/:id

	

- Controllers:

	

	- Account::JoinCodesController
	- Account::SettingsController
	- Account::EntropiesController
	- Account::ExportsController

	

- Helper methods:

	

	- account_join_code_path, account_settings_path, etc.

	





**Pattern to copy for BOS:**

Use a namespace for “areas” of the app:



- namespace :admin
- namespace :account
- namespace :public
- etc.





Anything that is conceptually “a section” of BOS (back office, reporting, system configuration) fits here.



------





### **2.2 Users with nested “satellite” resources (**

### **scope module:**

### **)**



```
resources :users do
	scope module: :users do
		resource :avatar
		resource :role
		resource :events
		resources :push_subscriptions
		resources :email_addresses, param: :token do
			resource :confirmation, module: :email_addresses
		end
	end
end
```

What this does:



- Outer resources :users → /users, /users/:id, controller UsersController.

- Inside that, scope module: :users:

	

	- Paths are **still** /users/:user_id/avatar, /users/:user_id/role, etc.

	- Controllers are:

		

		- Users::AvatarsController
		- Users::RolesController
		- Users::EventsController
		- Users::PushSubscriptionsController
		- Users::EmailAddressesController
		- Users::EmailAddresses::ConfirmationsController

		

	

- Route helpers like:

	

	- user_avatar_path(@user)
	- user_push_subscriptions_path(@user)

	





**Why this is nice**



- Keeps URL hierarchy simple and readable.
- Keeps controllers organised in modules by owner resource (Users::…) instead of one huge UsersController.
- Uses resource (singular) when a user only has **one** avatar, **one** role, **one** events feed, etc.





**What you can copy for BOS**



For BOS, think:

```
resources :products do
	scope module: :products do
		resource :formula      # product has one current formula
		resources :formulas    # or many historical formulas, if you want
		resource :pricing      # current pricing record
		resource :inventory    # aggregated inventory summary
	end
end
```

That gives you:



- “All product stuff is under /products/:product_id/...”
- Controllers live in Products::FormulasController, Products::PricingController, etc.
- And you avoid stuffing everything into ProductsController.





------





### **2.3 Boards + columns + cards: nested behaviour**





Example pattern:

```
resources :boards do
	scope module: :boards do
		resource :subscriptions
		resource :involvement
		resource :publication
		resource :entropy

		namespace :columns do
			resource :not_now
			resource :stream
			resource :closed
		end

		resources :columns
	end

	resources :cards, only: :create

	resources :webhooks do
		scope module: :webhooks do
			resource :activation, only: :create
		end
	end
end
```

What’s going on:



- Each board has its own controllers in Boards::....

- resource :subscriptions, :involvement, :publication, :entropy are **singleton per board**:

	

	- Path: /boards/:board_id/subscriptions
	- Controller: Boards::SubscriptionsController

	

	You can imagine show, create, update, etc. working on *that one subscription for that board*.

- The namespace :columns inside the scope adds an extra level in both path and module:

	

	- /boards/:board_id/columns/not_now → Boards::Columns::NotNowController
	- Those are like specialised behaviours or “modes” about columns, not the columns themselves.

	

- Then resources :columns gives you the main Boards::ColumnsController / /boards/:board_id/columns CRUD.





**For BOS**



You can mirror this pattern for “behaviour controllers”:



- Drag-drop, reordering, toggles, state changes, etc. can live in their own controllers under namespaces, not in the main controller.





For example:

```
resources :formulas do
	scope module: :formulas do
		namespace :ingredients do
			resource :import
			resource :export
		end
		resource :audits  # single “view” of audit info per formula
	end
end
```



------





### **2.4 Columns & cards as a separate axis**





Later you have:

```
resources :columns, only: [] do
	resource :left_position,  module: :columns
	resource :right_position, module: :columns
end

namespace :columns do
	resources :cards do
		scope module: :cards do
			namespace :drops do
				resource :not_now
				resource :stream
				resource :closure
				resource :column
			end
		end
	end
end
```

Patterns:



- resources :columns, only: [] is a trick to express “actions that belong to a column” without CRUD for columns themselves here.

	

	- /columns/:column_id/left_position → Columns::LeftPositionsController
	- /columns/:column_id/right_position → Columns::RightPositionsController

	

	So left/right reordering of a column is just a resource attached to the column.

- Separate namespace :columns → everything under /columns/... and Columns::... controllers, e.g.:

	

	- /columns/cards/:id/drops/not_now
	- Columns::Cards::Drops::NotNowsController

	





You can use the same trick for BOS when you have “actions attached to something but not CRUD”. For example:

```
resources :products, only: [] do
	scope module: :products do
		resource :archive
		resource :duplicate
	end
end
```



------





### **2.5 Notifications, searches, events…**





You see a recurring pattern:

```
resources :notifications do
	scope module: :notifications do
		get "tray", to: "trays#show", on: :collection

		resource :reading
		collection do
			resource :bulk_reading, only: :create
		end
	end
end
```



- Main REST: NotificationsController.
- Additional actions in Notifications::ReadingsController, Notifications::BulkReadingsController, etc.
- Kept under the notifications URL tree, but not bloating the main controller.





Same with:

```
resource :search
namespace :searches do
	resources :queries
end
```

and:

```
resources :events, only: :index
namespace :events do
	resources :days
end
```

→ One base resource, and a namespace for “sub-resources” with their own controllers.



------





### **2.6 Public / My / Prompts – “areas” of the app**





Examples:

```
namespace :my do
	resources :pins
	resource :timezone
	resource :menu
end

namespace :prompts do
	resources :cards
	resources :tags
	resources :users

	resources :boards do
		scope module: :boards do
			resources :users
		end
	end
end

namespace :public do
	resources :boards do
		scope module: :boards do
			namespace :columns do
				resource :not_now, only: :show
				resource :stream,  only: :show
				resource :closed,  only: :show
			end
			resources :columns, only: :show
		end
	end
end
```

These are clearly “zones”:



- My::... → each user’s personal settings/stuff.
- Prompts::... → prompt-specific subsystem.
- Public::... → public-facing views of boards/columns.





**For BOS**



You almost certainly want:



- namespace :public – public menu / store locator / product catalogue views.
- namespace :admin – system-level admin.
- Maybe namespace :ops or :factory if you want separate UI for the factory vs shops.





------





### **2.7 The** 

### **resolve**

###  **blocks**





At the end:

```
resolve "Notification" do |notification, options|
	polymorphic_url(notification.notifiable_target, options)
end

resolve "Event" do |event, options|
	polymorphic_url(event.eventable, options)
end

resolve "Webhook" do |webhook, options|
	route_for :board_webhook, webhook.board, webhook, options
end
```

This customises polymorphic_url so that:



- polymorphic_url(@notification) will redirect to the *thing the notification is about* (notification.notifiable_target).
- Same for events.





This is a nice advanced trick but optional for BOS. You could use it for:



- “Activity” records that link back naturally to product, order, formula, etc., without casework in views.





------





## **3. What you should take into BOS (summary)**







### **Concrete patterns worth copying**





1. **Use namespaces for “zones”:**



```
namespace :account do
  resource :settings
end

namespace :admin do
  resources :users
  resources :stores
end

namespace :public do
  resources :products, only: [:index, :show]
end
```



1. 
2. **Use** **scope module:** **under a resource to split controllers without changing URLs:**



```
resources :products do
  scope module: :products do
    resource  :pricing
    resource  :inventory
    resources :images
  end
end
```



1. 

2. **Use** **resource** **(singular) when there is at most one per parent:**

	

	- Product has **one** main formula → resource :formula.
	- Company has **one** settings record → resource :settings.

	

3. **Create small “behaviour” controllers instead of dumping actions into main controllers:**



```
resources :formulas, only: [] do
  scope module: :formulas do
    resource :duplicate
    resource :archive
  end
end
```



1. 
2. **Group personal stuff under** **namespace :my****, and public-facing info under** **namespace :public****.**
3. **Use** **only:** **everywhere** to keep your routes surface small and explicit.





------



If you’d like, next step I can help you sketch a BOS routes.rb that mirrors Fizzy’s style, but mapped to your actual domain: accounts, companies, stores, products, formulas, materials, production batches, etc.