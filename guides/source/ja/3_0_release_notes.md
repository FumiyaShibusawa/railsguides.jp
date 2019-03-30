Ruby on Rails 3.0 リリースノート
===============================

Rails 3.0は、晩ご飯も作れば洗濯物も畳んでくれる、夢のような製品です。Rails 3.0なしで今までどうやって暮らしていけたのか、不思議で仕方がなくなるでしょう。Rails 3.0は、これまでのRailsバージョンの中でかつてない高みに到達しました。

冗談はともかく、Rails 3.0は実によい仕上がりとなりました。Merbチームが我らがRailsチームに参加したことでもたらされた素晴らしいアイデアがすべて込められています。Merbチームはフレームワークの謎に満ちた内部をスリムかつ高速化し、素敵なAPIを多数もたらしてくれました。Merb 1.xをご存じの方であれば、Rails 3.0にその成果が多数盛り込まれていることがおわかりいただけるはずです。Rails 2.xをご存じの方もきっとRails 3.0を好きになっていただけるでしょう。

多くの機能や改良APIを搭載したRails 3.0は、内部がきれいになったかどうかに関心のない方も夢中になるでしょう。Railsアプリケーション開発者にとって、かつてないほど素晴らしい時期が到来しました。その中からいくつかご紹介します。

* RESTful宣言を重視する新しいルーター
* Action Controllerの後でモデル化される新しいAction Mailer API（もうマルチパートメッセージの送信で悩むことはありません！）
* リレーショナル代数の上で構築される、Active Recordのチェイン可能なクエリ言語
* Prototype.jsやjQueryなどのドライバを搭載したUnobtrusive JavaScript（UJS）ヘルパー（さようならインラインJS）
* Bundlerによる明示的な依存性管理

そして何よりも、今回私たちは旧APIを非推奨化する際にできるかぎり適切なwarningを表示することを心がけました。つまり、Rails 3に移行するために、既存のアプリケーションの古いコードを今すぐ最新のベストプラクティスにすべて書き換えなくてもよいということです。

これらのリリースノートでは主要なアップグレードを取り扱いますが、細かなバグ修正や変更点については記載しませんのでご注意ください。Rails 3.0は、250人を超える人々による4,000件近いコミットを含みます。すべての変更点をチェックしたい方は、GitHub上のメインRailsリポジトリで[コミットリスト](https://github.com/rails/rails/commits/3-0-stable)をご覧ください。

--------------------------------------------------------------------------------

Rails 3のインストール方法は次のとおりです。

```bash
# セットアップで必要な場合はsudoを使います
$ gem install rails
```


Rails 3へのアップグレード
--------------------

既存のアプリケーションをアップグレードするのであれば、その前に質のよいテストカバレッジを用意するのはよい考えです。アプリケーションがRails 2.3.5までアップグレードされていない場合は先にそれを完了し、アプリケーションが正常に動作することを十分確認してからRails 3にアップデートしてください。終わったら、以下の変更点にご注意ください。

### Rails 3ではRuby 1.8.7以降が必須

Rails 3.0ではRuby 1.8.7以上が必須です。これより前のバージョンのRubyのサポートは公式に廃止されたため、速やかにRubyをアップグレードすべきです。Rails 3.0はRuby 1.9.2とも互換性があります。

TIP: Ruby 1.8.7のp248とp249には、Railsクラッシュの原因となるマーシャリングのバグがあります。なおRuby Enterprise Editionでは1.8.7-2010.02のリリースでこの問題が修正されました。現行のRuby 1.9のうち、Ruby 1.9.1はRails 3.0でセグメンテーションフォールト（segfault）で完全にダウンするため利用できません。Railsをスムーズに動かすため、Rails 3でRuby 1.9.xを使いたい場合は1.9.2をお使いください。

### Railsの「アプリケーションオブジェクト」

Rails 3では、同一プロセス内で複数のRailsアプリケーション実行をサポートするための基礎の一環として、「アプリケーションオブジェクト」という概念が導入されました。1つのアプリケーションオブジェクトには、そのアプリケーション固有の設定がすべて保持されます。しかも、この設定は従来のRailsの`config/environment.rb`と極めて似通っています。

今後、各Railsアプリケーションはそれに対応するアプリケーションオブジェクトを持たなければなりません。このアプリケーションオブジェクトは`config/application.rb`で定義されます。既存のアプリケーションをRails 3にアップグレードする場合、この`config/application.rb`を追加して、`config/environment.rb`内の設定を適宜そこに移動しなければなりません。

### script/*がscript/railsに置き換えられる

従来スクリプトの置き場に使われていた`script`ディレクトリは、新しい`script/rails`に置き換えられます。`script/rails`は直接実行するものではありませんが、Railsアプリケーションのルートディレクトリで呼び出されたことを`rails`コマンドが検知して、代わりにスクリプトを実行します。望ましい用法は以下のとおりです。

```bash
$ rails console                      # script/consoleではない
$ rails g scaffold post title:string # script/generate scaffold post title:stringではない
```

`rails --help`を実行すればすべてのオプションを表示できます。

### 依存関係とconfig.gem

従来の`config.gem`を用いる方式は廃止され、`bundler`と`Gemfile`を用いる方式に置き換えられました。後述の[gemに移行する](#gemに移行する)を参照してください。

### アップグレードプロセス

アップグレードプロセスを支援するために、[Rails Upgrade](http://github.com/rails/rails_upgrade)というプラグインが自動化の一部として作成されました。

このプラグインをインストールして`rake rails:upgrade:check`を実行すれば、アプリ内でアップデートの必要な箇所をチェックできます（アップグレード方法の情報へのリンクも表示されます）。このプラグインは、`config.gem`呼び出しに基づいて`Gemfile`を生成するタスクや、現在のルーティングから新しいルーティングファイルを生成するタスクも提供します。以下を実行すればプラグインを取得できます。

```bash
$ ruby script/plugin install git://github.com/rails/rails_upgrade.git
```

動作例については「[Rails Upgradeプラグインが公式化された] (http://omgbloglol.com/post/364624593/rails-upgrade-is-now-an-official-plugin)
（英語）」で参照できます。

Rails Upgradeツール以外にも、支援が必要な場合はIRCやGoogleグループの[rubyonrails-talk](http://groups.google.com/group/rubyonrails-talk)に自分と同じ問題に遭遇した人がおそらくいるでしょう。ぜひアップグレード作業をブログ記事にして、他の人々も知見を得られるようにしましょう。

Rails 3.0アプリケーションを作成する
--------------------------------

```
# 'rails'というRubyGemがインストールされている必要があります。
$ rails new myapp
$ cd myapp
```

### gemに移行する

今後のRailsでは、アプリケーションのルートディレクトリに置かれる`Gemfile`を使って、アプリケーションの起動に必要なgemを指定するようになりました。この`Gemfile`は[Bundler](https://github.com/carlhuda/bundler)というgemによって処理され、依存関係のある必要なgemをすべてインストールします。依存するgemをそのアプリケーションの中にだけインストールして、OS環境にある既存のgemに影響を与えないようにすることもできます。

詳細情報: [Bundlerホームページ](https://bundler.io/)

### 最新のgemを使う

`Bundler`と`Gemfile`のおかげで、専用の`bundle`コマンド一発でRailsアプリケーションのgemを簡単に安定させることができます。Gitリポジトリから直接bundleしたい場合は`--edge`フラグを追加します。

```
$ rails new myapp --edge
```

Railsアプリケーションのリポジトリをローカルにチェックアウトしたものがあり、それを使ってアプリケーションを生成したい場合は、`--dev`フラグを追加します。

```
$ ruby /path/to/rails/bin/rails new myapp --dev
```

Railsアーキテクチャの変更
---------------------------

Railsのアーキテクチャで6つの大きな変更点が発生しました。

### Railtiesが一新された

Railtiesが更新され、Railsフレームワーク全体で一貫したプラグインAPIを提供するようになるとともに、ジェネレータやRailsバインディングが完全に書き直されました。これによって、開発者がジェネレータやアプリケーションフレームワークのあらゆる重要なステージに統一的かつ定義済みの方法でフックをかけられるようになりました。

### Railsのあらゆるコアコンポーネントが分離された

MerbとRailsの主要なマージ作業のひとつが、Railsのコアコンポーネントの密結合を切り離すことでした。この作業が完了したことで、Railsのあらゆるコアコンポーネントが同一のAPIを使うようになりました。これらのAPIはプラグイン開発に利用できます。つまり、作成したプラグインやコアコンポーネントを置き換える（DataMapperやSequelなど）際に、Railsコアコンポーネントからアクセスできるあらゆる機能にアクセスして自由自在に拡張できるようになったということです。

詳しくは「[The Great Decoupling](http://yehudakatz.com/2009/07/19/rails-3-the-great-decoupling/)」を参照してください。

### Active Modelを抽象化した

密結合したコアコンポーネントの切り離し作業の一環として、Active Recordへの結合をAction Packから切り出しました。この作業が完了したことで、新しいORMプラグインではActive Modelインターフェイスを実装するだけでAction Packとシームレスに協調動作できるようになりました。

詳しくは「[Make Any Ruby Object Feel Like ActiveRecord](http://yehudakatz.com/2010/01/10/activemodel-make-any-ruby-object-feel-like-activerecord/)」を参照してください。

### コントローラを抽象化した

密結合したコアコンポーネントの切り離し作業として、基底スーパークラスを1つ作成しました。このクラスは、ビューのレンダリングなどを扱うHTTPの記法から分離されています。この`AbstractController`クラスを作成したことで、`ActionController`や`ActionMailer`が極めてシンプルになり、これらのライブラリすべてから共通部分が切り出されてAbstract Controllerに移動しました。

詳しくは「[Rails Edge Architecture](http://yehudakatz.com/2009/06/11/rails-edge-architecture/)」を参照してください。

### Arelを統合した

[Arel](http://github.com/brynary/arel)（Active Relationとも呼ばれます）がActive Recordの下に配置され、Railsで必須のコンポーネントになりました。Arelは、Active Recordを簡潔にするSQL抽象化を提供し、Active Recordのリレーション機能を支えます。

詳しくは「[Why I wrote Arel](http://magicscalingsprinkles.wordpress.com/2010/01/28/why-i-wrote-arel/)」を参照してください。

### メールを切り出した

Action Mailerは最初期からモンキーパッチやプリパーサーだらけで、配信エージェントや受信エージェントまであり、さらにソースツリーでTMailをベンダリングしているというありさまでした。Rails 3では、メールに関連するあらゆる機能を[Mail](http://github.com/mikel/mail) gemで抽象化しました。こちらでもコードの重複が著しく解消され、Action Mailerとメールパーサーの間に定義可能な境界を作成しやすくなりました。

詳しくは「[New Action Mailer API in Rails 3](http://lindsaar.net/2010/1/26/new-actionmailer-api-in-rails-3)」を参照してください。

ドキュメント
-------------

Railsツリーのドキュメントが更新されてAPI変更がすべて反映されました。さらに、[Rails Edgeガイド](http://edgeguides.rubyonrails.org/)にもRails 3.0の変更点を順次反映中です。ただし、[guides.rubyonrails.org](http://guides.rubyonrails.org/)の本ガイドについては、安定版Railsのドキュメントのみを含みます（執筆時点では3.0がリリースされるまで2.3.5となります）。

詳しくは「[Rails Documentation Projects](http://weblog.rubyonrails.org/2009/1/15/rails-documentation-projects.)」を参照してください。

国際化（I18n）
--------------------

Rails 3では国際化（I18n）のサポートに関して、速度改善が著しい最新の[I18n](http://github.com/svenfuchs/i18n) gemなど多くの作業が行われました。

* あらゆるオブジェクトでのI18n: `ActiveModel::Translation`や`ActiveModel::Validations`を含むあらゆるオブジェクトにI18nの振る舞いを追加できます。訳文の`errors.messages`フォールバック機能もあります。
* 属性にデフォルトの訳文を与えられます。
* フォームのsubmitタグが、オブジェクトのステータスに応じて自動的に正しいステータス（CreateまたはUpdate）を取れるようになったことで、正しい訳文も取れるようになりました。
* I18n化されたラベルに属性名を渡すだけで使えるようになりました。

詳しくは「[Rails 3 I18n changes](http://blog.plataformatec.com.br/2010/02/rails-3-i18n-changes/)」を参照してください。

Railties
--------

主要なRailsフレームワークの分離作業に伴って、Railtiesも大規模にオーバーホールされ、フレームワーク/エンジン/プラグインをできるだけ楽に拡張できる形で連結しました。

* アプリケーションごとに独自の名前空間が与えられます。アプリケーションはたとえば`アプリ名.boot`で始まり、他のアプリケーションとのやりとりが今よりもずっと簡単になります。
* `Rails.root/app`以下に置かれるものはすべて読み込みパスに追加されるようになりました。これにより、たとえば`app/observers/user_observer.rb`を作るだけで、設定変更なしでRailsが読み込んでくれます。
* Rails 3.0では`Rails.config`オブジェクトが提供されます。これは、Railsの膨大な設定オプションをすべて集約する中央リポジトリを提供します。

アプリケーション生成時に、test-unit/Active Record/Prototype.js/Gitのインストールをスキップするフラグも渡せるようになりました。また、`--dev`フラグも新たに追加されたことで、Railsを指す`Gemfile`をチェックアウト状態でアプリをセットアップできるようになりました（これは`rails`バイナリへのパスで決定されます）。詳しくは`rails --help`を参照してください。

Rails 3.0のRailtiesジェネレータでは、基本的に以下のようなさまざまな注意点があります。

* ジェネレータが完全に書き直されたことで後方互換性が失われた
* RailsテンプレートAPIとジェネレータAPIがマージされた: 利用上は従来と同じです
* ジェネレータは特殊なパスからの読み込みを行わなくなった: 今後はRubyの読み込みパスのみを探索しますので、`rails generate foo`を呼び出すと`generators/foo_generator`を探索します。
* 新しいジェネレータではフックが提供される: あらゆるテンプレートエンジン/ORM/テストフレームワークを簡単にフックインできます
* 新しいジェネレータでは、`Rails.root/lib/templates`にテンプレートのコピーを置くことでテンプレートをオーバーライドできる
* `Rails::Generators::TestCase`も提供される: これを用いて独自のジェネレータを作成・テストできます

また、Railtiesジェネレータで生成されるビューについてもいくつかオーバーホールが行われました。

* ビューで`p`タグの代わりに`div`タグが使われるようになった
* scaffoldジェネレータで、editビューとnewビューの重複コードの代わりに`_form`パーシャルを使うようになった
* scaffoldフォームで`f.submit`が使われるようになった: これは、渡されるオブジェクトのステートに応じて「Create ModelName」または「Update ModelName」を返します

最後に、rakeタスクもいくつかの点が拡張されました。

* `rake db:forward`が追加された: マイグレーションを個別またはグループにまとめてロールフォワードできます
* `rake routes CONTROLLER=x`が追加された: コントローラを1つ指定してルーティングを表示できます。

Railtiesで以下が非推奨化されました。

* `RAILS_ROOT`が非推奨化: 今後は`Rails.root`を使います
* `RAILS_ENV`が非推奨化: 今後は`Rails.env`を使います
* `RAILS_DEFAULT_LOGGER`が非推奨化: `Rails.logger`を使います

`PLUGIN/rails/tasks`と`PLUGIN/tasks`は今後どのタスクでも読み込まれなくなったので、今後は`PLUGIN/lib/tasks`を使わなければなりません。

詳しくは以下を参照してください。

* [Discovering Rails 3 generators](http://blog.plataformatec.com.br/2010/01/discovering-rails-3-generators)
* [Making Generators for Rails 3 with Thor](http://caffeinedd.com/guides/331-making-generators-for-rails-3-with-thor)
* [The Rails Module (in Rails 3)](http://litanyagainstfear.com/blog/2010/02/03/the-rails-module/)

Action Pack
-----------

Action Packは内部と外部ともに大きく変更されました。

### Abstract Controller

Action Controllerから一般性の高い部分をAbstract Controllerに切り出して再利用可能なモジュールとし、テンプレートやパーシャルのレンダリング、ヘルパー、訳文、ログ出力など「リクエスト-レスポンス」サイクルのあらゆる要素をどのライブラリからでも利用できるようにしました。この抽象化によって、`ActionMailer::Base`は`AbstractController`を継承してRails DSLをMail gemでラップするだけで済むようになりました。

Abstract Controllerを導入したことで、Action Controllerのコードをクリーンアップするよい機会となり、コードをシンプルにする部分が抽象化されました。

ただし、Abstract Controllerはユーザー（開発者）が直接使うAPIではない点にご注意ください。日々のRails開発でAbstract Controllerを使うことはありません。

詳しくは「[Rails Edge Architecture](http://yehudakatz.com/2009/06/11/rails-edge-architecture/)」を参照してください。

### Action Controller

* `application_controller.rb`に`protect_from_forgery`がデフォルトで含まれるようになりました。
* `cookie_verifier_secret`は非推奨化され、今後は`Rails.application.config.cookie_secret`で代入されます。また、これは`config/initializers/cookie_verification_secret.rb`という独自のファイルに移動しました。
* `session_store`は`ActionController::Base.session`で設定されるようになり、`Rails.application.config.session_store`に移動しました。デフォルトは`config/initializers/session_store.rb`で設定されます。
* `cookies.secure`を用いて、暗号化された値を`cookie.secure[:key] => value`でcookieに設定できます。
* `cookies.permanent`を用いて、恒久的な値を`cookie.permanent[:key] => value`でcookieハッシュに設定できます。署名済みの値が検証に失敗すると例外がraiseされます。
* `respond_to`ブロック内の`format`呼び出しに、`:notice => 'This is a flash message'`や`:alert => 'Something went wrong'`を渡せるようになりました。`flash[]`ハッシュの動作は従来と同じです。
* `respond_with`メソッドがコントローラに追加されます。これを用いて、込み入った`format`ブロックをシンプルに書けます。
* `ActionController::Responder`が追加されました。これを用いて、レスポンスを柔軟に生成できます。

以下が非推奨化されました。

* `filter_parameter_logging`が非推奨化されました。今後は`config.filter_parameters << :password`をお使いください。

詳しくは以下を参照してください。

* [Render Options in Rails 3](http://www.engineyard.com/blog/2010/render-options-in-rails-3/)
* [Three reasons to love ActionController::Responder](http://weblog.rubyonrails.org/2009/8/31/three-reasons-love-responder)

### Action Dispatch

Action DispatchはRails 3.0で新しく追加されました。これはルーティングの明快な実装を新たに提供します。

* ルーターが大々的に書き直されてクリーンアップされたことで、RailsのルーターがRails DSLを持つ`rack_mount`となりました。これは単独のソフトウェアです。
* 各アプリケーションで定義されるルーティングは、以下のようにApplicationモジュール内で名前空間化されるようになりました。

    ```ruby
    # 従来

    ActionController::Routing::Routes.draw do |map|
      map.resources :posts
    end

    # 今後はこのように書く

    AppName::Application.routes do
      resources :posts
    end
    ```

* `match`メソッドがルーターに追加され、マッチしたルーティングに任意のRackアプリケーションも渡せるようになりました。
* `constraints`メソッドがルーターに追加され、定義済みの制約を用いてルーターを保護できるようになりました。
* `scope`メソッドがルーターに追加され、以下のようにさまざまな言語（ロケール）やアクションへのルーティングを名前空間化できるようになりました。

    ```ruby
    scope 'es' do
      resources :projects, :path_names => { :edit => 'cambiar' }, :path => 'proyecto'
    end

    # /es/proyecto/1/cambiarでeditアクションにアクセスできる
    ```

* Added `root` method to the router as a short cut for `match '/', :to => path`.
* You can pass optional segments into the match, for example `match "/:controller(/:action(/:id))(.:format)"`, each parenthesized segment is optional.
* Routes can be expressed via blocks, for example you can call `controller :home { match '/:action' }`.

NOTE. The old style `map` commands still work as before with a backwards compatibility layer, however this will be removed in the 3.1 release.

Deprecations

* The catch all route for non-REST applications (`/:controller/:action/:id`) is now commented out.
* Routes `:path_prefix` no longer exists and `:name_prefix` now automatically adds "_" at the end of the given value.

More Information:
* [The Rails 3 Router: Rack it Up](http://yehudakatz.com/2009/12/26/the-rails-3-router-rack-it-up/)
* [Revamped Routes in Rails 3](http://rizwanreza.com/2009/12/20/revamped-routes-in-rails-3)
* [Generic Actions in Rails 3](http://yehudakatz.com/2009/12/20/generic-actions-in-rails-3/)


### Action View

#### Unobtrusive JavaScript

Major re-write was done in the Action View helpers, implementing Unobtrusive JavaScript (UJS) hooks and removing the old inline AJAX commands. This enables Rails to use any compliant UJS driver to implement the UJS hooks in the helpers.

What this means is that all previous `remote_<method>` helpers have been removed from Rails core and put into the [Prototype Legacy Helper](http://github.com/rails/prototype_legacy_helper.) To get UJS hooks into your HTML, you now pass `:remote => true` instead. For example:

```ruby
form_for @post, :remote => true
```

Produces:

```html
<form action="http://host.com" id="create-post" method="post" data-remote="true">
```

#### Helpers with Blocks

Helpers like `form_for` or `div_for` that insert content from a block use `<%=` now:

```html+erb
<%= form_for @post do |f| %>
  ...
<% end %>
```

Your own helpers of that kind are expected to return a string, rather than appending to the output buffer by hand.

Helpers that do something else, like `cache` or `content_for`, are not affected by this change, they need `&lt;%` as before.

#### Other Changes

* You no longer need to call `h(string)` to escape HTML output, it is on by default in all view templates. If you want the unescaped string, call `raw(string)`.
* Helpers now output HTML 5 by default.
* Form label helper now pulls values from I18n with a single value, so `f.label :name` will pull the `:name` translation.
* I18n select label on should now be :en.helpers.select instead of :en.support.select.
* You no longer need to place a minus sign at the end of a Ruby interpolation inside an ERB template to remove the trailing carriage return in the HTML output.
* Added `grouped_collection_select` helper to Action View.
* `content_for?` has been added allowing you to check for the existence of content in a view before rendering.
* passing `:value => nil` to form helpers will set the field's `value` attribute to nil as opposed to using the default value
* passing `:id => nil` to form helpers will cause those fields to be rendered with no `id` attribute
* passing `:alt => nil` to `image_tag` will cause the `img` tag to render with no `alt` attribute

Active Model
------------

Active Model is new in Rails 3.0. It provides an abstraction layer for any ORM libraries to use to interact with Rails by implementing an Active Model interface.


### ORM Abstraction and Action Pack Interface

Part of decoupling the core components was extracting all ties to Active Record from Action Pack. This has now been completed. All new ORM plugins now just need to implement Active Model interfaces to work seamlessly with Action Pack.

More Information: - [Make Any Ruby Object Feel Like ActiveRecord](http://yehudakatz.com/2010/01/10/activemodel-make-any-ruby-object-feel-like-activerecord/)


### Validations

Validations have been moved from Active Record into Active Model, providing an interface to validations that works across ORM libraries in Rails 3.

* There is now a `validates :attribute, options_hash` shortcut method that allows you to pass options for all the validates class methods, you can pass more than one option to a validate method.
* The `validates` method has the following options:
    * `:acceptance => Boolean`.
    * `:confirmation => Boolean`.
    * `:exclusion => { :in => Enumerable }`.
    * `:inclusion => { :in => Enumerable }`.
    * `:format => { :with => Regexp, :on => :create }`.
    * `:length => { :maximum => Fixnum }`.
    * `:numericality => Boolean`.
    * `:presence => Boolean`.
    * `:uniqueness => Boolean`.

NOTE: All the Rails version 2.3 style validation methods are still supported in Rails 3.0, the new validates method is designed as an additional aid in your model validations, not a replacement for the existing API.

You can also pass in a validator object, which you can then reuse between objects that use Active Model:

```ruby
class TitleValidator < ActiveModel::EachValidator
  Titles = ['Mr.', 'Mrs.', 'Dr.']
  def validate_each(record, attribute, value)
    unless Titles.include?(value)
      record.errors[attribute] << 'must be a valid title'
    end
  end
end
```

```ruby
class Person
  include ActiveModel::Validations
  attr_accessor :title
  validates :title, :presence => true, :title => true
end

# Or for Active Record

class Person < ActiveRecord::Base
  validates :title, :presence => true, :title => true
end
```

There's also support for introspection:

```ruby
User.validators
User.validators_on(:login)
```

More Information:

* [Sexy Validation in Rails 3](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/)
* [Rails 3 Validations Explained](http://lindsaar.net/2010/1/31/validates_rails_3_awesome_is_true)


Active Record
-------------

Active Record received a lot of attention in Rails 3.0, including abstraction into Active Model, a full update to the Query interface using Arel, validation updates and many enhancements and fixes. All of the Rails 2.x API will be usable through a compatibility layer that will be supported until version 3.1.


### Query Interface

Active Record, through the use of Arel, now returns relations on its core methods. The existing API in Rails 2.3.x is still supported and will not be deprecated until Rails 3.1 and not removed until Rails 3.2, however, the new API provides the following new methods that all return relations allowing them to be chained together:

* `where` - provides conditions on the relation, what gets returned.
* `select` - choose what attributes of the models you wish to have returned from the database.
* `group` - groups the relation on the attribute supplied.
* `having` - provides an expression limiting group relations (GROUP BY constraint).
* `joins` - joins the relation to another table.
* `clause` - provides an expression limiting join relations (JOIN constraint).
* `includes` - includes other relations pre-loaded.
* `order` - orders the relation based on the expression supplied.
* `limit` - limits the relation to the number of records specified.
* `lock` - locks the records returned from the table.
* `readonly` - returns an read only copy of the data.
* `from` - provides a way to select relationships from more than one table.
* `scope` - (previously `named_scope`) return relations and can be chained together with the other relation methods.
* `with_scope` - and `with_exclusive_scope` now also return relations and so can be chained.
* `default_scope` - also works with relations.

More Information:

* [Active Record Query Interface](http://m.onkey.org/2010/1/22/active-record-query-interface)
* [Let your SQL Growl in Rails 3](http://hasmanyquestions.wordpress.com/2010/01/17/let-your-sql-growl-in-rails-3/)


### Enhancements

* Added `:destroyed?` to Active Record objects.
* Added `:inverse_of` to Active Record associations allowing you to pull the instance of an already loaded association without hitting the database.


### Patches and Deprecations

Additionally, many fixes in the Active Record branch:

* SQLite 2 support has been dropped in favor of SQLite 3.
* MySQL support for column order.
* PostgreSQL adapter has had its `TIME ZONE` support fixed so it no longer inserts incorrect values.
* Support multiple schemas in table names for PostgreSQL.
* PostgreSQL support for the XML data type column.
* `table_name` is now cached.
* A large amount of work done on the Oracle adapter as well with many bug fixes.

As well as the following deprecations:

* `named_scope` in an Active Record class is deprecated and has been renamed to just `scope`.
* In `scope` methods, you should move to using the relation methods, instead of a `:conditions => {}` finder method, for example `scope :since, lambda {|time| where("created_at > ?", time) }`.
* `save(false)` is deprecated, in favor of `save(:validate => false)`.
* I18n error messages for Active Record should be changed from :en.activerecord.errors.template to `:en.errors.template`.
* `model.errors.on` is deprecated in favor of `model.errors[]`
* validates_presence_of => validates... :presence => true
* `ActiveRecord::Base.colorize_logging` and `config.active_record.colorize_logging` are deprecated in favor of `Rails::LogSubscriber.colorize_logging` or `config.colorize_logging`

NOTE: While an implementation of State Machine has been in Active Record edge for some months now, it has been removed from the Rails 3.0 release.


Active Resource
---------------

Active Resource was also extracted out to Active Model allowing you to use Active Resource objects with Action Pack seamlessly.

* Added validations through Active Model.
* Added observing hooks.
* HTTP proxy support.
* Added support for digest authentication.
* Moved model naming into Active Model.
* Changed Active Resource attributes to a Hash with indifferent access.
* Added `first`, `last` and `all` aliases for equivalent find scopes.
* `find_every` now does not return a `ResourceNotFound` error if nothing returned.
* Added `save!` which raises `ResourceInvalid` unless the object is `valid?`.
* `update_attribute` and `update_attributes` added to Active Resource models.
* Added `exists?`.
* Renamed `SchemaDefinition` to `Schema` and `define_schema` to `schema`.
* Use the `format` of Active Resources rather than the `content-type` of remote errors to load errors.
* Use `instance_eval` for schema block.
* Fix `ActiveResource::ConnectionError#to_s` when `@response` does not respond to #code or #message, handles Ruby 1.9 compatibility.
* Add support for errors in JSON format.
* Ensure `load` works with numeric arrays.
* Recognizes a 410 response from remote resource as the resource has been deleted.
* Add ability to set SSL options on Active Resource connections.
* Setting connection timeout also affects `Net::HTTP` `open_timeout`.

Deprecations:

* `save(false)` is deprecated, in favor of `save(:validate => false)`.
* Ruby 1.9.2: `URI.parse` and `.decode` are deprecated and are no longer used in the library.


Active Support
--------------

A large effort was made in Active Support to make it cherry pickable, that is, you no longer have to require the entire Active Support library to get pieces of it. This allows the various core components of Rails to run slimmer.

These are the main changes in Active Support:

* Large clean up of the library removing unused methods throughout.
* Active Support no longer provides vendored versions of [TZInfo](http://tzinfo.rubyforge.org/), [Memcache Client](http://deveiate.org/projects/RMemCache/) and [Builder](http://builder.rubyforge.org/,) these are all included as dependencies and installed via the `bundle install` command.
* Safe buffers are implemented in `ActiveSupport::SafeBuffer`.
* Added `Array.uniq_by` and `Array.uniq_by!`.
* Removed `Array#rand` and backported `Array#sample` from Ruby 1.9.
* Fixed bug on `TimeZone.seconds_to_utc_offset` returning wrong value.
* Added `ActiveSupport::Notifications` middleware.
* `ActiveSupport.use_standard_json_time_format` now defaults to true.
* `ActiveSupport.escape_html_entities_in_json` now defaults to false.
* `Integer#multiple_of?` accepts zero as an argument, returns false unless the receiver is zero.
* `string.chars` has been renamed to `string.mb_chars`.
* `ActiveSupport::OrderedHash` now can de-serialize through YAML.
* Added SAX-based parser for XmlMini, using LibXML and Nokogiri.
* Added `Object#presence` that returns the object if it's `#present?` otherwise returns `nil`.
* Added `String#exclude?` core extension that returns the inverse of `#include?`.
* Added `to_i` to `DateTime` in `ActiveSupport` so `to_yaml` works correctly on models with `DateTime` attributes.
* Added `Enumerable#exclude?` to bring parity to `Enumerable#include?` and avoid if `!x.include?`.
* Switch to on-by-default XSS escaping for rails.
* Support deep-merging in `ActiveSupport::HashWithIndifferentAccess`.
* `Enumerable#sum` now works will all enumerables, even if they don't respond to `:size`.
* `inspect` on a zero length duration returns '0 seconds' instead of empty string.
* Add `element` and `collection` to `ModelName`.
* `String#to_time` and `String#to_datetime` handle fractional seconds.
* Added support to new callbacks for around filter object that respond to `:before` and `:after` used in before and after callbacks.
* The `ActiveSupport::OrderedHash#to_a` method returns an ordered set of arrays. Matches Ruby 1.9's `Hash#to_a`.
* `MissingSourceFile` exists as a constant but it is now just equals to `LoadError`.
* Added `Class#class_attribute`, to be able to declare a class-level attribute whose value is inheritable and overwritable by subclasses.
* Finally removed `DeprecatedCallbacks` in `ActiveRecord::Associations`.
* `Object#metaclass` is now `Kernel#singleton_class` to match Ruby.

The following methods have been removed because they are now available in Ruby 1.8.7 and 1.9.

* `Integer#even?` and `Integer#odd?`
* `String#each_char`
* `String#start_with?` and `String#end_with?` (3rd person aliases still kept)
* `String#bytesize`
* `Object#tap`
* `Symbol#to_proc`
* `Object#instance_variable_defined?`
* `Enumerable#none?`

The security patch for REXML remains in Active Support because early patch-levels of Ruby 1.8.7 still need it. Active Support knows whether it has to apply it or not.

The following methods have been removed because they are no longer used in the framework:

* `Kernel#daemonize`
* `Object#remove_subclasses_of` `Object#extend_with_included_modules_from`, `Object#extended_by`
* `Class#remove_class`
* `Regexp#number_of_captures`, `Regexp.unoptionalize`, `Regexp.optionalize`, `Regexp#number_of_captures`


Action Mailer
-------------

Action Mailer has been given a new API with TMail being replaced out with the new [Mail](http://github.com/mikel/mail) as the email library. Action Mailer itself has been given an almost complete re-write with pretty much every line of code touched. The result is that Action Mailer now simply inherits from Abstract Controller and wraps the Mail gem in a Rails DSL. This reduces the amount of code and duplication of other libraries in Action Mailer considerably.

* All mailers are now in `app/mailers` by default.
* Can now send email using new API with three methods: `attachments`, `headers` and `mail`.
* Action Mailer now has native support for inline attachments using the `attachments.inline` method.
* Action Mailer emailing methods now return `Mail::Message` objects, which can then be sent the `deliver` message to send itself.
* All delivery methods are now abstracted out to the Mail gem.
* The mail delivery method can accept a hash of all valid mail header fields with their value pair.
* The `mail` delivery method acts in a similar way to Action Controller's `respond_to`, and you can explicitly or implicitly render templates. Action Mailer will turn the email into a multipart email as needed.
* You can pass a proc to the `format.mime_type` calls within the mail block and explicitly render specific types of text, or add layouts or different templates. The `render` call inside the proc is from Abstract Controller and supports the same options.
* What were mailer unit tests have been moved to functional tests.
* Action Mailer now delegates all auto encoding of header fields and bodies to Mail Gem
* Action Mailer will auto encode email bodies and headers for you

Deprecations:

* `:charset`, `:content_type`, `:mime_version`, `:implicit_parts_order` are all deprecated in favor of `ActionMailer.default :key => value` style declarations.
* Mailer dynamic `create_method_name` and `deliver_method_name` are deprecated, just call `method_name` which now returns a `Mail::Message` object.
* `ActionMailer.deliver(message)` is deprecated, just call `message.deliver`.
* `template_root` is deprecated, pass options to a render call inside a proc from the `format.mime_type` method inside the `mail` generation block
* The `body` method to define instance variables is deprecated (`body {:ivar => value}`), just declare instance variables in the method directly and they will be available in the view.
* Mailers being in `app/models` is deprecated, use `app/mailers` instead.

More Information:

* [New Action Mailer API in Rails 3](http://lindsaar.net/2010/1/26/new-actionmailer-api-in-rails-3)
* [New Mail Gem for Ruby](http://lindsaar.net/2010/1/23/mail-gem-version-2-released)


Credits
-------

See the [full list of contributors to Rails](http://contributors.rubyonrails.org/) for the many people who spent many hours making Rails 3. Kudos to all of them.

Rails 3.0 Release Notes were compiled by [Mikel Lindsaar](http://lindsaar.net.)