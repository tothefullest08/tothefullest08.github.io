---
layout: post
title: Udemy Laravel 01 - Get Started 
category: Laravel
tags: [PHP, Laravel]
comments: true
---

## 1. Get Started

#### 1.1 XAMPP 설치

[XAMPP 설치](<https://www.apachefriends.org/index.html>) & XAMPP Control Panel 실행. 

- Apache - Config - Apach (httpd.config) 설정으로 들어가 Post 넘버 변경(Listen 3000 & ServerName localhost:3000)
- 마찬가지로 Apach (httpd-ssl.config) 설정으로 들어가 변경(Listen 1443,  \<VirtualHost \_default_:1443>, ServerName www.example.com:1443)
- Config로 들어가 service settings에 Port 넘버 최종 변경(3000, 1443)

Apache 뿐만 아니라 MySQL 등에서 포트넘버 충돌로 에러가 발생한다면 Unique한 포트넘버로 수정하도록 하자.

<http://localhost:3000/> 으로 접속하여 XAMPP 확인



#### 1.2 composer

policy(Dependency) maanger for PHP. 패키지간의 의존성을 관리도구이다. 필요한 확장 기능을 쉽게 설치해주는 기능도 제공하지만, 프로젝트에서 필요한 확장 기능을 통합해서 관리해주는 도구(`npm`, `pip `같은 친구)

Download로 들어가 composer를 설치해주도록 하자. 설치 이후, cmd에서 `composer`을 입력하면 composer에 대한 설명을 볼 수 있음.

- <https://getcomposer.org/>
- <https://packagist.org/>



#### 1.3 Laravel Project 생성

- `/c/xampp/htdocs` 으로 이동
- `composer create-project laravel/laravel 프로젝트명 라라벨버젼 `
- 에러 발생시 `composer install --no-scripts`  > `composer install`

브라우저에서 url 주소의 끝에 `.dev`로 입력했을 경우 에러가 발생하다면 `.test`로 변경해서 입력할 것



#### 1.4 Virtual Hosts

`<http://localhost:3000/cms/>`  으로 입력하면 라라벨 기본 프로젝트를 확인할 수 있음. 편의성을 위해 url주소를 변경할 수 있음. modifier이나 virtual host를 아파치 서버에 설치해야함.

- C:\xampp\apache\conf\extra\httpd-vhosts.conf 실행. 아래와 같이 virtual Host를 설정할 수 있음.

```bash
<VirtualHost *:3000>
	DocumentRoot "C:/xampp/htdocs"
	ServerName localhost
</VirtualHost>

<VirtualHost *:3000>
	DocumentRoot "C:/xampp/htdocs/cms"
	ServerName cms.test
</VirtualHost>
```

- C:\Windows\System32\drivers\etc\hosts을 관리자 권한으로 실행후 아래 코드 삽입

```bash
127.0.0.1       localhost
127.0.0.1       cms.test
```

- `<http://cms.test:3000/>` 으로 접속 가능
