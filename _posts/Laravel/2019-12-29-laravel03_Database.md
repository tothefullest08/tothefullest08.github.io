---
layout: post
title: Laravel 5.7 From Scratch 03 - Databases / Migrations
category: PHP
tags: [Laracast, Laravel]
comments: true
---



#### 1. GET STARTED

`laravel new project`  : project라는 새로운 라라벨 프로젝트 생성
`.env` : configuration & private detail을 나타냄. (private api key, DB password 등) 

Sequel_pro 설치 후 DB 연결, tutorial이라는 DB 생성 후 그에 맞게 `.env` 파일 수정. 각 변수를 `.env`에 저장하였지만, 어떻게 이 변수들이 어플리케이션에서 참조가 가능할까? => config/database.php 확인

```php
// env/DB_CONNECTION을 우선적으로 읽음. 실패할 경우 두번째 인자를 읽음.
'default' => env('DB_CONNECTION', 'mysql'),
```



#### 2. Migrations

`php artisn migrate` : 데이터베이스에 마이그레이션 적용
`php artisan migrate:rollback` : 마이그레이션 롤백

```php
mysql> show tables;
+--------------------+
| Tables_in_tutorial |
+--------------------+
| failed_jobs        |
| migrations         |
| password_resets    |
| users              |
+--------------------+
4 rows in set (0.00 sec)

mysql> describe users;
+-------------------+---------------------+------+-----+---------+----------------+
| Field             | Type                | Null | Key | Default | Extra          |
+-------------------+---------------------+------+-----+---------+----------------+
| id                | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
| name              | varchar(255)        | NO   |     | NULL    |                |
| email             | varchar(255)        | NO   | UNI | NULL    |                |
| email_verified_at | timestamp           | YES  |     | NULL    |                |
| password          | varchar(255)        | NO   |     | NULL    |                |
| remember_token    | varchar(100)        | YES  |     | NULL    |                |
| created_at        | timestamp           | YES  |     | NULL    |                |
| updated_at        | timestamp           | YES  |     | NULL    |                |
+-------------------+---------------------+------+-----+---------+----------------+
8 rows in set (0.00 sec)
```

#### 

#### 3. Database 컬럼명 변경

users 테이블의 name을 변경하고 싶다면 config/migrations 디렉토리 내에 저장된 `2014_10_12_000000_create_users_table.php` 파일에서 내용을 변경하면됨.

이렇게 변경된 컬럼명을 적용시키는데는 크게 2가지 방법이 있을 수 있음.

1. 마이그레이션 롤백 후 재 적용
2. `php artisan migrate:fresh`

```php
class CreateUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->bigIncrements('id');
						$table->string('username');            
          	//$table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
```



#### 4. Database 테이블 생성

`php artisan make:migration` : 새로운 마이그레이션 파일 생성 => `php artisan make:migration create_projects_table`

- `up()` : 마이그레이션 시 사용
- `down()` : 마이그레이션 롤백 시 사용. 함수 코드를 주석 처리할 경우, `migration:rollback`을 하더라도 마이그레이션 롤백이 적용되지 않음. 이런 경우 `migration:fresh` 를 사용함 (마이그레이션을 완전 새로고치므로...)

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProjectsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('projects', function (Blueprint $table) {
            $table->bigIncrements('id');
          	$table->string('title');
            $table->text('description');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('projects');
    }
}
```
