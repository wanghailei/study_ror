# Rename

### 如何重命名一个controller

Renaming files manually, giving confidence in knowing what’s created, listing controller file, views, routes, tests, helpers, assets. Let’s detail:

##### Step 1: Rename controllers

Rename controller file from `app/controllers/users_controller.rb` to `profiles_controller.rb`.
Open, change class from class `UsersController` to class `ProfilesController`.

##### Step 2: Rename views and partials

Move views directory `app/views/users/` to `app/views/profiles/`.
Update partials (e.g., _form.html.erb) to reference new paths if needed (e.g., `render 'profiles/form'`).

##### Step 3: Edit routes

Edit `config/routes.rb`, change `resources :users` to `resources :profiles`, update custom routes (e.g., `get '/users', to: 'users#index'` to `get '/profiles', to: 'profiles#index'`).

##### Step 4: Rename helpers

Rename `app/helpers/users_helper.rb` to `app/helpers/profiles_helper.rb`. 
Update module (e.g., `module UsersHelper` to `module ProfilesHelper`), ensure views reference.

##### Step 5: Rename tests

e.g., `test/controllers/users_controller_test.rb` to `profiles_controller_test.rb`, update class names inside, adjust test cases.

