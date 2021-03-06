WebSocketサーバの変更
=======================

<AS_ROOT>/config/message_pusher.yml により、AsakusaSatelliteで利用するWebSocketサーバを変更できます。

現時点ではSocky, Pusher, Keima の 3 方式をサポートしています。

Socky
------------------------------

設定方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

デフォルト設定のため特に設定せずに利用できます。

利点・欠点
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

外部サービスを利用せずに動作するため、社内での利用等に適しています。

ただし、WebSocket以外の通信方式には対応していません。


Pusher
------------------------------

Pusher ( http://pusher.com ) を利用して通信します。

設定方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

http://pusher.com に登録して、app_id, key, secretを取得します。

<AS_ROOT>/config/message_pusher.yml を以下のように編集します。

::

    default: pusher

    engines:
      # pusherの設定
      pusher:
        app_id: <your app id>
        key:    <your key>
        secret: <your secret>

AsakusaSatellite を再起動することで設定が反映されます。

利点・欠点
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

有料プランを利用すれば多数のユーザの同時使用にも耐えることができます。 また、WebSocketだけでなくFlashSocketによる接続も可能なため、WebSocket非対応のブラウザからも利用できます。

ただし、外部サービスを利用するため、社内での利用等には向きません。

Keima
------------------------------

Pusher クローンである Keima ( https://github.com/codefirst/keima ) を利用して通信します。

設定方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

keima にアクセスし、appkey を取得します。

<AS_ROOT>/config/message_pusher.yml を以下のように編集します。

::

    default: keima
    
    engines:
      url: <your keima url>
      key: <your key>

AsakusaSatellite を再起動することで設定が反映されます。

利点・欠点
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

社内のネットワーク等に設置した Keima を利用できるため、セキュリティを確保することができます。

また、Pusher 利用時と同様に、WebSocket 非対応のブラウザからも利用が可能です。

