---

title: Jwt-Auth实现API用户认证、刷新令牌
date: 2020-05-05 21:15:57

categories:
- laravel
- auth

tags:
- auth
- jwt

toc: true

---

### 概述：
**JWT**： JWT（JSON Web Token）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。
更详细讲解地址：https://laravelacademy.org/post/3640.html

**应用场景**：前后端分离的项目
**原理**：`http`协议是无状态的，当项目需要用户保存在线时就需要一种介质来保持
<!--more-->

---

### 使用

#### 自定义认证中间件
实现效果：提供账号密码前来登录。如果登录成功，那么我会给前端颁发一个 `access_token` ，设置在 `header` 中以请求需要用户认证的路由。
如果用户的令牌如果过期了，可以暂时通过此次请求，并在此次请求中刷新该用户的 `access_token`，最后在响应头中将新的 `access_token` 返回给前端，这样子可以无痛的刷新 `access_token`，用户可以获得一个很良好的体验，所以开始动手写代码。

执行如下命令以新建一个中间件：
```php
php artisan make:middleware RefreshToken
```

中间件app\Http\Middleware\RefreshToken.php代码如下：

```php
<?php

namespace App\Http\Middleware;

use Auth;
use Closure;
use Tymon\JWTAuth\Exceptions\JWTException;
use Tymon\JWTAuth\Http\Middleware\BaseMiddleware;
use Tymon\JWTAuth\Exceptions\TokenExpiredException;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;

// 注意，我们要继承的是 jwt 的 BaseMiddleware
class RefreshToken extends BaseMiddleware
{
    /**
     * @param $request
     * @param Closure $next
     * @return \Illuminate\Http\JsonResponse|\Illuminate\Http\Response|mixed
     * @throws JWTException
     */
    public function handle($request, Closure $next)
    {
        // 检查此次请求中是否带有 token，如果没有则抛出异常。
        $this->checkForToken($request);

        // 使用 try 包裹，以捕捉 token 过期所抛出的 TokenExpiredException  异常
        try {
            // 检测用户的登录状态，如果正常则通过
            if ($this->auth->parseToken()->authenticate()) {
                return $next($request);
            }
            throw new UnauthorizedHttpException('jwt-auth', '未登录');
        } catch (TokenExpiredException $exception) {
            // 此处捕获到了 token 过期所抛出的 TokenExpiredException 异常，我们在这里需要做的是刷新该用户的 token 并将它添加到响应头中
            try {
                // 刷新用户的 token
                $token = $this->auth->refresh();
                // 使用一次性登录以保证此次请求的成功
                Auth::guard('api')->onceUsingId($this->auth->manager()->getPayloadFactory()->buildClaimsCollection()->toPlainArray()['sub']);
            } catch (JWTException $exception) {
                // 如果捕获到此异常，即代表 refresh 也过期了，用户无法刷新令牌，需要重新登录。
                throw new UnauthorizedHttpException('jwt-auth', $exception->getMessage());
            }
        }

        // 在响应头中返回新的 token
        return $this->setAuthenticationHeader($next($request), $token);
    }
}

```

然后我们注册一下中间件，文件位置app\Http\Kernel.php在路由处会使用
```php
protected $routeMiddleware = [
        'refresh' => \App\Http\Middleware\RefreshToken::class,
]

```

更新异常处理的 Handler

由于我们构建的是 api 服务，所以我们需要更新一下 `app/Exceptions/Handler.php` 中的 `render`方法，自定义处理一些异常。

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;

class Handler extends ExceptionHandler
{
   ...

   /**
    * Render an exception into an HTTP response.
    *
    * @param  \Illuminate\Http\Request $request
    * @param  \Exception $exception
    * @return \Illuminate\Http\Response
    */
   public function render($request, Exception $exception)
   {
       // 参数验证错误的异常，我们需要返回 400 的 http code 和一句错误信息
       if ($exception instanceof ValidationException) {
           return response(['error' => array_first(array_collapse($exception->errors()))], 400);
       }
       // 用户认证的异常，我们需要返回 401 的 http code 和错误信息
       if ($exception instanceof UnauthorizedHttpException) {
           return response($exception->getMessage(), 401);
       }

       return parent::render($request, $exception);
   }
}

```

