---
layout: classic-docs
title: iOS アプリケーションのデプロイ
short-title: iOS アプリケーションのデプロイ
categories:
  - プラットフォーム
description: iOS アプリケーションのデプロイ
order: 1
version:
  - Cloud
---

ここでは、CircleCI 上で iOS アプリを配信サービスに自動的にデプロイするための [fastlane](https://fastlane.tools/) の設定方法について説明します。

* 目次
{:toc}

## 概要
{: #overview }
{:.no_toc}

fastlane を使用して、iOS アプリを様々なサービスに自動的にデプロイすることができます。 これにより、iOS アプリのベータ版またはリリース版を対象ユーザーに配信する際の手動作業が不要になります。

デプロイレーンをテストレーンと組み合わせることで、ビルドとテストが成功したアプリが自動的にデプロイされます。

**注:** 下記のデプロイ例を使用するには、お客様のプロジェクトにコード署名が設定されている必要があります。 To learn how to set up code signing, see the [Setting Up Code Signing]({{site.baseurl}}/2.0/ios-codesigning/) page.

## ベストプラクティス
{: #best-practices }

### Git ブランチの使用
{: #using-git-branches }

リリースレーンは、Git リポジトリの特定のブランチでのみ実行することをお勧めします。例えば、専用のリリース/ベータブランチなどです。 そうすることで、指定したブランチへのマージが成功した場合にのみリリースが可能となり、開発期間中にプッシュがコミットされるたびにリリースが行われることを防ぐことができます。 また、iOSアプリのバイナリのサイズによっては外部サービスへのアップロードに時間がかかる場合があるため、ジョブ完了までの時間を短縮することができます。 For information on how to set up a workflow to achieve this, check out the [Branch-Level Job Execution]({{site.baseurl}}/2.0/workflows/#branch-level-job-execution) page.

### ビルド番号の設定
{: #setting-the-build-number }

デプロイサービスにアップロードする際には、iOS アプリのバイナリのビルド番号を考慮することが重要です。 一般的には、 `.xcproject` で設定されていますが、一意になるように手動で更新する必要があります。 各デプロイレーンの実行前にビルド番号が更新されていない場合、受信サービスがビルド番号の競合によりバイナリを拒否することがあります。

Fastlane provides an [`increment_build_number` action](https://docs.fastlane.tools/actions/increment_build_number/) which allows the build number to be modified during the lane execution. たとえば、特定の CircleCI ジョブにビルド番号を関連付けたい場合は、 環境変数 `$CIRCLE_BUILD_NUM` の使用を検討してください。

```ruby
increment_build_number(
  build_number: "$CIRCLE_BUILD_NUM"
)
```

## App Store Connect
{: #app-store-connect }

### 設定
{: #setting-up }

fastlane が iOS バイナリを App Store Connect や TestFlight に自動的にアップロードするように設定するには、fastlane が App Store Connect アカウントにアクセスできるよういくつかのステップを実施する必要があります。

この設定には、App Store Connect APIキーを生成して使用することをお勧めします。 それにより、Apple ID で必須となっている 2FA で問題が発生することを防ぎ、最も確実な方法でサービスを利用することができます。

API キーを作成するには、 [Apple 開発者向けドキュメント](https://developer.apple.com/documentation/appstoreconnectapi/creating_api_keys_for_app_store_connect_api)で説明されている手順に従ってください。 その結果 `.p8` を取得したら、[App Store Connect API キーのページ](https://appstoreconnect.apple.com/access/api)に表示される*発行者 ID* と*キー ID* をメモします。

**注意:** `.p8` ファイルをダウンロードし、安全な場所に保存したことを確認してください。 App Store Connect のポータルから離れてしまうと、ファイルを再度ダウンロードすることはできません。

次に、いくつかの環境変数を設定する必要があります。 CircleCI プロジェクトで、 **ビルド設定 > 環境変数** に移動し、以下を設定します。

* `APP_STORE_CONNECT_API_KEY_ISSUER_ID` to the Issuer ID
  * (例：`6053b7fe-68a8-4acb-89be-165aa6465141`)
* `APP_STORE_CONNECT_API_KEY_KEY_ID` to your Key ID
  * (例: `D383SF739`)
* `APP_STORE_CONNECT_API_KEY_KEY` to the contents of your `.p8` file
  * (例: `-----BEGIN PRIVATE KEY-----\nMIGTAgEAMGByqGSM49AgCCqGSM49AwEHBHknlhdlYdLu\n-----END PRIVATE KEY-----`)

**注意:** `.p8` ファイルの内容を確認するには、テキストエディターで開きます。 各行を `\n` に置き換えて、1つの長い文字列にする必要があります。

最後に、fastlane ではどの Apple ID を使用するか、またどのアプリの識別子をターゲットにするかを知るために、いくつかの情報が要求されます。 これらの情報は、 `fastlane/Appfile` で以下のように設定できます。

```ruby
# fastlane/Appfile
apple_id "ci@yourcompany.com"
app_identifier "com.example.HelloWorld"
```

Once this is configured, you just need to call [`app_store_connect_api_key`](http://docs.fastlane.tools/actions/app_store_connect_api_key/#app_store_connect_api_key) in your lane before calling any actions that interact with App Store Connect (such as `pilot` and `deliver`).

### App Store へのデプロイ
{: #deploying-to-the-app-store }

下記の例は、バイナリをビルドして署名し、App Store Connect にアップロードする基本的なレーンです。 The [`deliver` action](http://docs.fastlane.tools/actions/deliver/#deliver/) provided by Fastlane is a powerful tool that automates the App Store submission process.

また、メタデータやスクリーンショット ([screenshot](https://docs.fastlane.tools/actions/snapshot/) や [frameit](https://docs.fastlane.tools/actions/frameit/) アクションで生成可能) を自動的にアップロードするなど、さまざまなオプションが可能です。 設定の詳細については、fastlane の [配信に関するドキュメント](https://docs.fastlane.tools/actions/deliver/)を参照してください。

```ruby
# fastlane/Fastfile
default_platform :ios

platform :ios do
  before_all do
    setup_circle_ci
  end

  desc "Upload Release to App Store"
  lane :upload_release do
    # プロジェクトからバージョン番号を取得します。
    # App Store Connect で既に使用可能な最新のビルドと照合します。
    # ビルド番号を１増やします。 使用可能なビルドがない場合は、
    # 1 から始めます。
    increment_build_number(
      build_number: app_store_build_number(
        initial_build_number: 1,
        version: get_version_number(xcodeproj: "HelloCircle.xcodeproj"),
        live: false
      ) + 1,
    )
    # 配信コード署名を設定し、アプリをビルドします。
    match(type: "appstore")
    gym(scheme: "HelloCircle")
    #App Store Connect にバイナリをアップロードします。
    deliver(
      submit_for_review: false,
      force: true
    )
  end
end
```

### TestFlight へのデプロイ
{: #deploying-to-testflight }

TestFlight は、App Store Connect と連動した Apple のベータ版配信サービスです。 Fastlane provides the [`pilot` action](https://docs.fastlane.tools/actions/pilot/) to make managing TestFlight distribution simple.

下記の例では、 iOS バイナリを自動的にビルド、署名、アップロードするように fastlane を設定する方法を紹介しています。 Pilot には TestFlight にアプリを配信するためのカスタムオプションがたくさんあります。その詳細を [Pilot のドキュメント](https://docs.fastlane.tools/actions/pilot/)で詳細をぜひご確認ください。

```ruby
# fastlane/Fastfile
default_platform :ios

platform :ios do
  before_all do
    setup_circle_ci
  end

  desc "Upload to Testflight"
  lane :upload_testflight do
    # プロジェクトからバージョン番号を取得します。
    # TestFlight で既に利用可能な最新のビルドと照合します。
    # ビルド番号を 1 増やします。 使用可能なビルドがない場合は、
    # 1 から始めます。
    increment_build_number(
      build_number: latest_testflight_build_number(
        initial_build_number: 1,
        version: get_version_number(xcodeproj: "HelloWorld.xcodeproj")
      ) + 1,
    )
    # 配信コード署名を設定し、アプリをビルドします。
    match(type: "appstore")
    gym(scheme: "HelloWorld")
    # TestFlight にバイナリをアップロードし、
    # 設定したベータ版のテストグループに自動的にパブリッシュします。
    pilot(
      distribute_external: true,
      notify_external_testers: true,
      groups: ["HelloWorld Beta Testers"],
      changelog: "This is another new build from CircleCI!"
    )
  end
end
```

## Firebase へのデプロイ
{: #deploying-to-firebase }

Firebaseは、Google が提供する配信サービスです。 Firebase へのデプロイは、 [Firebase アプリ配信プラグイン](https://github.com/fastlane/fastlane-plugin-firebase_app_distribution)をインストールすることで簡単に行うことができます。

### Fastlane プラグインの設定
{: #fastlane-plugin-setup }

To set up the plugin for your project, on your local machine, open your project directory in Terminal and run the following command:
```bash
fastlane add_plugin firebase_app_distribution
```
するとプラグインがインストールされ、必要な情報が `fastlane/Pluginfile` と `Gemfile` に追加されます。

**注意:** `bundle install` ステップにより、ジョブの実行中にこのプラグインをインストールできるよう両方のファイルを Git レポジトリに組み込んでおくことが重要です。

### CLI トークンの生成
{: #generating-a-cli-token }

Firebase では、認証時にトークンを使用する必要があります。 トークンを生成するには、Firebase CLI とブラウザを使用する必要があります。CircleCIはヘッドレス環境であるため、ランタイムではなくローカルでトークンを生成し、環境変数として CircleCI に追加する必要があります。

1. コマンド `curl -sL https://firebase.tools | bash`で、Firebase CLI をダウンロードしてローカルにインストールします。
2. `firebase login:ci`というコマンドでログインをトリガーします。
3. ブラウザウィンドウでサインインを完了し、ターミナルの出力で提供されたトークンをコピーします。
4. CircleCI のプロジェクト設定で、 `FIREBASE_TOKEN` という名前の新しい環境変数を作成し、トークンの値を入力します。

### Fastlane の設定
{: #fastlane-configuration }

Firebase プラグインは、最小限の設定で iOS のバイナリを Firebase にアップロードすることができます。 主なパラメータは `app` で、Firebase が設定した App ID が必要になります。 To find this, go to your project in the [Firebase Console](https://console.firebase.google.com), then go to **Project Settings -> General**. Under **Your apps**, you will see the list of apps that are part of the project and their information, including the App ID (generally in the format of `1:123456789012:ios:abcd1234abcd1234567890`).

その他の設定オプションについては、 [Firebase Fastlane プラグインのドキュメント](https://firebase.google.com/docs/app-distribution/ios/distribute-fastlane#step_3_set_up_your_fastfile_and_distribute_your_app)を参照してください。

```ruby
# Fastlane/fastfile
default_platform :ios

platform :ios do
  before_all do
    setup_circle_ci
  end

  desc "Upload to Firebase"
  lane :upload_firebase do
    increment_build_number(
      build_number: "$CIRCLE_BUILD_NUM"
    )
    match(type: "adhoc")
    gym(scheme: "HelloWorld")
    firebase_app_distribution(
      app: "1:123456789012:ios:abcd1234abcd1234567890",
      release_notes: "This is a test release!"

    )
  end
end
```

Firebase Fastlane のプラグインを使用するには、 `curl -sL https://firebase.tools | bash` コマンドにより Firebase CLI をジョブの一部としてインストールする必要があります。

```yaml
version: 2.1
jobs:
  adhoc:
    macos:
      xcode: "12.5.1"
    environment:
      FL_OUTPUT_DIR: output
    steps:
      - checkout
      - run: echo 'chruby ruby-2.6' >> ~/.bash_profile
      - run: bundle install
      - run: curl -sL https://firebase.tools | bash
      - run: bundle exec fastlane upload_firebase

workflows:
  adhoc-build:
    jobs:
      - adhoc
```

**注意:** Firebase プラグインは、macOS システムの Ruby で実行するとエラーが発生することがあります。 It is therefore advisable to [switch to a different Ruby version]({{site.baseurl}}/2.0/testing-ios/#using-ruby).

## Visual Studio App Center へのデプロイ
{: #deploying-to-visual-studio-app-center }

[Visual Studio App Center](https://appcenter.ms/) (formally HockeyApp), is a distribution service from Microsoft.  [App Center のプラグイン](https://github.com/microsoft/fastlane-plugin-appcenter)をインストールすると、App Center と Fastlane の統合が可能になります。

### Fastlane プラグインの設定
{: #fastlane-plugin-setup }

To set up the plugin for your project, On your local machine open your project directory in Terminal and run the following command:
```bash
fastlane add_plugin appcenter
```
 するとプラグインがインストールされ、必要な情報が `fastlane/Pluginfile` と `Gemfile` に追加されます。

**注意:** `bundle install` ステップにより、ジョブの実行中にこのプラグインをインストールできるよう両方のファイルを Git レポジトリに組み込んでおくことが重要です。

### App Center の設定
{: #app-center-setup }

まず、VS App Center でアプリを作成する必要があります。

1. [Visual Studio App Center](https://appcenter.ms/)にログイン、またはサインアップします。
2. ページの右上にある [Add New (追加)] "をクリックし、[Add New App (新しいアプリを追加する)] を選択します。
3. 必要に応じて、必要な情報を入力します。

完了したら、Fastlane が App Center にアップロードできるようにするための API トークンを生成する必要があります。

1. 設定の [API トークン](https://appcenter.ms/settings/apitokens) に移動します。
2. [New API Token (新しい API トークン)] "をクリックします。
3. トークンの説明を入力し、アクセスを [Full Access (フルアクセス)] に設定します。
4. When the token is generated, make sure to copy it somewhere safe
5. Go to your project settings in CircleCI and create a new environment variable named `VS_API_TOKEN` with the value of the API Key

### fastlane の設定
{: #fastlane-configuration }

下記は、ベータ版アプリのビルドを Visual Studio App Center に配信するレーンの例です。 App Center にバイナリをアップロードするには、App Center アカウントのユーザー名と「フルアクセス」 の API トークンの両方が必要です。

```ruby
# Fastlane/fastfile
default_platform :ios

platform :ios do
  before_all do
    setup_circle_ci
  end

desc "Upload to VS App Center"
  lane :upload_appcenter do
    #  ここでは ビルド番号に
    # CircleCI ジョブ番号を使います。
    increment_build_number(
      build_number: "$CIRCLE_BUILD_NUM"
    )
    # Adhoc コード署名を設定し、アプリをビルドします。
    match(type: "adhoc")
    gym(scheme: "HelloWorld")
    # 必要な情報を設定し、
    # アプリのバイナリを VS App Center にアップロードします。
    appcenter_upload(
      api_token: ENV[VS_API_TOKEN],
      owner_name: "YOUR_VS_APPCENTER_USERNAME",
      owner_type: "user",
      app_name: "HelloWorld"
    )
  end
end

```

## TestFairy へのアップロード
{: #uploading-to-testfairy }

[TestFairy](https://www.testfairy.com) は、よく使用されるエンタープライズアプリの配信およびテストサービスです。 Fastlane には TestFairy のサポートが組み込まれており、新しいビルドを迅速かつ簡単にアップロードすることができます。

![TestFairy の設定]({{site.baseurl}}/assets/img/docs/testfairy-open-preferences.png)

1. TestFairy ダッシュボードで、[Preferences (設定)] ページに移動します。
2. On the Preferences page, go to the API Key section and copy your API Key
3. Go to your project settings in CircleCI and create a new environment variable named `TESTFAIRY_API_KEY` with the value of the API Key

### fastlane の設定
{: #fastlane-configuration }

fastlane 内で TestFairy へのアップロードを設定するには、次の例を参照してください。

```ruby
# Fastlane/fastfile
default_platform :ios

platform :ios do
  before_all do
    setup_circle_ci
  end

desc "Upload to TestFairy"
  lane :upload_testfairy do
    # ここではビルド番号に
    # CircleCI ジョブ番号を使います。
    increment_build_number(
      build_number: "$CIRCLE_BUILD_NUM"
    )
    # Adhoc コード署名を設定し、アプリをビルドします。
    match(type: "adhoc")
    gym(scheme: "HelloWorld")
    # 必要な情報を設定し、
    # アプリのバイナリを VS App Center にアップロードします。
    testfairy(
      api_key: ENV[TESTFAIRY_API_KEY],
      ipa: 'path/to/ipafile.ipa',
      comment: ENV[CIRCLE_BUILD_URL]
    )
  end
end
```
