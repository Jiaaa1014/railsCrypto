
#### PostSQLgre
`Gemfile`底部置入
```rb
group :production do
  gem 'pg'
  gem 'rails_12factor', '~> 0.0.3'
end
```
並且移`gem 'sqlite3'`到
```rb
group :development, :test do
  gem 'byebug'
  gem 'sqlite3'
end
group :development do
  gem 'web-console', '~> 2.0'
  gem 'sqlite3'
  gem 'spring'
end
```
在dev模式用SQLite3，`Heroku`用的是`PostgreSQL`


####
```
$ bundle install --without production
$ heroku --version
$ heroku login
$ heroku keys:add
$ heroku create
```
`$ git push heroku master`時，出現錯誤

`pg`版本
```rb
group :production do
  gem 'pg', '~> 0.21.0'
  gem 'rails_12factor', '~> 0.0.3'
end
```

```
$ bundle install --without production
$ git push heroku master
```
有時候找error不值得，可能過個一小時又可以啦～不要太計較QQ
就當作上一指令成功吧，要把資料庫給`Heroku`使用
```
$ heroku run rake db:migrate
```


到`config/environments/production.rb`最下面
```rb
config.action_mailer.default_url_options = { host: 'https://vast-eyrie-49072.herokuapp.com/'}
```

`host`可以改你想要的，登入Heroku可以看到app有上述的東西，然後`Add Domain`