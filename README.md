% RailsでWebpackerを使おう
% ka
% 2018-07-29

# Author

ka

[![Gravatar](https://gravatar.com/avatar/884be098693425b409d25aaec5091de8?s=150)](https://gravatar.com/ka000)

Website: [kaosfield](http://www.kaosfield.net)

Twitter: [ka](https://twitter.com/ka_)

GitHub: [kaosf](https://github.com/kaosf)

# License

[![CC BY-NC-SA 4.0](https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/4.0/)

Copyright (C) 2018 ka

# このページとリポジトリ

[https://kaosf.github.io/20180729-tokushimarb-slide](https://kaosf.github.io/20180729-tokushimarb-slide)

Repository: [kaosf/20180729-tokushimarb-slide - GitHub](https://github.com/kaosf/20180729-tokushimarb-slide)

# 準備

Dockerのインストール (DBのため)

Node.jsとYarnのインストール (Webpackerのため)

Rubyのインストール (言わずもがな)

# 準備

<ul>
<li>Docker: [https://docs.docker.com/install/#supported-platforms](https://docs.docker.com/install/#supported-platforms)</li>
<li>Node.js: nodebrewとかnodenvとか好きに使うと良いです</li>
  <ul>
  <li>nodebrew: [https://github.com/hokaccha/nodebrew](https://github.com/hokaccha/nodebrew) (私はnodebrew使ってる)</li>
  </ul>
<li>Yarn: [https://yarnpkg.com/lang/ja/docs/install/#arch-stable](https://yarnpkg.com/lang/ja/docs/install/#arch-stable)</li>
</ul>

# DBコンテナ

```sh
sudo docker run -d --restart=always -p 15432:5432 postgres:10.3
```

または

```sh
sudo docker run -d --restart=always -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 13306:3306 mysql:5.7.22
```

これに

<ul>
<li>host: localhost</li>
<li>port: 15432 (MySQLなら 13306)(ここは各自が自由に変える)</li>
<li>user: postgres (MySQLなら root)</li>
<li>password: "" (無し)(空文字列)</li>
</ul>

で接続するのが簡単で環境を汚しません

※SQLite3を使うのは個人でやるもの以外ではよほどの理由が無い限りやめましょう

# Railsプロジェクト

```sh
rails new my_webpacker_app -d postgresql
# or
bundle exec rails new my_sample_app -d mysql
# などなど…
```

# webpacker導入

`Gemfile`に以下を追加して

```
gem 'webpacker', '3.5.0'
```

`Gemfile.lock`更新

```sh
bin/bundle install
# or
bin/bundle install --path vendor/bundle
```

# Railsプロジェクト(初期化時に実は…)

実はRails 5.1からは

```sh
rails new --webpack
```

オプションが使えました(気付かなかった)

と言ってもGemfileの更新までやってくれてるだけっぽいので気にしなくて良いです(多分)

# webpacker導入

```sh
bin/rails webpacker:install
```

時間掛かります

```sh
bin/yarn install
```

時間掛かります

# Bootstrap導入

```sh
bin/yarn add bootstrap
```

`package.json`は手動で編集しない方が良いです

`yarn.lock`も同時に変更されます

# Bootstrap導入

`app/javascript/packs/application.js`を編集

```diff
-console.log('Hello World from Webpacker')
+import '../stylesheet'
```

`app/javascript/stylesheet.scss`を作成

```scss
@import "~bootstrap/scss/bootstrap";
```

なんか変な感じですがWebpackerで扱うCSSは`app/javascript`内に置きます

# rails-ujs復活

```sh
bin/yarn add rails-ujs
```

# rails-ujs復活

`app/javascript/packs/application.js`を以下のように編集します

```diff
 // layout file, like app/views/layouts/application.html.erb

+import Rails from 'rails-ujs'
+Rails.start()
+
 import '../stylesheet'
```

これが無いと例えばRailsがscaffoldで生成するdestroyアクションのあのボタンとかがちゃんと動きません

# webpacker導入

`app/views/layouts/application.html.erb`を編集

```diff
-    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
-    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
+    <%= javascript_pack_tag 'application' %>
+    <%= stylesheet_pack_tag 'application' %>
```

参考 [https://github.com/rails/webpacker#usage](https://github.com/rails/webpacker#usage)

#

これでwebpackerの導入はおしまい

後は

```sh
bin/rails assets:precompile
```

を通常通りに行います

成果物が `public/packs` に出来上がるのでこれを本番環境にコピーしてやります

※本番環境で作らないのか？という理由は後ほど

# Rubyコンテナ

本番稼働時の想定

永続化するものを確保する場所作成

```sh
sudo mkdir -p /var/docker/rails
sudo mkdir /var/docker/rails/bundle
sudo mkdir /var/docker/rails/log
sudo mkdir /var/docker/rails/tmp
sudo mkdir /var/docker/rails/pack
sudo chmod o+w /var/docker/rails/pack
```

※別に `/var/docker` とかで無く `/Users/yourname/foo/bar` でも何処でも良いです

# Rubyコンテナ

本番稼働時の想定

※今のディレクトリ(`$PWD`)がRailsのプロジェクトルートだとします

```sh
sudo docker run -it \
  -v $PWD:/app \
  -v /var/docker/rails/bundle:/usr/local/bundle \
  -v /var/docker/rails/log:/app/log \
  -v /var/docker/rails/tmp:/app/tmp \
  -v /var/docker/rails/pack:/app/public/pack \
  -w /app \
  -p 3000:3000 \
  -e TZ=Asia/Tokyo \
  -e DATABASE_URL=postgres://postgres:@172.17.0.1:15432/db \
  -e RAILS_ENV=production \
  -e RAILS_MASTER_KEY=xxxx \
  -e RAILS_SERVE_STATIC_FILES=1 \
  ruby:2.5.1 bash
```

MySQLの人は

```
  -e DATABASE_URL=mysql2://root:@172.17.0.1:13306/db \
```

に読み替えて下さい

# Rubyコンテナ

`RAILS_MASTER_KEY`には`config/master.key`の値を設定して下さい(※勝手に作られてるはず)

※コンテナから見たホストが `172.17.0.1` になります

※ネットワークを別途構築したりした場合の解説は省略

# Rubyコンテナ

Bundlerがgemをインストールする場所は `/usr/local/bundle` です

参考: [該当箇所](https://github.com/docker-library/ruby/blob/699a04311386ecc98ca242fc9bdee17fb4008863/2.5/alpine3.7/Dockerfile#L100-L110)

```sh
bin/bundle install --path vendor/bundle
```

しても結局 .bundle ディレクトリが作られない(設定が保存されない)ので `--path vendor/bundle` オプションは使わないが吉

gemをインストールした状態を使いまわしたいなら大人しくホストに永続化して `/usr/local/bundle` にマウントしましょう

もしくは Dockerfile でgemのインストールまで終わらせたイメージでも作っておいてプライベートレジストリに上げておくとデプロイは多分これが一番早いと思います

# Yarn

残念ながらRubyのオフィシャルコンテナにはYarnは入っていません

インストールするにはDockerfileに以下を追記すると良いでしょう

```
# Node.js
ENV PATH /root/.nodebrew/current/bin:$PATH
RUN curl -L git.io/nodebrew | perl - setup && \
  nodebrew install-binary v9.11.2 && \
  nodebrew use v9.11.2

# Yarn
RUN apt-get update && apt-get -y install apt-transport-https && \
  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update && apt-get -y install yarn
```

## 参考

<ul>
<li>[https://github.com/hokaccha/nodebrew](https://github.com/hokaccha/nodebrew)</li>
<li>[https://yarnpkg.com/lang/en/docs/install/#debian-stable](https://yarnpkg.com/lang/en/docs/install/#debian-stable)</li>
<li>[http://scribble.washo3.com/linux/debian%E3%81%A7sourcelist%E5%86%85%E3%81%AEhttp...](http://scribble.washo3.com/linux/debian%E3%81%A7sourcelist%E5%86%85%E3%81%AEhttps%E3%81%8C%E5%8F%96%E5%BE%97%E5%87%BA%E6%9D%A5%E3%81%AA%E3%81%84%E3%81%A8%E3%81%8D.html)</li>
</ul>

# CSS, JavaScriptの用意

手元で作っておいた `public/packs/*` を `/var/docker/rails/packs` にコピーすればOKです

```sh
sudo rm /var/docker/rails/packs/*
sudo cp public/packs/* /var/docker/rails/packs
```

# Railsサーバ起動前の開発完了

`db/schema.rb`をきちんと用意しておきます

`config/database.yml`を以下のように編集しておきます

```diff
   # http://guides.rubyonrails.org/configuring.html#database-pooling
   pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
+  user: <%= ENV.fetch("DB_USER") { 'postgres' } %>
+  password: <%= ENV.fetch("DB_PASSWORD") { '' } %>
+  host: <%= ENV.fetch("DB_HOST") { '127.0.0.1' } %>
+  port: <%= ENV.fetch("DB_PORT") { 5432 } %>

 development:
```

MySQLの人はデフォルト値を `root`, `3306` などとしておきましょう

# Railsサーバ起動前の開発完了

```sh
export DB_PORT=15432
# export DB_HOST=example.com # 必要に応じて追加
bin/rails db:create
bin/rails db:migrate
```

※開発時に`DATABASE_URL`を指定するのはdevelopmentとtestのDB名が衝突するため非常に面倒になるのでやめときます

※ていうか今回の所詮デモであれば `config/database.yml` に直書きしても支障はありません

# Railsサーバ起動前の開発完了

```sh
bin/rails g controller home index
```

`app/views/home/index.html.erb`編集

```
<div class="container">
  <div class="row">
    <div class="col-4">col-4</div>
    <div class="col-8">col-8</div>
  </div>
  <div class="row">
    <div class="col-md-4 col-sm-12">col-md-4 col-sm-12</div>
    <div class="col-md-8 col-sm-12">col-md-8 col-sm-12</div>
  </div>
</div>
```

# Rails起動

Rubyのコンテナで動いているbash上で以下を実行すればOKです

```sh
bin/bundle install
bin/rails db:setup
bin/rails s
```

※雑ですが本気でやりたい人は `config/puma.rb` をちゃんと編集して

```sh
bin/bundle exec pumactl start
```

とかやって下さい

# 開発時Tips

```sh
bin/webpack-dev-server
```

を起動しておく(※`bin/rails s`と別に)と，CSSやJSを編集したときにブラウザを自動で更新してくれます

便利
