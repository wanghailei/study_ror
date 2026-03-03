# Seed Data

To add initial data after a database is created, Rails has a built-in 'seeds' feature that speeds up the process. This is especially useful when reloading the database frequently in development and test environments, or when setting up initial data for production.

To get started with this feature, open up `db/seeds.rb` and add some Ruby code, then run `bin/rails db:seed`.

The code here should be idempotent so that it can be executed at any point in every environment.

```ruby
["Action", "Comedy", "Drama", "Horror"].each do |genre_name|
      MovieGenre.find_or_create_by!(name: genre_name)
end
```

This is generally a much cleaner way to set up the database of a blank application.

### % Using gem seed\_dump %

https://github.com/rroblak/seed_dump

Add it to your Gemfile with: `gem 'seed_dump'`

```ruby
#  Dump all data directly to db/seeds.rb:
rake db:seed:dump

# Append to db/seeds.rb instead of overwriting it:
rake db:seed:dump APPEND=true

# Dump only data from the users table and dump a maximum of 1 record:
$ rake db:seed:dump MODELS=User LIMIT=1

# Use another output file instead of db/seeds.rb:
rake db:seed:dump FILE=db/seeds/users.rb
```

