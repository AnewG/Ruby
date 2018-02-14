
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
