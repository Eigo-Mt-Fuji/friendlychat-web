# README

![capture](./capture.png)

## Commands

```
$ npm -g install firebase-tools
$ firebase --version
$ firebase login
$ firebase use --add
$ firebase serve
$ firebase deploy
```

## Output

https://friendlychat-f6c3a.firebaseapp.com

## 詰まった点
* Cloud Authentication
  Signin しようとすると、403エラーが表示されて、Google Provider認証がエラーになる-> OAuthへ同意する。アプリケーション ホームページ] リンクと[アプリケーション プライバシー ポリシー] リンクが必要
  - [API とサービス / 認証情報](https://console.developers.google.com/apis/credentials/consent?project={xxxx})

* Cloud FireStore
  * エラー `writing new message to Firebase Database FirebaseError: Missing or insufficient permissions` の対処
    - Filre store ruleで request.authがnullではない場合だけ許可するように書き換え
    - test modeで作っていれば発生しなかった模様だが、
    - Firestoreのデータベースを初期設定する際のモード選択で、lock modeを選んで作成したことで発生
    - FirestoreのRule初期設定が、test mode(Codelabで指示された方)を選んだ場合とlock modeを選んだ場合で異なる模様
       - https://console.firebase.google.com/u/1/project/friendlychat-f6c3a/database/firestore/rules

    - 以下のように変更した
    
    ```
    rules_version = '2';
    service cloud.firestore {
        match /databases/{database}/documents {
            match /{document=**} {
                allow read, write: if false;
            }
        }
    }

    ↓

    rules_version = '2';
    service cloud.firestore {
    match /databases/{database}/documents {
        match /{document=**} {
        allow read, write: if request.auth != null;
        }
    }
    }
    ```

  * Firestore内のデータ構造の単位
    - collections, documents, fields, and subcollections
  * FirestoreにSDK経由で最初のメッセージ書き込みを行ったタイミングで自動でmessagesコレクションが作成されたのを確認した
     - messagesコレクションは、あらかじめFireBaseのコンソールで作っておかなくてもOK。
        - messagesコレクションはConsoleで作成する場合は、１つ目のdocumentを作成させられる。
  

* Developer Console Memo

```
[2019-07-21T04:09:44.477Z]  @firebase/firestore: Firestore (6.3.0): 
  The timestampsInSnapshots setting now defaults to true and you no
  longer need to explicitly set it. In a future release, the setting
  will be removed entirely and so it is recommended that you remove it
  from your firestore.settings() call now.
```

* Cloud Storage:  is better suited for storing files

* Cloud Messaging 
  - publicディレクトとりに配置するmanifest.jsonファイルの中に記載するgcm_sender_idはFirebase Cloud Messagingを使う場合、 `103953800507` 
    - 変えてはいけない

```
Unable to get messaging token. FirebaseError: Messaging: We are unable to register the default service worker. Failed to register a ServiceWorker: A bad HTTP response code (404) was received when fetching the script. (messaging/failed-serviceworker-registration).
```

* Performance Monitoring

  - 早速FirebaseのConsole上で確認してみたが、表示されない。
    - 最初のデータは 12時間程度待つと表示されるとのこと。
  - そもそもログが飛んでいるのかの確認は、ChromeのDeveloperコンソールでログをみる
    - FriendlychatをデプロイしたfirebaseのホストURLをChromeで開く
      - https://friendlychat-f6c3a.firebaseapp.com
    - ChromeのDeveloperコンソールを開いて、Networkタブをチェック
      - firebaselogging.googleapis.comに対してリクエストが送信されていることを確認できればOK
         - https://firebaselogging.googleapis.com/v0cc/log?format=json_proto
    - 送信されてなければ、以下のfirebase-performance.jsの読み込みとオブジェクトの初期化が行われてるか再確認

```
<script src="/__/firebase/6.3.0/firebase-performance.js"></script>
<script>
  var perf = firebase.performance();
</script>
```
    - 


## Note

Firebaseサンプル https://firebase.google.com/docs/samples/?hl=ja の中で https://efgriver.slack.com/archives/C8KFR9Q68/p1563625823000300 に関連する機能1 (開発)

```
* Authentication 認証API(Web)
* Cloud Firestore NoSQL データベース(新) - Realtime Database NoSQLデータベース JSON保管(旧)(Web)
* Cloud Messaging ユーザーに対して表示される通知メッセージを送信, 個々のデバイスに、デバイス グループに、または特定トピックの配信登録をしているデバイスに(Web)
* Dynamic Links(Invites)(Native) 独自ドメインでアプリスキーム だけでなく、Invitesが提供していた機能も提供アプリのインストールを挟んでも有効な、クロスプラットフォームの招待リンク。共有画面で自分の Google コンタクトと端末にローカル保存された連絡先から受信者を選択 /  SMS で送信され、アプリへのダイナミック リンクを送る
* Cloud Storage 写真や動画など、ユーザーが作成したコンテンツを保管
```

上記以外で、アプリ運営に必要な機能

```
AdMob    モバイル広告　AdMob API を使用して、アプリの UI にバナー広告用のスペースを作成
アナリティクス アクセス解析
App Indexing    アプリを Google 検索結果に表示することができる
Performance Monitoring パフォーマンス分析
Cloud Functions FaaS
```

やること

## Firebaseの基本
* [Friendlychat](https://codelabs.developers.google.com/codelabs/firebase-web/#14)

## Android/iPhoneアプリを作るには?
* WebViewを表示できるようになるまで雛形が必要
* Firebase Webで使ったFirebaseプロジェクトを、iOS/Androidでも使うことはできるのか。
 - 少なくとも、Dynamic LinksはAndroid/iOSのSDKで使う
* 参考になりそうなサイト
 * Android: android-professional-website-app-with-firebase-backend
   - https://codecanyon.net/item/android-professional-website-app-with-firebase-backend-and-admob/19202563
 * iPhone
   - https://www.sitepoint.com/creating-a-firebase-backend-for-ios-app/ (e