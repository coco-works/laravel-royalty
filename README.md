# Laravel Royalty（ララベル ロイヤルティ）

Laravelでリアルタイム対応の報酬・ロイヤリティ・経験ポイントを付与できるユーザーポイントパッケージです。
これはFork版です。
---

## 仕組みについて

各ポイントは「アクションファイル」と結び付いており、そのアクションファイルを呼び出すことで、対象のモデルにポイントを付与します。

アクションファイルを使う理由：

- ポイントの付与履歴を追跡しやすい
- 付与ポイント数の変更が簡単（例：50 → 100 にしたいとき、新しいアクションを作って差し替えるだけ）

---

## インストール

```
composer require miracuthbert/laravel-royalty
```

---

## セットアップ

Laravelのオートディスカバリ機能に対応しているため、基本的にServiceProviderを手動で追加する必要はありません。

もしオートディスカバリを使用しない場合は、`config/app.php` に以下を追加してください：

```php
Miracuthbert\Royalty\RoyaltyServiceProvider::class
```

### 自動セットアップコマンド

```
php artisan royalty:setup
```

このコマンドで `config` と `migrations` を自動で公開します。

> `config/royalty.php` 内の `model` キーを正しく設定してからマイグレーションを行ってください。

### 個別に公開する場合：

#### コンフィグの公開

```
php artisan vendor:publish --provider=Miracuthbert\Royalty\RoyaltyServiceProvider --tag=royalty-config
```

#### マイグレーションの公開

```
php artisan vendor:publish --provider=Miracuthbert\Royalty\RoyaltyServiceProvider --tag=royalty-migrations
```

#### Vueコンポーネントの公開

```
php artisan vendor:publish --provider=Miracuthbert\Royalty\RoyaltyServiceProvider --tag=royalty-components
```

---

## 使用方法

### ユーザーモデルの設定

1. `config/royalty.php` の `model` に対象ユーザーモデルのクラスを設定
2. モデルに `CollectsPoints` トレイトを追加：

```php
use CollectsPoints;
```

---

### アクションファイルの作成（ポイント定義）

#### Artisanコマンドで作成

```
php artisan royalty:action CompletedLesson
php artisan royalty:action Course\\CompletedLesson
```

- `--key` : キーを手動で指定
- `--name` : データベースにポイントを作成
- `--points` : ポイント数指定
- `--description` : 説明

#### 手動で作成

```php
namespace App\Royalty\Actions;

use Miracuthbert\Royalty\Actions\ActionAbstract;

class CompletedLesson extends ActionAbstract
{
    public function key()
    {
        return 'completed-lesson';
    }
}
```

---

### データベースにポイントを追加

```php
use Miracuthbert\Royalty\Models\Point;

Point::create([
    'name' => 'レッスン完了',
    'key' => 'completed-lesson',
    'description' => 'レッスンを完了した報酬',
    'points' => 100,
]);
```

---

### ポイントを付与

```php
$user = User::find(1);
$user->givePoints(new CompletedLesson());
```

---

### ユーザーポイントの取得

```php
$user->points()->number(); // 数値
$user->points()->shorthand(); // 例：1k, 2.5k
```

---

## リアルタイム反映

`PointsGiven` イベントが発火されます。

### チャンネル定義（routes/channels.php）

```php
Broadcast::channel('users.{id}', function ($user, $id) {
    return (int) $user->id === (int) $id;
});
```

### Vue.js + Laravel Echo の例

```js
Echo.private(`users.${this.userId}`)
    .listen('.points-given', (e) => {
        this.point = e.point;
        this.userPoints = e.user_points;
    });
```

### Vue コンポーネント使用例

```vue
<royalty-badge
  :user-id="{{ auth()->user()->id }}"
  initial-points="{{ auth()->user()->points()->shorthand() }}"
/>
```

---

## Artisan コマンド一覧

- `royalty:setup`: 初期セットアップを実行
- `royalty:action`: アクションファイル＆ポイント作成
- `royalty:actions`: ポイントと対応アクション一覧を表示

---

## クレジット

- [Cuthbert Mirambo](https://github.com/miracuthbert)

---

## ライセンス

このプロジェクトは [MIT ライセンス](https://opensource.org/licenses/MIT) のもとで公開されています。