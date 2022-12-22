# Laravel9 實作Jetstream 與 API Token

[![hackmd-github-sync-badge](https://hackmd.io/6UkNQjOtSGGWy4HDcXK4zg/badge)](https://hackmd.io/6UkNQjOtSGGWy4HDcXK4zg)

###### tags: `Laravel` `PHP`
## 安裝
### 安裝Jetstream
[套件官網](https://jetstream.laravel.com/2.x/installation.html)
```
composer require laravel/jetstream
```

### 使用 Livewire 安裝 Jetstream
```
php artisan jetstream:install livewire

php artisan jetstream:install livewire --teams
```

### 完成安裝
```
npm install
npm run build
php artisan migrate
```

### Livewire
```
php artisan vendor:publish --tag=jetstream-views
```

## Jetstream測試
安裝完成後輸入http://127.0.0.1 右上角會出現登入與註冊功能
### 首頁
![](https://i.imgur.com/S4KuKuy.png)
### 登入頁
![](https://i.imgur.com/P1kgDya.png)
### 註冊頁
![](https://i.imgur.com/2aILhH0.png)
### 註冊完並登入後就會看到dashboard
![](https://i.imgur.com/en8nfP5.png)

## 開啟API Token

### 修改config/jetstream.php
```
'features' => [
    // Features::termsAndPrivacyPolicy(),
    Features::profilePhotos(),
    Features::api(),
    Features::accountDeletion(),
],
```

### 定義權限
如果要修改預設權限則App\Providers\JetstreamServiceProvider類中的方法定義
```
Jetstream::defaultApiTokenPermissions(['read']);

Jetstream::permissions([
    'post:create',
    'post:read',
    'post:update',
    'post:delete',
]);
```

### 驗證權限
routes\web.php或routes\api.php中經過身份驗證的路由的請求，都將與一個 Sanctum 令牌對象相關聯。tokenCan您可以使用特徵提供的方法來確定關聯的令牌是否具有給定的權限Laravel\Sanctum\HasApiTokens。
通常可以在Controller 或 Middleware 去驗證api token。
```
use Laravel\Sanctum\Contracts\HasApiTokens;

public function index(Request $request)
{
    // 判斷是否是允許的api token
    if($request->user()->tokenCan('read')){
        return 'sucess';
    }
    return 'false';
}
```

## Jetstream Api Token 測試
### 修改 route\api.php
```
Route::group(['middleware' => ['auth:sanctum']], function() {    
    Route::get('apitest', [TestController::class, 'index']);
});
```

### 創建TestController
```
php artisan make:controller TestController --api
```

### 修改TestController
```
<?php

namespace App\Http\Controllers;

use App\Models\Test;
use Illuminate\Http\Request;
use Laravel\Sanctum\Contracts\HasApiTokens;

class TestController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index(Request $request)
    {
        // 判斷是否是允許的api token
        if($request->user()->tokenCan('read')){
            return 'sucess';
        }
        return 'false';
       
    }

    ...
}
```

### 申請API token
API token 功能開啟後，則會出現API token功能頁
![](https://i.imgur.com/fXTs5pu.png)
![](https://i.imgur.com/hWNHFB7.png)

創建名為test的API token
![](https://i.imgur.com/RDGYzFO.png)

### Running with postman
沒有帶入API token，被轉導回登入頁
![](https://i.imgur.com/J8GEi9m.png)

帶入後則會收到sucess的訊息
![](https://i.imgur.com/ZUavgzC.png)

