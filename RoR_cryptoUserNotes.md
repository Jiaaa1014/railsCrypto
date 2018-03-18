# 前置作業

使用到的檔案
```
app/config/environments/development.rb
app/config/environments/production.rb
app/views/home/_flash.html.erb
app/views/cryptos/index.html.erb
app/controllers/cryptos_controller.rb
app/models/user.rb
app/models/crypto.rb
```
點進[here](rubygems.org)，找`devise`，複製程式碼放到`Gemfiles`
點進[here](https://github.com/plataformatec/devise)，參考文檔
```bash
$ bundle install
$ rails generate devise:install
```
第二個指令完成會有要你做的事情1. ~ 4.

1. `development.rb`，到下方放

```rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
2. `_flash.html.erb`

```rb
<% flash.each do |name, msg| %>
  <div class="alert alert-warning alert-dimissible">
    <button class="close" data-dismiss="alert">
      <i class="glyphicon glyphicon-remove-circle"></i>
    </button>
    <%= content_tag(:div, msg) %>
  </div>
<% end %>
```
3. 
然後到`application.html.erb`放入，你的user會在每一頁如同navbar出現摟～
```rb
<div class="container">
  <%= render 'home/flash' %>
```  

4. 執行`$ rails g devise:views`
在app/views除了上一段教學的`home`資料夾，也有一連串的`divise`資料夾

5. 
`$ rails generate devise MODEL`，這個指令會布置好資料庫
`MODEL`可以是`admin`or`user`

`$ rails db:migrate`不可用，因為`rails`是version 5，改成`rake`，產生`schema.rb`


#### 加入到navbar

login/logout/signup/editProfile丟到`_navbar.html.erb`

但很直覺地遇到情況是，如果已經登入狀態，只需要logout/editProfile的選項。
尚未登入狀態也只需要login/signup選項
```rb
<% if user_signed_in? %>
  <li><%= link_to "Log Out", destroy_user_session_path, method: :delete %></li>
  <li><%= link_to "Edit Profile", edit_user_registration_path %></li>
<% else %>
  <li><%= link_to "Sign Up", new_user_registration_path %></li>
  <li><%= link_to "Log In", new_user_session_path %></li>
<% end %>
```


#### 新增
`$ rails g scaffold crypto symbol:string user_id:integer:index cost_per:decimal amount_owned:decimal`
`user_id`是索引
然後`db/migrate/[時間]_create_crypto.rb`有這些
```rb
class CreateCryptos < ActiveRecord::Migration
  def change
    create_table :cryptos do |t|
      t.string :symbol
      t.integer :user_id
      t.decimal :cost_per
      t.decimal :amount_owned

      t.timestamps null: false
    end
    add_index :cryptos, :user_id
  end
end
```

`$ rake db:migrate`，`schema.rb`會合併剛剛的東西，

因此database就是不停create and push the migration的循環


#### 條件

新增貨幣的動作下由於尚未登入所以會出現錯誤，因為表單會有`value: current_user.id`，更嚴謹的判斷：

`cryptos_controller.rb`，加入

```rb
  before_action  :authenticate_user!
```

當然在新增貨幣時，就會跳轉到Log 



#### Associations: 自己的交易自己看

[here](http://guides.rubyonrails.org/association_basics.html#the-types-of-associations)

`user.rb`

```rb
  has_many :cryptos
end
```

`crypto.rb`
```rb
  belong_to :user
end
```

`app/views/cryptos/index.html.erb`的<tbody>寫條件
```rb
<tbody>
  <% if crypto.user_id == current_user.id %>
  ...
  <% end %>
</tbody>
```


自己的交易紀錄就專屬於自己的了。還有一個問題是user_id不是在我們新增紀錄時需要的資料，把表單隱藏
```rb
<%= f.number_field :user_id, value: current_user.id, type: "hidden" %>
```
html為
```html
<input value="1" name="crypto[user_id]" id="crypto_user_id" type="hidden">
<!-- value是user的編號 -->
```
#### 隱私
如果url輸入
https://crypto-linao264590000.c9users.io/cryptos/2
這個剛好是另一個user的交易紀錄，要避免偷看到別人的紀錄

`cryptos_controller.rb`
```rb
before_action :correct_user, only: [:edit, :update, :destroy, :show]
  def correct_user
    @correct = current_user.cryptos.find_by(id: params[:id])
    redirect_to cryptos_path, notice: "非本人無法查看此筆資料" if @correct.nil?
  end
```
當前使用者的cryptos表格，找看看有沒有這個id