# Laravel9 實作API JWT 機制

[![hackmd-github-sync-badge](https://hackmd.io/sfgVzCfGTzukHtnthP7pWg/badge)](https://hackmd.io/sfgVzCfGTzukHtnthP7pWg)

###### tags: `Laravel` `PHP` `JWT`
## 安裝
### 安裝jwt-auth
[套件官網](https://jwt-auth.readthedocs.io/en/develop/laravel-installation/)
```
composer require tymon/jwt-auth --ignore-platform-reqs
```

### 發行套件設定檔
```
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```

### 產生JWT憑證
```
php artisan jwt:secret
```

### 修改User model
User model 引入 JWTSubject，宣告User model為JWTSubject介面，並加入下列2個方法：
```
<?php

namespace App\Entities;

use Tymon\JWTAuth\Contracts\JWTSubject;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements JWTSubject
{
    use Notifiable;
    
    ... (略)
        
        
    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

### 修改 config\auth.php
```
'defaults' => [
    'guard' => 'api',
    'passwords' => 'users',
],

...

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

### 創建路由 routes\api.php
```
Route::group(['prefix' => 'auth'], function () {
    Route::get('/', [AuthController::class, 'me']);
    Route::post('login', [AuthController::class, 'login']);
    Route::post('logout', [AuthController::class, 'logout']);
    Route::post('refresh', [AuthController::class, 'refresh']);
});
```

### 創建AuthController
```
php artisan make:controller AuthController
```
#### app\Http\Controllers\AuthController.php
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Http\Controllers\Controller;

class AuthController extends Controller
{
    /**
     * Create a new AuthController instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth:api', ['except' => ['login']]);
    }

    /**
     * Get a JWT via given credentials.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function login()
    {
        $credentials = request(['email', 'password']);

        if (! $token = auth()->attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    /**
     * Get the authenticated User.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function me()
    {
        return response()->json(auth()->user());
    }

    /**
     * Log the user out (Invalidate the token).
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function logout()
    {
        auth()->logout();

        return response()->json(['message' => 'Successfully logged out']);
    }

    /**
     * Refresh a token.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function refresh()
    {
        return $this->respondWithToken(auth()->refresh());
    }

    /**
     * Get the token array structure.
     *
     * @param  string $token
     *
     * @return \Illuminate\Http\JsonResponse
     */
    protected function respondWithToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
}

```

## 測試
### API參數對照表


| URI      | description  | HTTP Method | Header |
| -------- | -------- | -------- | -------- |
| api/auth | 取得user資訊 | GET | Authorization: bearer Token <br> Accept: application/json |
| api/auth/login | 登入 | POST |  |
| api/auth/logout | 登出 | POST | Authorization: bearer Token <br> Accept: application/json |
| api/auth/refresh | 刷新user token | POST | Authorization: bearer Token <br> Accept: application/json |

### Running with postman
打開postman後，使用「取得user資訊」的API，若不夾帶token、token失效或者使用非法token，就會收到如下圖的回應，且HTTP Status Code 為401
![](https://i.imgur.com/OeZRMcm.png)

為何我們會收到401的狀態碼呢？因為這支 API 被宣告使用「auth:api」的中介層，倘若你身分是不合法的，就會被中介層拒於門外。這時我們就透過「登入」，取得合法的Token，並再次取得user資訊，結果就會如下
![](https://i.imgur.com/gQfscSe.png)

就這樣，利用 JWT 進行 API 身分驗證的機制就完成了!