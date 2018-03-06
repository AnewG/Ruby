
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

# Association Callbacks

class Author < ApplicationRecord
  has_many :books, before_add: :check_credit_limit
  # multi callback ===> before_add: [:check_credit_limit, :calculate_shipping_charges]
 
  def check_credit_limit(book)
    # ...
  end
end

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

=============================

# polymorphic association

class Picture < ApplicationRecord
  belongs_to :imageable, polymorphic: true
end
 
class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end
 
class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end

# @employee.pictures
# @product.pictures
# @picture.imageable will get parent modal

=============================

# self joins

class Employee < ApplicationRecord
  has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"
  belongs_to :manager,    class_name: "Employee"
end

# @employee.subordinates, @employee.manager

```

### Query

```
# Lock

      Optimistic lock
          table need lock_version field

          ActiveRecord::Base.lock_optimistically = false # off optimistic lock

          class Client < ApplicationRecord
              self.locking_column = :lock_client_column    # change default lock field name
          end
          
      Pessimistic lock
          
          Item.transaction do
              i = Item.lock.first
              i.name = 'Jones'
              i.save!
          end
          
          will generate SQL:
              BEGIN
              SELECT * FROM `items` LIMIT 1 FOR UPDATE
              UPDATE `items` SET `updated_at` = '2009-02-07 18:05:56', `name` = 'Jones' WHERE `id` = 1
              COMMIT
              
          # specific lock type
          Item.transaction do
              i = Item.lock("LOCK IN SHARE MODE").find(1)
              i.increment!(:views)
          end
          
          item = Item.first
              item.with_lock do
              # 这个块在事务中调用, item 已经锁定
              item.increment!(:views)
          end
          
# Join

class Category < ApplicationRecord
  has_many :articles
end
 
class Article < ApplicationRecord
  belongs_to :category
  has_many   :comments
  has_many   :tags
end
 
class Comment < ApplicationRecord
  belongs_to :article
  has_one    :guest
end
 
class Guest < ApplicationRecord
  belongs_to :comment
end
 
class Tag < ApplicationRecord
  belongs_to :article
end

=====================

Category.joins(:articles)  

SELECT categories.* FROM categories
  INNER JOIN articles ON articles.category_id = categories.id

----------------------

Article.joins(:category, :comments)

SELECT articles.* FROM articles
  INNER JOIN categories ON articles.category_id = categories.id
  INNER JOIN comments ON comments.article_id = articles.id

----------------------

Article.joins(comments: :guest)

SELECT articles.* FROM articles
  INNER JOIN comments ON comments.article_id = articles.id
  INNER JOIN guests ON guests.comment_id = comments.id

----------------------

Category.joins(articles: [{ comments: :guest }, :tags])

SELECT categories.* FROM categories
  INNER JOIN articles ON articles.category_id = categories.id
  INNER JOIN comments ON comments.article_id = articles.id
  INNER JOIN guests ON guests.comment_id = comments.id
  INNER JOIN tags ON tags.article_id = articles.id

----------------------

Author.left_outer_joins(:posts).distinct.select('authors.*, COUNT(posts.*) AS posts_count').group('authors.id')

SELECT DISTINCT authors.*, COUNT(posts.*) AS posts_count FROM "authors"
LEFT OUTER JOIN posts ON posts.author_id = authors.id GROUP BY authors.id
```

## Views

```
.erb     : ERB 模板（含有嵌入式 Ruby 代码的 HTML）的标准扩展名 
.builder : Builder 模板（XML 生成器）的标准扩展名

# render

render "edit"         | render :edit         | render "other/show"   |   render file: "/xxx/xxx"

render json: @product | render xml: @product | render js: "alert('Hello Rails');"

layout:
    default rule is: PhotosController => app/views/layouts/photos.html.erb
    if no exist => app/views/layouts/application.html.erb
    
    class ProductsController < ApplicationController
        layout "inventory"                                                                 # specific layout
        layout Proc.new { |controller| controller.request.xhr? ? "popup" : "application" } # dynamic layout
        # render 的内容将在 layout 的 yield 处输出, 有多个 yield 用 content_for
    end
    
    # 局部视图文件名已下划线开头
    # 在 ERB 文件中 <%= render "xx" %>
    # 传值到局部视图的 yield 中并传参
    <%= render "shared/search_filters", search: @q do |f| %> # get local param can use local_assigns[:xxx]
      <p>
        Name contains: <%= f.text_field :name_contains %>
      </p>
    <% end %>

# redirect_to

redirect_to photos_url, status: 301  

redirect_back(fallback_location: root_path)

# head

head :bad_request

will generate:
  HTTP/1.1 400 Bad Request
  Connection: close
  Date: Sun, 24 Jan 2010 12:15:53 GMT
  Transfer-Encoding: chunked
  Content-Type: text/html; charset=utf-8
  X-Runtime: 0.013483
  Set-Cookie: _blog_session=...snip...; path=/; HttpOnly
  Cache-Control: no-cache
```

### Form

```
<%= form_tag do %>
  Form contents
<% end %>

default will generate:

<form accept-charset="UTF-8" action="/" method="post">    # default post
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input name="authenticity_token" type="hidden" value="J7CBxfHalt49OSHp27hblqK20c9PgwJ108nDHX/8Cts=" />
  Form contents
</form>

----------------------

more detail:

<%= form_tag("/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>

<form accept-charset="UTF-8" action="/search" method="get">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <label for="q">Search for:</label>
  <input id="q" name="q" type="text" />
  <input name="commit" type="submit" value="Search" />
</form>

----------------------

by modal, @person.name() will be:

<%= text_field(:person, :name) %>

<input id="person_name" name="person[name]" type="text" value="Henr

----------------------

bind object to form:

def new
  @article = Article.new
end

<%= form_for @article, url: {action: "create"}, html: {class: "nifty_form"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>

<form accept-charset="UTF-8" action="/articles" method="post" class="nifty_form">
  <input id="article_title" name="article[title]" type="text" />
  <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
  <input name="commit" type="submit" value="Create" />
</form>

----------------------

fields_for (bind object to field):

<%= form_for @person, url: {action: "create"} do |person_form| %>
  <%= person_form.text_field :name %>
  <%= fields_for @person.contact_detail do |contact_detail_form| %>
    <%= contact_detail_form.text_field :phone_number %>
  <% end %>
<% end %>

<form accept-charset="UTF-8" action="/people" class="new_person" id="new_person" method="post">
  <input id="person_name" name="person[name]" type="text" />
  <input id="contact_detail_phone_number" name="contact_detail[phone_number]" type="text" />
</form>
```

### Upload

```
<%= form_tag({action: :upload}, multipart: true) do %>
  <%= file_field_tag 'picture' %>
<% end %>
# params[:picture]

or

<%= form_for @person do |f| %>
  <%= f.file_field :picture %>
<% end %>
# params[:person][:picture]


def upload
  uploaded_io = params[:person][:picture]
  File.open(Rails.root.join('public', 'uploads', uploaded_io.original_filename), 'wb') do |file|
    file.write(uploaded_io.read)
  end
end
# put in {Rails.root}/public/uploads
```

## Controller

```
Rails 控制器的命名约定是，最后一个单词使用复数形式，但也有例外，控制器的命名约定与模型不同，模型的名字习惯使用单数形式。

class ClientsController < ApplicationController
  def new
    # GET /clients?ids[]=1&ids[]=2&ids[]=3 ====> params[:ids] is ["1", "2", "3"]
    if params[:status] == "activated"
      @clients = Client.activated
    else
      @clients = Client.inactivated
    end
  end
  
  def default_url_options    # rails internal use
    { locale: I18n.locale }
  end
end
```

### params

```
白名单
params.permit(:xx)
params.permit(xx: [])         # array
params.permit(xx: {})         # hash
params.require(:xx).permit!   # all permit
```

### session

```
ActionDispatch::Session::CookieStore
ActionDispatch::Session::CacheStore
ActionDispatch::Session::ActiveRecordStore

session[:current_user_id] = user.id
session[:current_user_id] = nil
reset_session

Flash Data:
    flash[:notice] = "You have successfully logged out." # will display on next request
    flash.now[:error] = "Could not save client"          # will display on current page

    or with redirect

    redirect_to root_url, notice: "You have successfully logged out."
    redirect_to root_url, alert: "You're stuck here!"
    redirect_to root_url, flash: { referral_code: 1234 }

    <% flash.each do |name, msg| -%>
      <%= content_tag :div, msg, class: name %>
    <% end -%>

    class MainController < ApplicationController
      def index
        flash.keep           # 从别处重定向来后持久存储所有闪现
        flash.keep(:notice)  # 还可以指定一个键，只保留某种闪现
        redirect_to users_url
      end
    end
```

### cookies

```
cookies[:commenter_name] = @comment.author
cookies.delete(:commenter_name)
```

### filter

```
class ApplicationController < ActionController::Base
  before_action :require_login
  # skip_before_action :require_login, only: [:new, :create], will skip new and create method 
  # around_action xxxxx
  private
 
  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end
```

### request

```
params             --> all
query_parameters   --> query_string
request_parameters --> POST
path_parameters    --> route
```

### response

```
response.headers["Content-Type"] = "application/pdf"
```

### auth

```
# Basic
class AdminsController < ApplicationController
  http_basic_authenticate_with name: "humbaba", password: "5baa61e4"
end

# Digest
# authenticate_or_request_with_http_digest 接受一个参数用户名，返回值是密码。
# authenticate_or_request_with_http_digest 返回 false 或 nil，表明身份验证失败。
class AdminsController < ApplicationController
  USERS = { "lifo" => "world" }
 
  before_action :authenticate
 
  private
    def authenticate
      authenticate_or_request_with_http_digest do |username|
        USERS[username]
      end
    end
end
```

### download

```
class ClientsController < ApplicationController
  def download_pdf
    client = Client.find(params[:id])
    send_file("#{Rails.root}/files/clients/#{client.id}.pdf",
              filename: "#{client.name}.pdf",
              type: "application/pdf")
  end
end
```

### https

```
class DinnerController
  force_ssl
  force_ssl only: :cheeseburger
  force_ssl except: :cheeseburger
end

all https: config.force_ssl
```

## Routes

```
get '/patients/:id', to: 'patients#show'                  # patients.show(id), or to: :show, controller: 'patients'
get '/patients/:id', to: 'patients#show', as: 'patient'   # @patient = Patient.find(17), in html patient_path(@patient)

DELETE /photos/17 ==> resources :photos ==> photos.destroy({ id: '17' })

resources :photos will generate
GET          /photos              photos#index          显示所有照片的列表
GET          /photos/new          photos#new            返回用于新建照片的 HTML 表单
POST         /photos              photos#create         新建照片
GET          /photos/:id          photos#show           显示指定照片
GET          /photos/:id/edit     photos#edit           返回用于修改照片的 HTML 表单
PATCH/PUT    /photos/:id          photos#update         更新指定照片
DELETE       /photos/:id          photos#destroy        删除指定照片

photos_path          ==> /photos
new_photo_path       ==> /photos/new
edit_photo_path(:id) ==> /photos/:id/edit
photo_path(:id)      ==> /photos/:id

*_path return path, *_url return host.port.prefix

----------------------

resource :geocoder will generate (Singular Resources)
GET          /geocoder/new        geocoders#new         返回用于创建 geocoder 的 HTML 表单
POST         /geocoder            geocoders#create      新建 geocoder
GET          /geocoder            geocoders#show        显示唯一的 geocoder 资源
GET          /geocoder/edit       geocoders#edit        返回用于修改 geocoder 的 HTML 表单
PATCH/PUT    /geocoder            geocoders#update      更新唯一的 geocoder 资源
DELETE       /geocoder            geocoders#destroy     删除 geocoder 资源

new_geocoder_path    ==> /geocoder/new
edit_geocoder_path   ==> /geocoder/edit
geocoder_path        ==> /geocoder

----------------------

Admin:: , app/controllers/admin

namespace :admin do
  resources :articles, :comments
end

will generate

GET          /admin/articles              admin/articles#index       admin_articles_path
GET          /admin/articles/new          admin/articles#new         new_admin_article_path
POST         /admin/articles              admin/articles#create      admin_articles_path
GET          /admin/articles/:id          admin/articles#show        admin_article_path(:id)
GET          /admin/articles/:id/edit     admin/articles#edit        edit_admin_article_path(:id)
PATCH/PUT    /admin/articles/:id          admin/articles#update      admin_article_path(:id)
DELETE       /admin/articles/:id          admin/articles#destroy     admin_article_path(:id)

scope module: 'admin' do
  resources :articles, :comments
end
will route /articles(without /admin) to Admin::Articles, or resources :articles, module: 'admin'

scope '/admin' do
  resources :articles, :comments
end
will route /admin/articles to Articles(without Admin::), or resources :articles, path: '/admin/articles'

----------------------

resources :magazines do
  resources :ads
end

GET          /magazines/:magazine_id/ads             ads#index    显示指定杂志的所有广告的列表
GET          /magazines/:magazine_id/ads/new         ads#new      返回为指定杂志新建广告的 HTML 表单
POST         /magazines/:magazine_id/ads             ads#create   为指定杂志新建广告
GET          /magazines/:magazine_id/ads/:id         ads#show     显示指定杂志的指定广告
GET          /magazines/:magazine_id/ads/:id/edit    ads#edit     返回用于修改指定杂志的广告的 HTML 表单
PATCH/PUT    /magazines/:magazine_id/ads/:id         ads#update   更新指定杂志的指定广告
DELETE       /magazines/:magazine_id/ads/:id         ads#destroy  删除指定杂志的指定广告

----------------------

scope shallow_path: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end

GET        /articles/:article_id/comments(.:format)       comments#index     article_comments_path
POST       /articles/:article_id/comments(.:format)       comments#create    article_comments_path
GET        /articles/:article_id/comments/new(.:format)   comments#new       new_article_comment_path
GET        /sekret/comments/:id/edit(.:format)            comments#edit      edit_comment_path
GET        /sekret/comments/:id(.:format)                 comments#show      comment_path
PATCH/PUT  /sekret/comments/:id(.:format)                 comments#update    comment_path
DELETE     /sekret/comments/:id(.:format)                 comments#destroy   comment_path

scope shallow_prefix: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end

GET        /articles/:article_id/comments(.:format)       comments#index      article_comments_path
POST       /articles/:article_id/comments(.:format)       comments#create     article_comments_path
GET        /articles/:article_id/comments/new(.:format)   comments#new        new_article_comment_path
GET        /comments/:id/edit(.:format)                   comments#edit       edit_sekret_comment_path
GET        /comments/:id(.:format)                        comments#show       sekret_comment_path
PATCH/PUT  /comments/:id(.:format)                        comments#update     sekret_comment_path
DELETE     /comments/:id(.:format)                        comments#destroy    sekret_comment_path

```

### concern

```
concern :commentable do
  resources :comments
end
concern :image_attachable do
  resources :images, only: :index
end
resources :messages, concerns: :commentable
resources :articles, concerns: [:commentable, :image_attachable]

will generate: ==========>

resources :messages do
  resources :comments
end
resources :articles do
  resources :comments
  resources :images, only: :index
end

http://rubyer.me/blog/583/
```

### other

```
get 'photos(/:id)', to: :display               
# /photos/1   => Photos.display(id), /photos will exec because id is option

get 'photos/:id/:user_id', to: 'photos#show'   
# /photos/1/2 => Photos.show, params[:id] is "1", params[:user_id] is "2"

get 'photos/:id/with_user/:user_id', to: 'photos#show'  
# /photos/1/with_user/2, params is { controller: 'photos', action: 'show', id: '1', user_id: '2' }

get 'photos/:id', to: 'photos#show'
# /photos/1?user_id=2 => Photos.show, params is { controller: 'photos', action: 'show', id: '1', user_id: '2' }

get 'photos/:id', to: 'photos#show', defaults: { format: 'jpg' }
# /photos/12 => Photos.show, params[:format] is "jpg"

match 'photos', to: 'photos#show', via: [:get, :post]    # get, post method
match 'photos', to: 'photos#show', via: :all             # all methods

get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ } or
get 'photos/:id', to: 'photos#show', id: /[A-Z]\d{5}/
# will constraint id by regex
```
