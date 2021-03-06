# imaclab Laravel Style Guide

*記錄了 imaclab 實作 Laravel Application 時的 Style Guide。*

# 建構中...

<a name="table-of-contents"></a>
## 目錄

  1. [目錄結構]()
    - [應用程式核心]()
    - [其他]()
  1. [第三方套件]()
    - [require]()
    - [require-dev]()
  1. [Coding Style]()
    - [程式碼相關]()
    - [命名相關]()
      - [資料表與欄位]()
      - [Migration]()
      - [Seeder]()
      - [類別與方法]()
      - [變數]()

## 目錄結構
### 應用程式核心
```
app
├── Console - 自訂 Command
│   └── Commands
├── Events - 事件（對應 Listeners）
├── Exceptions - 自訂例外
├── Http
│   ├── Controllers
│   ├── Middleware - 所有的中介層
│   └── Requests - 驗證的 Request 類別
├── Jobs - 隊列排程
├── Listeners - 監聽（對應 Events 及 Jobs（Queue））
├── Models - 所有的 Model
│   ├── Services - 所有 Service
│   ├── Entities - 所有 Eloquest Model
│   ├── Presenters - 所有顯示的 Presenters
│   └── Repositories - 所有封裝的 Repository
└── Providers - 框架的 Service Provider
```
### 其他
```
├── bootstrap - 框架啟動的核心
├── config - 框架的設定檔案
├── database - 資料庫相關
│   ├── factories - 資料工廠
│   ├── migration - 遷移
│   └── seeds - 資料填充
├── packages - 因該專案而開發的套件
├── public - 公有資料夾，存放公有的 Assets
│   ├── imgs - 圖片
│   ├── css - Css
│   ├── fonts - 字型
│   └── js - JavaScript
├── resources - 資源
│   ├── assets - 需要 transpile 的資源檔
│   │   ├── sass
│   │   ├── css
│   │   └── js
│   ├── lang - 多國語言
│   └── views - 視圖
├── storage - 框架的快取等檔案
├── tests - Unit Test
└── vendor - 所有的框架核心及 Package
```
## 第三方套件
初始化 Project 時請加上下述 Package，並查閱對應文件加上 Service Provider 及 Facade
### require
- `barryvdh/laravel-debugbar`
  - https://github.com/barryvdh/laravel-debugbar
  - 用於查看 booting time 及 query statement 或顯示變數用
- `guidovanbiemen/laravel-whoops`
  - https://github.com/guidovanbiemen/laravel-whoops
  - Laravel 5 被移除的 whoops error handling
- `laracasts/presenter`
  - https://github.com/laracasts/Presenter
  - 實作 Presenter pattern 的 Package
- `laravelcollective/html`
  - http://laravelcollective.com/docs/5.1/html
  - Laravel 5 的 Form Builder

### require-dev
- `laracasts/generators`
  - https://github.com/laracasts/Laravel-5-Generators-Extended
  - 加強官方 artisan 的 generator 功能
- `doctrine/dbal`
  - https://github.com/doctrine/dbal
  - 配合 ide-helper 產出 model 用於 autocomplate 的註解
- `barryvdh/laravel-ide-helper`
  - https://github.com/barryvdh/laravel-ide-helper
  - dump 出 Class 結構的檔案，幫助 IDE 做 autocomplate
  - 在 composer.json 的 scripts 加入以下程式碼，即可使用 
    composer run-script ide-helper 產生檔案

```
...
"scripts": {
    ...
    "ide-helper": [
        "php artisan clear-compiled",
        "php artisan ide-helper:generate",
        "php artisan ide-helper:meta",
        "php artisan ide-helper:models --write",
        "php artisan optimize"
    ]
},
...
```
評估中

- https://github.com/prettus/l5-repository
- https://github.com/Bosnadev/Repositories

## Coding Style
### 程式碼相關
- 所有程式碼請參照 PSR-1 及 PSR-2 規範，可使用 phpcs 等工具協助
  - PSR-1 中文 http://blog.mosil.biz/2012/08/psr-1-basic-coding-standard/
  - PSR-2 英文 http://www.php-fig.org/psr/psr-1/
  - PSR-2 中文 http://blog.mosil.biz/2012/09/psr-2-basic-coding-standard/
  - PSR-2 英文 http://www.php-fig.org/psr/psr-2/

- 遵照 ARCA pattern
  - ARCA http://blog.turn.tw/?p=2290
  - Repository http://blog.turn.tw/?p=2521
  - Presenter http://blog.turn.tw/?p=2511

- 在 Class 中避免使用 Facade，請改用 DI 替代

```php
//bad
use Request;
...
public function store()
{
    $data = Request::all()
}

//good
use Illuminate\Http\Request;
...
public function store(Request $request)
{
    $data = $request->all()
}
```

### 命名相關
#### 資料表與欄位
- 資料表名稱請使用**複數英文**並使用**蛇底式命名法（snake_case）**

```php
// bad
Schema::create('User', ...
Schema::create('user', ...

// good
Schema::create('users', ...
```

- 多對多（或同性質）的 pivot table 請命名為`{單數資料表A}_{單數資料表B}`，並按照**英文字母順序排序**

```php
// bad
Schema::create('articles_tags', ...
Schema::create('article_tag_mapping', ...
Schema::create('tag_article', ...

// good
Scheam::create('article_tag', ...
```

- 資料表欄位請使用**蛇底式命名法（snake_case）**且一律**不加前綴字**

```php
//bad
$table->string('Name');
$table->string('u_name');

// good
$table->string('name');
```

- Primary Key 一律為 `id`

```php
// bad
$table->increment('uid');
$table->increment('pId');
$table->increment('p_id');

// good
$table->increment('id');
```

- Foreign Key 一律為`{單數資料表名稱}_id`

```php
// bad
$table->integer('UserId');
$table->integer('userId');

// good
$table->integer('user_id');
```

#### Migration
- 新建資料表的 migration 命名為 `create_{資料表名稱}_table`

```shell
// bad
php artisan make:migration users --create=users

// good
php artisan make:migration create_users_table --create=users
```

- 增加欄位至原有資料表的 migration 命名為 `add_{欄位名稱}_to_{資料表名稱}_table`

```shell
// bad
php artisan make:migration update_articles --table=users

// good
php artisan make:migration add_published_at_to_articles_table --create=users
```

#### Seeder
- Seeder 請命名為 `{帕斯卡命名的單數資料表名稱}TableSeeder`

```shell
// bad
php artisan make:seeder articles

// good
php artisan make:seeder ArticleTableSeeder
```

#### 類別與方法
- 所有類別名稱請使用**帕斯卡命名法（PascalCase）**

```php
// bad
class Articles_Repository {...}
class articlesRepository {...}

// good
class ArticlesRepository {...}
```

- 所有的 Resources Controller 名稱為**複數**

```php
// bad
class ArticleController extends Controller {...}

// good
class ArticlesController extends Controller {...}
```

- 所有 Eloquent Model 名稱為**單數**

```php
// bad
class Users extends Model {...}

// good
class User extends Model {...}
```

- 所有方法名稱請使用**駝峰式命名法（camelCase）**

```php
// bad
public function CreateArticle {...}
public function create_article {...}

//good
public function createArticle {...}
```

#### 變數
- 所有 PHP 變數名稱請使用**駝峰式命名法（camelCase）**

```php
// bad
$request_data;
$Request_Data;

//good
$requestData;
```

- 所有 HTTP 傳遞的參數名稱請使用**蛇底式命名法（snake_case）**

```php
// bad
{!! Form::text('addressSwitch') !!}

//good
{!! Form::text('address_switch') !!}
```


