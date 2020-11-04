---
title: "Have Custom Attribute for a Rails ActiveRecord Model"
date: 2020-08-03T17:00:19+08:00
tags: [ruby,ruby on rails]
categories: [engineering]
---

Framework is pretty opinionated in its area, but that also means it handles many repetitive and tedious tasks. When using ORM like ActiveRecord, it saved us so much time defining which model has what attributes. The convention from ActiveRecord design is that all the columns of a table, which is a model in the app, are attributes of the model.

For example, let's imagine we have a User model and the table looks like below.
Please note that, the example below is not an ideal case but just an demonstration.

```SQL
> desc users;
+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| id         | int(11)      | NO   | PRI | NULL    | auto_increment |
| username   | varchar(255) | YES  | MUL | NULL    |                |
| first_name | varchar(255) | YES  |     | NULL    |                |
| last_name  | varchar(255) | YES  |     | NULL    |                |
+------------+--------------+------+-----+---------+----------------+

> SELECT * FROM users WHERE id = 3;
+----+----------+------------+-----------+
| id | username | first_name | last_name |
+----+----------+------------+-----------+
|  3 | pfrank   | Paul       | Frank     |
+----+----------+------------+-----------+
```

```ruby
class User < ActiveRecord::Base
end
```

```ruby
> user = User.find_by(id: 3)
# SELECT * FROM users WHERE id = 3

> user.username
"pfrank"
```

If we want to access user's `full_name` which is sipmly composed by `first_name` and `last_name`, the most fastest way is just adding a new method with string concatenation since we've have those attributes. So our `User` class becomes.

```ruby
class User < ActiveRecord::Base

  def full_name
    "#{first_name} #{last_name}"
  end
end
```

```ruby
> user.full_name
"Paul Frank"
```

But... we could do this with database query too, right?

```SQL
> SELECT *, CONCAT_WS(' ', first_name, last_name) AS full_name FROM users WHERE id = 3;
+----+----------+------------+-----------+------------+
| id | username | first_name | last_name | full_name  |
+----+----------+------------+-----------+------------+
|  3 | pfrank   | Paul       | Frank     | Paul Frank |
+----+----------+------------+-----------+------------+
```

If we want to have `full_name` calculated in our database query, we could use `attribute` to define it, then use the query to have the value being mapped from the query result.

```ruby
class User < ActiveRecord::Base
  scope :will_full_name, -> { select('`*`, CONCAT_WS(' ', `first_name`, `last_name`)')}
  attribute :full_name, :string, default: nil
end
```

```ruby
> User.with_full_name.where(id: 3).take
> user.full_name
"Paul Frank"
```

Yeah, the above example looks over-engineered, because it's actually simpler with the first implementation. However, this could help if the data we stored can only be calculated in the database or the processing is much simpler and faster in database level.

---

The use of `attribute` is not just the case above. It's even more powerful than it. Please refer to the [official API document](https://edgeapi.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html) for it.