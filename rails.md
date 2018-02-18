
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
class CreateProducts < ActiveRecord::Migration[5.0]
  def change
    create_table :products do |t|
      t.string :name
      t.text :description   # field_type :field_name
 
      t.timestamps          # timestamps includes created_at and updated_at
    end
  end
end

# withdraw
class ChangeProductsPrice < ActiveRecord::Migration[5.0]
  def change
    reversible do |dir|
      change_table :products do |t|
        dir.up   { t.change :price, :string }
        dir.down { t.change :price, :integer }
      end
    end
  end
end

or:
class ChangeProductsPrice < ActiveRecord::Migration[5.0]
  def up
    change_table :products do |t|
      t.change :price, :string
    end
  end
 
  def down
    change_table :products do |t|
      t.change :price, :integer
    end
  end
end

# bin/rails generate migration XXXX xxxxxx, detail in docs
# https://ruby-china.github.io/rails-guides/active_record_migrations.html
# http://web.siwei.me/migration.html
```

### Callback

```
class User < ApplicationRecord
  validates :login, :email, presence: true
 
  before_validation :ensure_login_has_a_value
  
  after_validation :set_location, on: [ :create, :update ]
 
  private
    def ensure_login_has_a_value
      if login.nil?
        self.login = email unless email.blank?
      end
    end
end

# after_save is always after after_create and after_update
# after_find before after_initialize
```

### Association

```
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end
 
class Book < ApplicationRecord
  belongs_to :author
end

@book = @author.books.create(published_at: Time.now)  # auto assign author_id
@author.destroy                                       # also destroy all author's books

=============================

# belongs_to                   one to one,  B belongs_to A, B table have A_id foreign key 

=============================

# has_one                      one to one,  A has_one B

=============================

# has_many                     one to many, A has_many B,   B table have A_id foreign key

=============================

# has_many :through            many to many

class Physician < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end
 
class Appointment < ApplicationRecord
  belongs_to :physician
  belongs_to :patient
end
 
class Patient < ApplicationRecord
  has_many :appointments
  has_many :physicians, through: :appointments
end

---------------------------

class Document < ApplicationRecord
  has_many :sections
  has_many :paragraphs, through: :sections
end
 
class Section < ApplicationRecord
  belongs_to :document
  has_many :paragraphs
end
 
class Paragraph < ApplicationRecord
  belongs_to :section
end

# @document.paragraphs will working, simplify nest normal has many

=============================

# has_one  :through            one to one by third modal

class Supplier < ApplicationRecord
  has_one :account
  has_one :account_history, through: :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
  has_one :account_history
end
 
class AccountHistory < ApplicationRecord
  belongs_to :account
end

=============================

# has_and_belongs_to_many      auto many to many, not need third modal

class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end
 
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end

# will generate assemblies_parts table, assembly_id and part_id 

```
