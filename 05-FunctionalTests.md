# 機能テスト

今、私達はいくつかの受け入れテストを書いてきましたが、機能テストもほぼ同様に書けます。ひとつの大きな違いがあるだけです：機能テストは実行するのにWebサーバーを必要としないことです。

簡単に言えば、`$_REQUEST`や`$_GET`、`$_POST`の変数をセットし、テストからアプリケーションを実行します。このことは機能テストがより速く動作し、失敗時に詳細なスタックとレースを提供するので価値のあることでしょう。

Codeceptionは機能テストをサポートしている異なるフレームワークと接続出来ます:Symfony2, Laravel4, Yii2, Zend Frameworkなどです。あなたはただ求められるモジュールをfunctionalスイートの設定において使用できるようにするだけです。

これらのフレームワークモジュールは同じインターフェイスを持っているので、フレームワークにテストが縛られることはありません。以下は機能テストのサンプルです。

```php
<?php
$I = new FunctionalTester($scenario);
$I->amOnPage('/');
$I->click('Login');
$I->fillField('Username', 'Miles');
$I->fillField('Password', 'Davis');
$I->click('Enter');
$I->see('Hello, Miles', 'h1');
// $I->seeEmailIsSent() - Symfony2特有のメソッドです
?>
```

ご覧の通り、機能テストと受け入れテストで同じテストを使うことが出来ます。

## 落とし穴

通常、受け入れテストは機能テストよりも相当な時間がかかります。しかし機能テストはひとつの環境でCodeceptionとアプリケーションを実行できるほど安定しません。

#### ヘッダー、クッキー、セッション

機能テストの一般的な問題のひとつに、`headers`, `sessions`, `cookies`を扱うPHPメソッドの使い方があります。
ご存知のように、`header`メソッドは同じヘッダを2回以上実行するとエラーを発生させます。機能テストにおいてはアプリケーションを何回も実行するため、たくさんエラーが起こってしまうでしょう。

#### 共有メモリ

従来の方法とは違い、機能テストではリクエストの処理が終わった後にPHPアプリケーションを停止しません。
ひとつのメモリコンテナですべてのリクエストが実行されるので、リクエストが分離されることはありません。
したがって、**もしあなたが、失敗するはずないと思っているテストが何故か失敗するときは、単一のテストで実行してみてください。**
これはテストの実行中にそれぞれが分離されているかをチェックします。すべてのテストがメモリを共有して実行されていると容易に環境を壊してしまうからです。
メモリをきれいに保ち、メモリリークを避け、globalやstaticな変数をきれいにしてください。

## フレームワークモジュールを使う

機能テストスイートは`tests/functional`ディレクトリにあります。
はじめに、スイートの設定ファイル:`tests/functional.suite.yml` にフレームワークモジュールを一つ含める必要があります。下記にもっともポピュラーなPHPフレームワークで機能テストをセットアップする簡単な手順を提供しています。

### Symfony2

Symfony2で動作させるために、バンドルをインストールする必要も設定を変更することもありません。
テストスイートに`Symfony2`モジュールを追加するひつようがあるだけです。もしDoctrine2も使用するのであれば、忘れずに追加してください。

`functional.suite.yml`の書き方

```yaml
class_name: FunctionalTester
modules:
    enabled: [Symfony2, Doctrine2, TestHelper] 
```

デフォルトでは、このモジュールは`app`ディレクトリのApp Kernelを検索するでしょう。

モジュールは追加の情報とアサーションを提供するためにSymfony Profilerを使用します。

[詳しくはリファレンス全文を見てください。](http://codeception.com/docs/modules/Symfony2)

### Laravel 4

[Laravel](http://codeception.com/docs/modules/Laravel4) モジュールは設定も必要なく、簡単にセットアップ出来ます。

```yaml
class_name: FunctionalTester
modules:
    enabled: [Laravel4, TestHelper]
```


### Yii2

Yii2のテストは[Basic](https://github.com/yiisoft/yii2-app-basic) と [Advanced](https://github.com/yiisoft/yii2-app-advanced)アプリケーションテンプレートに含まれています。 始めるには、Yii2のガイドに従ってください。

### Yii

Yiiフレームワークは、単体では機能テストを動作させる仕組みを持っていません。
なので、CodeceptionがYiiフレームワークの最初で唯一の機能テストです。
Yiiで機能テストを使用するには設定に`Yii1`モジュールを追加してください。

```yaml
class_name: FunctionalTester
modules:
    enabled: [Yii1, TestHelper]
```

先ほど議論した落とし穴を避けるために、CodeceptionはYiiエンジン上に基盤となるフックを提供しています。
それを次にしたがってセットアップしてください。[the installation steps in module reference](http://codeception.com/docs/modules/Yii1)

### Zend Framework 2

Zend Framework 2の内部で機能テストを実行するには[ZF2](http://codeception.com/docs/modules/ZF2) モジュールを使用してください。

```yaml
class_name: FunctionalTester
modules:
    enabled: [ZF2, TestHelper]
```

### Zend Framework 1.x

Zend FrameworkのためのモジュールはPHPUnitの機能テストに使われるControllerTestCase classに強く影響を受けています。
ブートストラップとクリーンアップに同様のアプローチが使われています。機能テストでZend Frameworkを使用する場合は`ZF1`を追加してください。

`functional.suite.yml`の書き方

```yaml
class_name: FunctionalTester
modules:
    enabled: [ZF1, TestHelper] 
```

[詳しくはリファレンス全文を見てください。](http://codeception.com/docs/modules/ZF1)

### Phalcon 1.x

`Phalcon1`モジュールは`\Phalcon\Mvc\Application`のインスタンスを返すブートストラップファイルを作成する必要があります。Phalconで機能テストを始めるには、`Phalcon1`モジュール追加し、このブートストラップファイルにパスを通す必要があります。:

```yaml
class_name: FunctionalTester
modules:
    enabled: [Phalcon1, FunctionalHelper]
    config:
        Phalcon1
            bootstrap: 'app/config/bootstrap.php'
```

[詳しくはリファレンス全文を見てください。](http://codeception.com/docs/modules/Phalcon1)

## 機能テストを書く

機能テストは`PhpBrowser`モジュールでの[Acceptance Tests](http://codeception.com/docs/04-AcceptanceTests)と同じような方法で書かれます。すべてのフレームワークモジュールと`PhpBrowser`モジュールは同じメソッドと同じエンジンを共有しています。

したがって、私たちは`amOnPage`メソッドでウェブページを開くことが出来ます。

```php
<?php
$I = new FunctionalTester;
$I->amOnPage('/login');
?>
```

アプリケーションのページを開くリンクをクリックします。

```php
<?php
$I->click('Logout');
// .nav要素のリンクをクリックする
$I->click('Logout', '.nav');
// CSSセレクタによるクリック
$I->click('a.logout');
// strict locatorを使用したクリック
$I->click(['class' => 'logout']);
?>
```

フォームの送信も同様です。：

```php
<?php
$I->submitForm('form#login', ['name' => 'john', 'password' => '123456']);
// 同じ操作
$I->fillField('#login input[name=name]', 'john');
$I->fillField('#login input[name=password]', '123456');
$I->click('Submit', '#login');
?>
```

そしてアサーションをします。：

```php
<?php
$I->see('Welcome, john');
$I->see('Logged in successfully', '.notice');
$I->seeCurrentUrlEquals('/profile/john');
?>
```

フレームワークのモジュールはフレームワーク内部にアクセスするメソッドも含んでいます。例えば、`Laravel4`, `Phalcon1`, そして `Yii2`モジュールはデータベースにレコードが存在するかチェックするためのActiveRecord layerを使用する`seeRecord`メソッドを持っています。
`Laravel4`モジュールはセッションをチェックするメソッドも含んでいます。フォームのバリデーションをテストするときに`seeSessionHasErrors`メソッドは役に立つとわかるでしょう。

使用するモジュールの完全なリファレンスを見てください。モジュールのほとんどのメソッドはすべてに共通で、固有のメソッドは数種類です。

また、フレームワーク内部globalないし、`FunctionalHelper`クラスの依存関係注入コンテナの内部へもアクセス出来ます。

```php
<?php
class FunctionalHelper extends \Codeception\Module
{
    function doSomethingWithMyService()
    {
        $service = $this->getModule('Symfony2') // Symfony 2モジュールを調べてください
            ->container // 現在のDI containerを取得get current DI container
            ->get('my_service'); // access a service

        $service->doSomething();
    }
}
?>
```

Symfony2内部のカーネルにアクセスし、サービスコンテナを取得しました。私たちはテストで使用出来る`FunctionalTester`クラスのメソッドをカスタムして作ることも出来ます。

使用するモジュールに対応するリファレンスの*Public Properties*の節をチェックすることで、もっとフレームワークにアクセスする情報を得ることが出来ます。

## エラーレポート

Codeceptionはデフォルトで`E_ALL & ~E_STRICT & ~E_DEPRECATED`のエラー出力レベルを使用しています。
機能テストの中で、このエラーレベルを変更したい場合はフレームワークごとの方針に従ってください。
エラー出力レベルはスイートの設定ファイルで設定する事が出来ます：

```yaml
class_name: FunctionalTester
modules:
    enabled: [Yii1, TestHelper]
error_level: "E_ALL & ~E_STRICT & ~E_DEPRECATED"
```

`error_level`は`codeception.yml`ファイルにグローバルに定義することも出来ます。


## 結論

パワフルなフレームワークを使用しているならば、機能テストはすばらしいです。機能テストを使用する事で、フレームワークの内部状態にアクセスでき、制御する事が出来ます。
このことはテストをより短く、より速くしてくれます。フレームワークを使用しない場合ならば、機能テストを書く実用的な理由はないです。
もしここで挙げたものと違うフレームワークを使用しているならば、モジュールを作成して、コミュニティで共有してください。
