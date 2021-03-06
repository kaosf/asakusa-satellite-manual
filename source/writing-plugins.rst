プラグインの作成
======================================

AsakusaSatellite はプラグインを作成することにより、
機能を拡張することができます。
以下の三種類の拡張方法があります。

メッセージフィルタ
    メッセージを変換するフィルタ機能を追加する拡張方法です。
    例えば、URL をリンクに変換することができます。
ビューフック
    チャットページの決められた場所に、HTML を追加する拡張方法です。
    例えば、各発言に好きなボタンを追加することができます。
認証方法の変更
    AsakusaSatellite へのログインに利用する認証を変更する拡張方法です。
    例えば、認証を Github の OAuth に変更することができます。

メッセージフィルタ
--------------------------------------

フィルタプラグイン生成コマンド
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ジェネレータが用意されています。

::

    $ rails g as_filter フィルタ名

とすることで plugins/as_フィルタ名_filter に以下に以下のように生成されます。

* init.rb: プラグインの初期設定
* lib/フィルタ名_filter.rb: フィルタプログラム
* spec/lib/フィルタ名_filter_spec.rb: フィルタのテスト

フィルタの登録
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
フィルタの登録は config/filter_infra.yml で行います。
登録されたフィルタは、上から順番に適用されます。

.. code-block:: yaml

  - name: フィルタ名_filter

任意の名前で引数を与えることもできます。

.. code-block:: yaml

  - name: フィルタ名_filter
    url: xxx

フィルタプログラム
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
フィルタプログラムは AsakusaSatellite::Filter::Base を継承したクラスを作成し、
process メソッドを実装します。
(ジェネレータで生成した場合は lib/フィルタ名_filter.rb に生成されています。) 
フィルタ設定で与えた引数はAsakusaSatellite::Filter::Base を継承したクラスでは
config 変数で保持しています。

.. code-block:: ruby

  class XxxFilter < AsakusaSatellite::Filter::Base
  
    # メッセージを処理します
    # text: メッセージの本文
    # return: フィルタしたメッセージ
    def process(text)
      %[<a target="_blank" href="#{config.url}">#{text}</a>]
    end
  
  end


ビューフック
--------------------------------------
ビューフックとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ビューフックは AsakusaSatellite のビュープログラムに
あらかじめ設定されたフックポイントにプラグインからHTMLを
埋め込んで画面を拡張する仕組みです。

ビューフックを使用して画面を拡張するには AsakusaSatellite::Hook::Listener
を継承したクラスを実装します。

AsakusaSatellite::Hook::Listener を継承したクラスはRailsの起動時に読み込まれ、
自動的に登録されます。

ビューフックプラグイン生成コマンド
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ジェネレータが用意されています。

::

    $ rails g as_viewhook プラグイン名

とすることで plugins/as_プラグイン名 に以下に以下のように生成されます。

* init.rb: プラグインの初期設定
* lib/プラグイン名.rb: ビューフックプログラム
* spec/lib/プラグイン名_spec.rb: ビューフックのテスト


ビューフックリスナの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作成コマンドを発行し、プラグインのテンプレートを生成します。

ここでは test_listener とします

::

    $  rails g as_viewhook test_listener

plugins/as_test_listener/lib/test_listener.rb を以下のように編集します。

.. code-block:: ruby

    class TestListener < AsakusaSatellite::Hook::Listener
    
      def message_buttons(context)
        if context[:message].body
          context[:message].body.size.to_s + "文字"
        else
          "0文字"
        end
      end

    end


リスナクラスでAsakusaSatelliteに設置されているフックポイント名と同名の
メソッドを実装します(上記の例ではmessage_buttons) 。引数(context)はビューフックから
渡されてくるオブジェクトを保持するハッシュが渡されます。

フックポイントの調べ方
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

フックポイントを調べるには、view の中で call_hook を呼んでいる
箇所を検索することができます。
具体的には、以下のコマンドが利用できます。

::

    $ git grep call_hook app/views

現在のバージョンでは、以下のフックポイントが有効です。

chat_room_top
    チャットルームの上部に HTML を差し込みます
chat_room_bottom
    チャットルームの下部に HTML を差し込みます
message_buttons
    各発言のボタンが表示される箇所に HTML を差し込みます
account_setting_item(0.8.1以降)
    個人設定画面に HTML を差し込みます
global_setting_item(0.8.1以降)
    Aboutページに HTML を差し込みます
global_header(0.8.1以降)
    すべてのページの上部に HTML を差し込みます。
global_footer(0.8.1以降)
    すべてのページの下部に HTML を差し込みます。

assetファイル(画像、CSS等)の公開 (0.8.1以降)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`plugins/<plugin>/app/assets/<type>/<filename>` にファイルを配置すると、 `/plugin/<plugin>/<type>/<filename>` でアクセスできるようになります。

例えば `plugins/as_hoge/app/assets/stylesheets/style.css` にファイルを配置すると `/plugin/as_hoge/stylesheets/sytle.css` でアクセスできます。

.. _auth:

認証方法の変更 (0.8.1 以降)
--------------------------------------

OmniAuth による認証の切り替え
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

AsaksuaSatellite へのログイン時の認証は `OmniAuth <https://github.com/intridea/omniauth>`_ を使用しています。
デフォルトではプロバイダとして Twitter の OAuth を利用していますが、プラグインを追加することで切り替えが可能となっています。

既存の OmniAuth Strategy を使用した認証の切り替え
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`公開されている OmniAuth Strategy <https://github.com/intridea/omniauth/wiki/List-of-Strategies>`_
を利用することで、認証プラグインを作成することができます。

以下、プラグインの作成手順について説明します。

1. プラグインの作成

<AS_ROOT>/plugins ディレクトリの直下に任意のディレクトリを作成し、その直下に Gemfile を以下のとおり作成します。

::

    gem 'omniauth-XXXX' # 利用する OmniAuth Strategy の gem

例えば、 `Github の OAuth による認証 <https://github.com/intridea/omniauth-github>`_ を利用する場合は以下のようになります。

::

    gem 'omniauth-github'


2. 依存 gem の再インストール

<AS_ROOT> ディレクトリに移動し、以下のコマンドを入力して、依存 gem をインストールしなおします.

::

    $ bundle install --path .bundle --without development test

3. 設定

<AS_ROOT>/config/settings.yml の "omniauth" の設定項目を修正して AsakusaSatellite で利用する認証を変更します。

::

    omniauth:
      provider: "プロバイダ"
      provider_args:
        - "引数1"
        - "引数2"
        - "..."

設定内容は以下の通りです。

* **provider** (必須): 使用する OmniAuth Strategy の名称を記述します。
* **provider_args** (任意): 使用する OmniAuth Strategy の設定時に渡される引数を配列で指定します。

例えば Github の OAuth による認証を利用する場合は以下のとおりとなります

::

    omniauth:
      provider: "github"
      provider_args:
        - "<Client ID>"
        - "<Client Secret>"

provider と provider_args に渡す値については各 OmniAuth Strategy を参照してください。

4. AsakusaSatellite の再起動

AsakusaSatellite を再起動することで認証が切り替わります。

既存の OmniAuth Strategy を使用しているプラグインは以下の通りです。

* `Twitter 認証 プラグイン <https://github.com/codefirst/AsakusaSatellite/tree/master/plugins/as_twitterauth_plugin>`_ 


独自 OmniAuth Strategy による認証
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OmniAuth Strategy を自作することが可能です。

独自 OmniAuth Strategy 作成の詳細については
`Strategy Contribution Guide <https://github.com/intridea/omniauth/wiki/Strategy-Contribution-Guide>`_
を参照してください。

以下、プラグインの作成手順について説明します。

1. OmniAuth Strategy クラスを作成する.

OmniAuth の規約にしたがって OmniAuth::Strategies モジュール以下にクラスを作成します。
<AS_ROOT>/plugins ディレクトリの直下に任意のディレクトリを作成し、lib/omniauth/strategies/mystrategy.rb を以下のように作成します。

.. code-block:: ruby

    module OmniAuth
      module Strategies
        class Mystrategy
          include OmniAuth::Strategy

          args [:arg1, :arg2] # provider_args で渡される引数

          def request_phase
            ...
          end

          def callback_phase
            ...
          end

          info {
            {:name => '....', :nickname => '....', :image => 'http://....'}
          }

        end
      end
    end

`OmniAuth Strategy <https://github.com/intridea/omniauth/wiki/Auth-Hash-Schema>`_ に従い、info で取得できる値を作成します。

AsakusaSatellite で使用する値は :name, :nickname および :image です。
それぞれの意味は以下の通りです。

* **:name** : ユーザを一意に識別する ID として使用します。
* **:nickname** : ユーザの表示名として使用します。
* **:image** : ユーザの発言などに付加される画像ファイルの場所を特定するために使用します。

AsakusaSatellite プラグインは、 app ディレクトリ以下に Rails の controller, view を独自に作成できるため、
request_phase メソッドの実装で独自に作成したページにリダイレクトすることにより、独自の認証フォームを作成することも可能です。

2. 独自 Strategy を読み込む

プラグインディレクトリの直下に init.rb ファイルを以下のように作成します。

.. code-block:: ruby

    require 'omniauth/strategies/mystrategy'

3. 設定

<AS_ROOT>/config/settings.yml の "omniauth" の設定項目を修正して AsakusaSatellite で利用する認証を変更します。

::

    omniauth:
      provider: "mystrategy" # Strategy 名
      provider_args: # Strategy に渡す値
        - "引数1"
        - "引数2"

4. AsakusaSatellite の再起動

AsakusaSatellite を再起動することで認証が切り替わります。

独自 OmniAuth Strategy を使用しているプラグインは以下の通りです。

* `ローカル認証プラグイン <https://github.com/codefirst/AsakusaSatellite/tree/master/plugins/as_localauth_plugin>`_
* `Redmine 認証プラグイン <https://github.com/codefirst/AsakusaSatellite/tree/master/plugins/as_redmineauth_plugin>`_

UserScript で AsakusaSatellite のイベントを取得する方法 (0.8.1 以降)
----------------------------------------------------------------------------

AsakusaSatellite は、UserScript が安全にクロスドメイン制約を回避できるようにするため、
window.postMessage を用いて各種イベントを通知します。

通知されるイベントの種類と含まれる情報
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

通知されるイベントは type, current, data の3つの情報を持っています。
current には現在の部屋の ID とユーザの ID が含まれます。
data の内容は type に応じて変化します。

type の種類と、data に含まれる情報は以下です。

.. glossary::

  connect
    websocket サーバとの接続が確立した際に通知されます。
    使用している websocket サーバにより含まれる情報は異なります。

  error
    websocket サーバとの接続でエラーが発生した際に通知されます。
    使用している websocket サーバにより含まれる情報は異なります。

  disconnect
    websocket サーバとの接続が切断された際に通知されます。
    使用している websocket サーバにより含まれる情報は異なります。

  create
    新規メッセージを受信した際に通知されます。
    新着メッセージの全情報が含まれます。詳しくは Message モデルを参照ください。

  update
    メッセージ更新された際に通知されます。
    新着メッセージの全情報が含まれます。詳しくは Message モデルを参照ください。

  delete
    メッセージが削除された際に通知されます。
    削除されたメッセージの ID が含まれます。


イベントハンドラのサンプル
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下のように message イベントに対してイベントハンドラを登録し、
イベント内の type の値で処理を切り替えてください。

::

    window.addEventListener('message', function(e){
        switch(e.data.type) {
        case "create":
            ...
        case "update":
            ...
        }
    });
