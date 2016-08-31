# F(x)

F(x) adds methods to `ActiveRecord::Migration` to create and manage database
functions and triggers in Rails.

Using F(x), you can bring the power of SQL functions and triggers to your Rails
application without having to switch your schema format to SQL. F(x) provides
a convention for versioning functions and triggers that keeps your migration
history consistent and reversible and avoids having to duplicate SQL strings
across migrations. As an added bonus, you define the structure of your function
in a SQL file, meaning you get full SQL syntax highlighting in the editor of
your choice and can easily test your SQL in the database console during
development.

F(x) ships with support for PostgreSQL. The adapter is configurable (see
`Fx::Configuration`) and has a minimal interface (see
`Fx::Adapters::Postgres`) that other gems can provide.

## Great, how do I create a trigger and a function?

You've got this great idea for a trigger you'd like to call
`uppercase_users_name`. You can create the migration and the corresponding
definition file with the following command:

```sh
% rails generate fx:trigger uppercase_users_name
      create  db/triggers/uppercase_users_name_v01.sql
      create  db/migrate/[TIMESTAMP]_create_trigger_uppercase_users_name.rb
```

Edit the `db/triggers/uppercase_users_name_v01.sql` file with the SQL statement
that defines your trigger. In our example, this might look something like this:

```sql
CREATE TRIGGER uppercase_users_name
    BEFORE INSERT ON users
    FOR EACH ROW
    EXECUTE PROCEDURE uppercase_users_name();
```

As you see, we execute a function called `uppercase_users_name` before each
`INSERT` on the `users` table, which is a function we don't have yet.

```sh
% rails generate fx:function uppercase_users_name
      create  db/functions/uppercase_users_name_v01.sql
      create  db/migrate/[TIMESTAMP]_create_function_uppercase_users_name.rb
```

The generated migrations contains `create_function` and `create_trigger`
statements. The migration is reversible and the schema will be dumped into your
`schema.rb` file.

```sh
% rake db:migrate
```

## Cool, but what if I need to change a trigger or function?

Here's where F(x) really shines. Run that same function generator once more:

```sh
% rails generate fx:function uppercase_users_name
      create  db/functions/uppercase_users_name_v02.sql
      create  db/migrate/[TIMESTAMP]_update_function_uppercase_users_name_to_version_2.rb
```

F(x) detected that we already had an existing `uppercase_users_name` function at
version 1, created a copy of that definition as version 2, and created a
migration to update to the version 2 schema. All that's left for you to do is
tweak the schema in the new definition and run the `update_function` migration.

## I don't need this trigger or function anymore. Make it go away.

F(x) gives you `drop_trigger` and `drop_function` too:

```ruby
def change
  drop_function :uppercase_users_name, revert_to_version: 2
end
```
