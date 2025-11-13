# Polymorphic Associations

> You should enforce referential integrity whenever you can by using foreign keys. It's almost a guarantee that if you do not enforce referential integrity in your database, you'll eventually have foreign key columns that point to rows that no longer exist. You should avoid any feature that does not allow the enforcement of referential integrity, such as polymorphic associations. 
>
> Excerpt From *Polished Ruby Programming* by Jeremy Evans

A **polymorphic association** is a mechanism, common in frameworks like Ruby on Rails, where a model can belong to more than one other type of model using a *single* association. 

It's a pragmatic solution for a common modeling problem: when you want ==a single model to be attachable to many other different types of models.== 

The classic example is a `Comment`. Imagine your application has `Posts`, `Videos`, and `Products`. You want users to be able to comment on *all* of them. 

A polymorphic association solves this by consolidating the relationship into one "interface." Instead of `post_id`, `video_id`, etc., the `comments` table has two columns: `commentable_id` (an integer) and `commentable_type` (a string).

This is extremely flexible at the application layer. The `comments` table can now hold comments for `Posts`, `Videos`, `Products`, or *any other model* you add later, all without changing the database schema. It is a brilliant solution for application-level flexibility.

This convenience, however, is precisely what Jeremy Evans warns about. The database itself cannot enforce that row #1 (with `commentable_id: 5`, `commentable_type: "Post"`) actually points to a valid row in the `posts` table. This means the application layer is solely responsible for integrity, and orphaned rows (where `associable_id` points to a non-existent row) are a significant risk.

### When to Choose Polymorphism (Convenience)

The choice between them is a classic design trade-off: application convenience vs. database integrity. There is no single right answer, only the best choice for a specific situation.

Choose a polymorphic association when the "attachable" model is **non-critical** and the main benefit is **extensibility**.

This pattern is best for features like:

- **Tags:** A `Tag` (e.g., "ruby") can apply to a `Post`, a `Product`, or a `Video`. If a `Post` is deleted and the `tagging` row is orphaned, it's messy but doesn't break your business.
- **Comments:** As in the example, `Comments` can apply to many things.1
- **Likes:** A `User` can "like" a `Photo`, a `Comment`, or a `Post`.

The key is that this data is often *supplementary*. The business can still run if a comment points to a post that no longer exists (though it's a bug you'd want to fix).

### When to Choose Strict Foreign Keys (Integrity)

==Choose strict foreign keys when the relationship is **essential to your business logic** and data integrity is paramount.==

This is the correct choice for almost all core data:

- **Invoicing:** An `invoice_line_item` *must* point to a real `product`. An orphaned row here means you're billing for something that doesn't exist.
- **Inventory:** A `stock_movement` *must* relate to a valid `ingredient` or `product`.
- **Recipes:** A `recipe_step` *must* belong to a real `recipe`.

For your core business (recipes, products, ingredients, orders), you should **always** default to strict foreign keys. The risk of data corruption from polymorphism is too high for data that runs your company.

### A "Best of Both Worlds" Alternative

There is a third, more robust pattern that gives you a single "child" table *and* full referential integrity. It's often called a **Supertype Table**.

Instead of `Comments` being polymorphic, you create a new table, let's call it `Commentable`.

1. A `Post` is created. At the same time, a corresponding row in the `Commentable` table is created. The `Post` table has a `commentable_id` foreign key.
2. A `Video` is created. It also gets a row in `Commentable` and stores the `commentable_id`.
3. Your `Comments` table now just has a standard `commentable_id` foreign key that points *only* to the `Commentable` table.

**Schema:**

- `commentables` (id)
- `posts` (id, ..., `commentable_id` [FK to `commentables.id`])
- `videos` (id, ..., `commentable_id` [FK to `commentables.id`])
- `comments` (id, ..., `commentable_id` [FK to `commentables.id`])

Pros: Full database-level referential integrity. A Comment cannot be orphaned.

Cons: It adds one level of indirection and a little more application logic (you have to create two records instead of one).