# 前置作業

* C9 被AWS買下來很難用，兩種入門方式，AWS官方網站很醜不是那個。用`c9.io`搜尋
* johnelder.org/code一些簡易的code

在IDE的command prompt中輸入`$ cat ~/.ssh/id_rsa.hub`

然後把出現的東西丟到github設定SSH key的地方，之後就可以從IDE上傳了，之前做過了吧，在於現在是線上IDE，過去是放在電腦使用者的`.ssh`資料夾

`$ rails g controller home index`
建立一個名為home的controller，將這個行為稱為index

`app/views/home/index.html.erb`，後面是代表embeded ruby


以下為之後會用到的檔案

```
app/assets/stylesheets/bootstraply.css.scss
app/assets/stylesheets/application.css
app/assets/javascripts/application.js
app/controllers/home_controller.rb
app/views/home/about.html.erb
app/views/home/index.html.erb
app/views/home/lookup.html.erb
app/views/layouts/application.html.erb
config/routes.rb
Gemfile
```

#### routes.rb

告知你的route

```rb
Rails.application.routes.draw.do
#get 'home/index'
root 'home#index'
#default頁面是'home/index'
```
[教程](guides.rubyonrails.org/routing.html)


#### about.html.erb

建立個`about.htnl.erb`，當然也要在`routes.rb`設定

```rb
Rails.application.routes.draw.do
get 'home/about'
root 'home#index'
```


`home_controller.rb`定義變數，這個變數只在`index` method裡面用
```rb
class HomeController < ApplicationController
  def index
    @hello = 'hello there'
  end
  def about
  end
end
```
就可以在個別的葉面引入這些變數

#### application.html.erb

打開原始碼有一串都不屬於`index.html.erb`？

有`<%= yield %>`，說明我都從`index.html.erb`拿東西
```rb
<body>
Hello
<%= yield %>
</body>
```
那麼之後其他的html檔案顯示出來都會有Hello字樣，因此可以應用於網頁中固定的地方，像是navbar。


#### 延續上面

```html
<a herf="/">Home</a> | <a herf="/home/about">About</a> 
```

不管用，不會是可以點擊的連結，要寫成這樣
`$ rake routes`查看要怎麼寫link後面的path

```rb
<%= link_to "Home", root_path %>
<%= link_to "About", home_about_path %>
```

如果我改了`about.html.erb`為`aboutus.html.erb`

1. 要順便改哪個檔案？
2. 如果有在`home.controller.rb`定義變數的話呢？

Ans:
1. `routes.rb`，輸入指令`$ rake routes`發現GET後面的路徑改惹，當然也要把`application.html.erb`的link中的path改一下嚕！
2. 也要一併改成`def aboutus`，但是就算不正確也沒差@@，讀不到而已，但不會警告你


#### _navbar.html.erb

把`application.html.erb`剛剛定義的`<link>`丟到獨立的檔案裏面，再引入就好
```rb
<%= render 'home/navbar' %>
```

#### 加入Bootstrap，Gemfile

點進[here](rubygems.org)，找`bootstrap sass`，複製程式碼放到`Gemfiles`
`$ bundle install`


`bootstraply.css.scss`，放入
```scss
@import "bootstrap-sprockets";
@import "bootstrap";
```

`application.js`，放入
```js
//= require jquery
//= require bootstrap-sprockets
```

#### 建造bootstrap的navbar

`application.html.erb`的`<head>`放入
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

#### api，正題開始

`home.controller.rb`
```rb
  def index
    require 'net/http'
    require 'json'
    @url = 'https://api.coinmaketcap.com/v1/ticker/'
    @uri = URI(@url)
    
    #到這個Website
    @response = Net::HTTP.get(@uri)
    #data
    @coins = JSON.parse(@response)
  end
```

`@response`回傳資料為json，而`@coins`將`:`表示為`=>`，轉換為ruby用的格式

```rb
{
  "id"=>"bitcoin",
  "name"=>"Bitcoin",
  "symbol"=>"BTC",
  ...(略),
  "percent_change_24h"=>"-11.87",
  "percent_change_7d"=>"-17.37",
  "last_updated"=>"1521074965"
}
```

Hint: if `<%= @variable%>` is without "=" sign，it doesn't show up.

#### for loop
[`===` and `==` difference?](https://stackoverflow.com/questions/7156955/whats-the-difference-between-equal-eql-and)
```rb
<% @coins %>
<% for x in @coins%>
    <% for coin in @my_coins %>
        <% if coin == x["symbol"] %>
            <%= x["name"] %><br/>
        <% end %>
    <% end %>
<% end %>
```

#### lookup.html.erb

```rb
text_field_tag(name, value = nil, options = {})
```

```rb
<%= form_tag home_lookup_path, :method => 'POST' do %>
    <%= text_field_tag 'sym', nil, placeholder:'Enter Crypto Symbol', size:50 %>
    <%= submit_tag 'Lookup Stock Quote' %>
<% end %>
```
變成

```html
<form action="/home/lookup" accept-charset="UTF-8" method="post">
    <input name="sym" id="sym" placeholder="Enter Crypto Symbol" size="50" type="text">
    <input name="commit" value="Lookup Stock Quote" type="submit">
</form>
```


在`routes.rb`，post到原先畫面
```rb
  get 'home/lookup'
  post 'home/lookup' => 'home/lookup'
```

在`home_controller.rb`定義變數
```rb
  def lookup
    
    require "net/http"
    require "json"
    @url = "https://api.coinmarketcap.com/v1/ticker/"
    @uri = URI(@url)
    @response = Net::HTTP.get(@uri)
    @lookup_coin = JSON.parse(@response)
  
  
    @symbol = params[:sym]
  
    if @symbol 
      @symbol = @symbol.upcase
    end
    
    if @symbol == ''
      @symbol = 'Hey Type Something'
    end
  end
```

當然`lookup.html.erb`
```rb
<% if @symbol %>
    <% for x in @lookup_coin %>
        <% if @symbol == x["symbol"] %>
            <%= x["name"] %>:  $<%= x["price_usd"] %><br/>
            Rank: <%= x["rank"] %><br/><br/>
        <% end %>
    <% end %>
<% end %>
```
搜尋某個貨幣名字出現在API的json資料裡就可以顯示目前價格。