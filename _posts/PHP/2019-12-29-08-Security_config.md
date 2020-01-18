---
layout: post
title: The PHP Practitioner 08 - Security and Configuration
category: PHP
tags: [Laracast, PHP]
comments: true
---

> Laracasts - The PHP Practitioner [강의](https://laracasts.com/series/php-for-beginners)를 듣고 정리한 포스팅 입니다.

#### 1. Config.php

`database/Connection.php` 에는 현재 DSN, 아이디, 비밀번호가 전부 노출되어있음. 보안상의 문제가 발생할 수 있으므로, 이러한 데이터들을 따로 일괄적으로 관리하는 것이 더 적합함. => `config.php` 생성

- `options` 은 PDO 객체를 생성할 때 지정할 수 있는 옵션으로, 에러가 발생 시, 어떠한 에러가 발생하는지를 알기 위해서는 `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION` 를 사용

```php
<?php

return [
    'database' => [
        'name' => 'mytodo',
        'username' => 'root',
        'password' => '111111',
        'connection' => 'mysql:host=localhost',
        'options' => [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
        ]
    ]
];
```



#### 2. config.php 호출

`bootrap.php` 에서 `config.php` 를 호출한 후, 데이터를 `Connection::make` 의 인자로 넘겨준다.

```php
<?php

$config = require 'config.php';
require 'database/Connection.php';
require 'database/QueryBuilder.php';


return new QueryBuilder(
    Connection::make($config['database'])
);
```



`Connection.php` 에서 PDO 객체의 파라미터로,  넘어온 연관배열의 값을 입력해준다.

```php
<?php

class Connection
{
    public static function make($config)
    {
        try {
            return new PDO(
                $config['connection'].';dbname='.$config['name'],
                $config['username'],
                $config['password'],
                $config['options']
            );
        } catch (PDOException $e) {
            die($e->getMessage());
        }
    }
}
```

