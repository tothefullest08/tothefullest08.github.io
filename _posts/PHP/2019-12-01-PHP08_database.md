---
layout: post
title: 생활 코딩 PHP 08 - Database
category: PHP
tags: [생활코딩, PHP]
comments: true
---





## 1. DATABASE

#### 1.1 GET STARTED

> 일반적으로 데이터를 저장해야 하는 에플리케이션을 구축하는 경우 데이터는 데이터베이스 서버에 저장한다. PHP 에플리케이션의 코드에는 데이터베이스 서버에 접속해서 데이터를 조회/수정/삭제하는 코드를 작성한다. 
>
> PHP 에플리케이션이 호출되면 PHP 엔진은 PHP 에플리케이션의 코드에 따라서 데이터베이스의 클라이언트가 되어서 데이터베이스에 접속해서 여러가지 작업을 처리한다.
>
> <https://opentutorials.org/course/62/5175>



- C:\Bitnami\wampstack-7.3.11-0\mysql\mysql.exe로 MySQL Cllient인 MySQL Monitor 실행 가능(cmd로 실행)
- 해당 디렉토리가 아니더라도 편하게 mysql을 실행할 수 있도록 환경변수 추가(bin까지의 경로)
- mysql -u root -p111111 -hlocalhost
  - `mysql` : mysql.exe를 실행
  - `-u` , `-p` , `-h` : 각각 유저, 패스워드, 호스트를 의미 

```bash
C:\Bitnami\wampstack-7.3.11-0\mysql\bin>mysql -uroot -p111111 -hlocalhost
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.18 MySQL Community Server - GPL
```



#### 1.2 Database, Table 제어

- `CREATE DATABASE DB 이름` : 데이터 베이스 생성
- `show databases` : 데이터 베이스 리스트 불러오기
- `use DB명` : 해당 DB로 이동
- `CREATE TABLE` : 스키마 생성.
- `show tables`  : 테이블 리스트 불러옴.
- `desc 테이블명` :  테이블의 스키마 불러옴. 

```mysql
CREATE DATABASE opentutorials CHARACTER SET utf8 COLLATE utf8_general_ci;
show databases;
use opentutorials;

CREATE TABLE topic (
 id int(11) NOT NULL AUTO_INCREMENT,
 title varchar(255) NOT NULL ,
 description text NULL ,
 created datetime NOT NULL ,
 PRIMARY KEY (id)
);

show tables;
desc topic;
```



#### 1.3 Database CRUD

- `insert` , `update` , `delete` , `select` 

```mysql
#DB 삽입
INSERT INTO topic (title, description, created) VALUES('htiml', 'html이란 무엇인가?', now()); 
INSERT INTO topic (title, description, created) VALUES('css', 'html 꾸며주는 언어', now());

#DB 조회
SELECT * FROM topic; #all
SELECT id, title, created FROM topic;
SELECT * FROM topic WHERE id=2;
SELECT * FROM topic where id=1 OR id=2;
SELECT * FROM topic ORDER BY id DESC;

#DB 수정
UPDATE topic SET title='case cading style sheet', description='아름다운 언어' WHERE id=2;

#DB 삭제
DELETE FROM topic where id=2;
```



## 2. PHP와 MySQL 확장

- PHP의 MySQL 확장 기능은 PHP 5.5부터 폐지되었음. 

- 폼태그로 내용을 작성한 후, process.php로 POST 방식으로 작성된 데이터를 전송

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8"/>
    </head>   
    <body>
        <form action="./process.php?mode=insert" method="POST">
            <p>제목 : <input type="text" name="title"></p>
            <p>본문 : <textarea name="description" id="" cols="30" rows="10">
                </textarea></p>
            <p><input type="submit" /></p>            
        </form>
    </body>
</html>
```

- process.php

```php
<?php
mysql_connect('localhost', 'root', '111111'); #인자: host, username, password
mysql_select_db('opentutorials'); # use open tutoriarls

#switch는 조건문. switch()값이 case의 값과 일치할 경우 아래 코드 실행. (반드시 break 걸어줄 것)
switch($_GET['mode']){
    case 'insert':
    $result = mysql_query("INSERT INTO topic (title, description, created) VALUES ('".mysql_real_escape_string($_POST['title'])."', '".mysql_real_escape_string($_POST['description'])."', now())");
    header("Location: list.php"); #redirect
    break;
    case 'delete':
    mysql_query('DELETE FROM topic WHERE id = '.mysql_real_escape_string($_POST['id']));
    header("Location: list.php"); 
    break;
    case 'modify':
    mysql_query('UPDATE topic SET title = "'.mysql_real_escape_string($_POST['title']).'", description = "'.mysql_real_escape_string($_POST['description']).'" WHERE id = '.mysql_real_escape_string($_POST['id']));
    header("Location: list.php?id={$_POST['id']}");
    break;
   }

#switch 조건문을 if로 대체했을 경우 아래와 같은 형태로 변경됨,
if($_GET['mode'] === 'insert'){
    $result = mysql_query("INSERT INTO topic (title, description, created) VALUES ('".mysql_real_escape_string($_POST['title'])."', '".mysql_real_escape_string($_POST['description'])."', now())");
    header("Location: list.php"); 
} else if ($_GET['mode'] === 'delete'){
    mysql_query('DELETE FROM topic WHERE id = '.mysql_real_escape_string($_POST['id']));
    header("Location: list.php"); 
} else if ($_GET['mode'] === 'modify'){
    mysql_query('UPDATE topic SET title = "'.mysql_real_escape_string($_POST['title']).'", description = "'.mysql_real_escape_string($_POST['description']).'" WHERE id = '.mysql_real_escape_string($_POST['id']));
    header("Location: list.php?id={$_POST['id']}");
}
?>
```



## 3. PHP와 PDO

> PDO(PHP Data Objects)란 여러가지 데이터베이스를 제어하는 방법을 표준화시킨 것이다. 데이터베이스는 다양한 종류가 있다. 그리고 종류에 따라서 서로 다른 드라이브를 사용해 왔는데 드라이브의 종류에 따라서 데이터베이스를 제어하기 위한 API가 달랐다. PDO를 사용하면 동일한 방법으로 데이터베이스를 제어할 수 있다.
>
> <https://opentutorials.org/course/62/5155>



#### 3-1. process.php

```php
<?php
#pdo 객체 생성 & DB 접속. host, username, password 순으로 PDO 인자 입력. 
$dbh = new PDO('mysql:host=localhost;dbname=opentutorials', 'root', '111111', array(PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8"));

switch($_GET['mode']){
    case 'insert': #데이터 추가

        #쿼리를 담은 PDO stmt(statement) 객체 생성
        #PDO를 사용하면 준비구문(prepare statements)를 활용할 수 있음
        #준비 구문 => SQL 인젝션 공격 방어 가능, 어플 성능 향상 
        $stmt = $dbh->prepare("INSERT INTO topic (title, description, created) VALUES (:title, :description, now())");
        
        # PDO stmt 객체가 가진 쿼리 파라미터에 변수를 바인딩
        $stmt->bindParam(':title',$title);
        $stmt->bindParam(':description',$description); 
        $title = $_POST['title'];
        $description = $_POST['description'];
        
        $stmt->execute(); # PDO stmt 객체가 가진 쿼리 실행
        header("Location: list.php"); # Redirect
        break;
    
    case 'delete': #데이터 삭제
        $stmt = $dbh->prepare('DELETE FROM topic WHERE id = :id');
        $stmt->bindParam(':id', $id);
 
        $id = $_POST['id'];
        $stmt->execute();
        header("Location: list.php"); 
        break;
    
    case 'modify': #데이터 수정
        $stmt = $dbh->prepare('UPDATE topic SET title = :title, description = :description WHERE id = :id');
        $stmt->bindParam(':title', $title);
        $stmt->bindParam(':description', $description);
        $stmt->bindParam(':id', $id);
 
        $title = $_POST['title'];
        $description = $_POST['description'];
        $id = $_POST['id'];
        $stmt->execute();
        header("Location: list.php?id={$_POST['id']}");
        break;
}
?>
```



#### 3-2. list.php

- `<?= ?>`는 `<?php echo ?>`의 shorthand  

```php+HTML
<?php
$dbh = new PDO('mysql:host=localhost;dbname=opentutorials', 'root', '111111');
$stmt = $dbh->prepare('SELECT * FROM topic');
$stmt-> execute();
$list = $stmt -> fetchAll(); #한번에 모든 행을 가져올때 사용하는 메서드
if(!empty($_GET['id'])) {
  $stmt = $dbh -> prepare('SELECT * FROM topic WHERE id=:id');
  $stmt->bindParam(':id', $id, PDO::PARAM_INT); #PARAM_INT: SQL INT 데이터 타입을 표현
  $id = $_GET['id'];
  $stmt -> execute();
  $topic = $stmt->fetch();
}
?>
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <style type="text/css">
      body {
          font-size: 0.8em;
          font-family: dotum;
          line-height: 1.6em;
      }
      header {
          border-bottom: 1px solid #ccc;
          padding: 20px 0;
      }
      nav {
          float: left;
          margin-right: 20px;
          min-height: 1000px;
          min-width:150px;
          border-right: 1px solid #ccc;
      }
      nav ul {
          list-style: none;
          padding-left: 0;
          padding-right: 20px;
      }
      article {
          float: left;
      }
      .description{
          width:500px;
      }
    </style>
  </head>   
  <body id="body">
    <div>
      <nav>
        <ul>
         <?php
          foreach($list as $row) {
            echo "<li><a href=\"?id={$row['id']}\">".htmlspecialchars($row['title'])."</a></li>";
          }
         ?>
        </ul>
        <ul>
          <li><a href="input.php">추가</a></li>
        </ul>
      </nav>
      <article>
      <?php
      if(!empty($topic)){
      ?>
      <h2><?=htmlspecialchars($topic['title'])?></h2> 
      <div class="description">
        <?=htmlspecialchars($topic['description'])?>
      </div>
      <div>
        <a href="modify.php?id=<?=$topic['id']?>">수정</a>
        <form method="POST" action="process.php?mode=delete">
          <input type="hidden" name="id" value="<?=$topic['id']?>" />
          <input type="submit" value="삭제" />
        </form>
      </div>
      <?php
      }
      ?>
      </article>
    </div>
  </body>
</html>
```



#### 3-3. modify.php

```php+HTML
<?php
$dbh = new PDO('mysql:host=localhost;dbname=opentutorials', 'root', '111111');
$stmt = $dbh->prepare('SELECT * FROM topic WHERE id = :id');
$stmt->bindParam(':id', $id, PDO::PARAM_INT);
$id = $_GET['id'];
$stmt->execute();
$topic = $stmt->fetch();
?>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8"/>
    </head>   
    <body>
        <form action="./process.php?mode=modify" method="POST">
            <input type="hidden" name="id" value="<?=$topic['id']?>" />
            <p>제목 : <input type="text" name="title" value="<?=htmlspecialchars($topic['title'])?>"></p>
            <p>본문 : <textarea name="description" id="" cols="30" rows="10"><?=htmlspecialchars($topic['description'])?></textarea></p>
            <p><input type="submit" /></p>
        </form>
    </body>
</html>
```