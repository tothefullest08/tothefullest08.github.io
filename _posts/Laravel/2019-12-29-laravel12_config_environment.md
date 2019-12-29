---
layout: post
title: Laravel 5.7 From Scratch 12 - Configuration / Environment
category: PHP
tags: [Laracast, Laravel]
comments: true
---



#### 1. GET STARTED

환경설정은 기본적으로 `.env` 파일에서 이루어짐. 그리고 config 디렉토리 내에 있는 파일들이 개별적으로 `.env` 파일의 내용을 호출하며 환경설정이 이루어짐. 

`.env` 파일을 보안상의 목적으로 외부에 공개되면 안되므로 `.gitignore`에 추가하여 관리함. 따라서, development & production server의  `.env` 파일의 내용은 다름.

```php
# development
APP_ENV=local
APP_DEBUG=true
SESSION_DRIVER=file
  
# production
APP_DEBUG=false
SESSION_DRIVER=reddis
```



#### 2. exercise - call private key

설정된 private key를 호출하는 예시를 보도록하자.

Config\services.php 내에 key, value값 생성후 app\Providers\SocialServiceProvider.php에서 호출

```php
// Config\services.php

return [
  
  // ... skip
  
'twitter' => [
  'key' => 'public-key',
  'secret' => 'secret-key'
],
  
// app\Providers\SocialServiceProvider.php
<?php

namespace App\Providers;

use App\Services\Twitter;
use Illuminate\Support\ServiceProvider;

class SocialServiceProvider extends ServiceProvider
{

    public function register()
    {
        $this->app->singleton(Twitter::class, function() {
            return new Twitter(config('services.twitter.secret'));
        });
    }
} 
```



이후 web.php에서 서비스 컨테이너를 제대로 호출하는지 확인

```php
<?php

use App\Services\Twitter;

Route::get('/', function(Twitter $twitter) {
    dd($twitter);

    return view('welcome');
});
```



#### 3. Expand 

Config\services.php 에서 고유 키값을 입력하는 것 보다, 고유값은 `.env`에서 입력한 후, 그 값을 불러오는 것이 더 효율적임

```php
// Config\services.php
return [
    'twitter' => [
        'key' => env('TWITTER_KEY'),
        'secret' => env('TWITTER_SECRET')
    ],
],

//.env
TWITTER_KEY="public-key"
TWITTER_SECRET="secret-key"
```



새로운 configuration 파일을 만든 것도 가능함(in config directory)

```php
<?php

return [
    'stripe' => [
        'private' => ''
    ]
];

// config('laracasts.strip.private')
```



Production 환경에서는 config 디렉토리 내 파일이 매우 많으므로 한 파일로 병합하는 것이 소스 코드를 불러 올때 빠름. development 에서는 사용할 필요 없음. this is for the purpose of performance optimization.

-  `php artisan config:cache`  - Create a cache file for faster configuration loading
-  `php artisan config:clear` - clear a chahe file for faster configuration loading

