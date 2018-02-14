
## Active Record

### Rules

```
class      table
=====================
Article    articles
LineItem   line_items
Deer       deers
Mouse      mice
Person     people

default: id                     is primary key
         TableName_id           is foreign key
         created_at
         updated_at
         lock_version           乐观锁, 悲观锁
         type                   单表继承
         (associationName)_type 存储多态关联的类型
         (tableName)_count      缓存所关联对象的数量。一个 Article 多个 Comment，comments_count 列存储文章现有评论数
```

### Create AR

```
class Product < ApplicationRecord  # table: products
    # do sth
    self.table_name = "custom_tableName"
    self.primary_key = "custom_primaryId"
end

if products is:
  CREATE TABLE products (
     id int(11) NOT NULL auto_increment,
     name varchar(255),
     PRIMARY KEY  (id)
  );

then:
  p = Product.new
  p.name = "Some Book"
  puts p.name # "Some Book"
```

### CRUD

```

C:

user = User.create(name: "David", occupation: "Code Artist")  # 创建新对象，并将其存入数据库

user = User.new                                               # 只创建新对象
user.name = "David"
user.occupation = "Code Artist"

user = User.new do |u|
  u.name = "David"
  u.occupation = "Code Artist"
end

R:

users = User.all
user = User.first
david = User.find_by(name: 'David')
users = User.where(name: 'David', occupation: 'Code Artist').order(created_at: :desc)

U:

user = User.find_by(name: 'David')
user.name = 'Dave'
user.save

user = User.find_by(name: 'David')
user.update(name: 'Dave')

// multiple row update
User.update_all "max_login_attempts = 3, must_change_password = 'true'"

D:

user = User.find_by(name: 'David')
user.destroy

```

### Validation

```
class User < ApplicationRecord
  validates :name, presence: true
end
 
user = User.new
user.save  # => return false
user.save! # => raise ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

### Migration

```
@TODO...
```
