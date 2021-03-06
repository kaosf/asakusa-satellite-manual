セットアップ
=======================
動作環境
-----------------------
以下のソフトウェアのインストールが必要です。

* Ruby 1.8.7 or 1.9.3
* RubyGems 1.4.2 or later
* Bundler 1.0.7 or later
* MongoDB 1.8.1 or later

インストール
-----------------------

以下では、3つの場合のインストール方法について説明します。

* Windows以外のOS(単体起動)
* Windows以外のOS(PassengerによるApacheとの連携)
* Windows

Windows以外のOS(単体起動)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ダウンロードリンク_ から最新版をダウンロードし、適当なディレクトリに展開してください。
展開したディレクトリを AsakusaSatellite にリネームし、以下のコマンドを実行してください。

.. _ダウンロードリンク: http://github.com/codefirst/AsakusaSatellite/tags

::

    # MongoDBの起動(起動済みの場合は不要)
    $ mongod --dbpath <dir_name>

    # 展開ディレクトリへの移動
    $ cd AsakusaSatellite

    # 依存ライブラリのインストール
    $ bundle install --path .bundle --without development test

    # リソースのプリコンパイル (0.8 以降)
    $ bundle exec rake assets:precompile RAILS_ENV=production

    # WebSocketサーバ、AsakusaSatellite本体の起動
    $ bundle exec thin -R socky/config.ru -p3002 -t0 start &
    $ bundle exec rails server -e production

localhost 以外で運用する場合には、WebSocket サーバの設定を変更してください。
config/message_pusher.yml の **localhost** を IP アドレス等で置換してください。

::

   socky:
     http: http://localhost:3002/http
     http: ws://localhost:3002/websocket
     app: as
     secret: secret

設定の反映には、AsakusaSatellite の再起動が必要です。

Windows以外のOS(PassengerによるApacheとの連携)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. 設定ファイルの修正

   config/environments/production.rb を編集して serve_static_assets の値を以下のとおり修正します。

::

    config.serve_static_assets = false


2. リソースのプリコンパイル (0.8 以降)

   以下のコマンドを実行します。

::

    $ bundle exec rake assets:precompile RAILS_ENV=production

3. Passenger のインストール

   gem から Passenger をインストールします。

::

  $ gem install passenger

Apacheモジュールをビルドします。
::

  $ passenger-install-apache2-module

Apacheの設定ファイルへの記述が表示されるのでメモを取ります。

例)
::

  LoadModule passenger_module /usr/lib/ruby/gems/1.8/gems/passenger-2.2.11/ext/apache2/mod_passenger.so
  PassengerRoot /usr/lib/ruby/gems/1.8/gems/passenger-2.2.11
  PassengerRuby /usr/bin/ruby1.8

必要なパッケージのインストールを促された場合、
指示にしたがってインストールの後再実行してください。

4. Apache の設定

   Apache への設定に以下を記述します。

::

  RailsEnv production
  RailsBaseURI /as

DocumentRoot に AsakusaSatellite の public ディレクトリへのシンボリックリンクを作成します。

例)

* DocumentRoot が /var/www
* AsakusaSatellite が /var/AsakusaSatellite

にインストールされている場合

::

  $ cd /var/www
  $ sudo ln -s /var/AsakusaSatellite/public as

Apacheの再起動の後、http://hostname/as でアクセスできるようになります。

Windows
~~~~~~~~~~~~~~~~~~~~

以下の Ruby での動作を確認しています。

* Ruby 1.8.7 (http://rubyinstaller.org/downloads/)

また、以下のアドオンのインストールが必要です。

* DevKit (http://rubyinstaller.org/downloads/)

それ以外は、Windows 以外の OS の場合と同じです。

.. _browser:

対応ブラウザ
-----------------------

AsakusaSatellite は以下のブラウザをサポートしています。

* Google Chrome
* Safari

また、制限付きで以下のブラウザをサポートしています。

* Mozilla Firefox
* Opera

Google Chrome
~~~~~~~~~~~~~~~~~~~~

すべての機能をご利用可能です。

Safari
~~~~~~~~~~~~~~~~~~~~

バージョン 6.0 以降ですべての機能をご利用可能です。

Mozilla Firefox
~~~~~~~~~~~~~~~~~~~~

バージョン 4 からのサポートです。
WebSocket を有効にするために、以下の設定を行ってください。
(バージョン 7 以降ではこの設定は不要です)

1. アドレスバーに "about\:config" と入力します。
2. network.websocket.override-security-block の値を "true" に変更します。

以下の機能がご利用いただけません。
(アドオンをインストールすればご利用いただけます)

* デスクトップ通知

Opera
~~~~~~~~~~~~~~~~~~~~

バージョン 11 からのサポートです。
WebSocket を有効にするために、以下の設定を行ってください。

1. アドレスバーに "about\:config" と入力します。
2. "User Prefs" の "Enable WebSockets" をチェックします。
3. "保存" をクリックします。

以下の機能がご利用いただけません。

* デスクトップ通知
* ファイルアップロード

