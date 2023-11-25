# Rails 7.xにバージョンアップした話

author
:   Kazuhiro NISHIYAMA

content-source
:   Kyoto.rb Meetup

date
:   2023-11-27

allotted-time
:   9m

theme
:   lightning-simple

# self.introduction

- 西山 和広
- Ruby のコミッター
- github など: `@znz`
- 株式会社Ruby開発 www.ruby-dev.jp

# 対象

- 個人メモ用アプリ <https://github.com/znz/memo-app-r>
- scaffold をちょっと改造した程度の機能のみ
  - ログイン、検索
  - 位置情報 (これが欲しかったので自作)
  - (自分専用なので)メモ本文は生HTMLが書ける
- Dokku にデプロイ

# 更新バージョン

- Rail 6.1.7.6 → 7.0.8 → 7.1.2
- Ruby 3.1.4 → 3.2.2

# 更新方法

以下のように新規作成したアプリと比較

```
docker run --rm -it ruby:3.1.4 /bin/bash
gem i rails -v '~> 7.0.0'
rails new /tmp/hoge --database=postgresql
```

# 新規と比較して更新

- `Gemfile`
- `config/environments/*.rb`

# bin/rails app:update

- `bin/rails app:update` で更新
- Active Storage の migration などの不要なものは除外してマージ

# decaffeinate

- テストを実行しようとすると `*.coffee` があるとエラーになった
- <https://github.com/decaffeinate/decaffeinate>
  <https://decaffeinate-project.org/>
  を使って `*.coffee` を変換

# turbolinks

- `turbolinks` も `Gemfile` から削除していたのでエラー
- `turbo` に書き換え
- `app/assets/javascripts/application.js` から
  `//= require turbolinks` を削除
  (`turbo` の追加は必要なかった)

# 警告対応

- `to_s(:delimited)` → `to_fs(:delimited)`

# assets:precompile 失敗

Dokku に deploy すると、なぜか `rake assets:precompile` でエラー

    -----> Preparing app for Rails asset pipeline
           Running: rake assets:precompile
           rake aborted!
           LoadError: cannot load such file -- coffee_script

原因不明なので `coffee-rails` を `Gemfile` に戻した。

# ruby も更新

- ついでに ruby も 3.2.2 に更新
- `ruby file: ".ruby-version"` はデプロイでエラー
- `ruby File.read(".ruby-version").chomp` にした

# new framework defaults

- `config.load_defaults 7.0` に更新
- `new_framework_defaults_7_0.rb` を削除
- デプロイするとログアウトしていたのでログインしなおし

# 7.1 に更新

以下と比較して `Gemfile` を更新

```
docker run --rm -it ruby:3.2.2 /bin/bash
gem i rails
rails new /tmp/hoge --database=postgresql
```

# app:update

- `bin/rails app:update`
- Active Storage の migration は除外してマージ

# デプロイ

- テストも問題なく通るのでデプロイ
- 問題なく動いてそう
- Rails 7.0 から 7.1 は Rails のバージョンを上げるだけならあっさりできた

# 今後

- `new_framework_defaults_7_1` はまだ未対応なので対応予定
- JavaScript の位置情報取得部分もテストしたい
- Rails 標準の minitest を試しているが、単純なテストのみなので rspec と両方にしたい
- bootstrap 4 のままなので、更新か他のものに移行したい
- 位置情報取得部分を jQuery から移行したい
- `coffee-rails` は調査不足で原因不明のままだが `sprockets` を消せば解決すると期待したい
