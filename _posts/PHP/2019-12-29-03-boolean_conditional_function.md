---
layout: post
title: The PHP Practitioner 03 - Boolean / Conditional / Function
category: PHP
tags: [Laracast, PHP]
comments: true
---



#### 1. Boolean & Conditional

Boolean은 조건문을 통해 여러가지 방법으로 표현이 가능함

- 삼항 연산자: `<?= $task['completed'] ? 'Complete' : 'Incomplete'; ?>`
- `If/else if/else`
- Not: `!true`

```php
<?php

$task = [
    'title' => 'Finish homework',
    'due' => 'today',
    'assigned_to' => 'jeffery',
    'completed' => false

]; // title, due, assigned_to,

require 'index.view.php';
```

```php+HTML
</<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <h1>Task For The Day</h1>
    <ul>
        <?php foreach ($task as $heading => $value) :?>
        <li>
            <!-- ucwords: Capitalize first letter of each word -->
            <Strong><?= ucwords($heading); ?>: </Strong> <?= $value; ?>
        </li>
        <?php endforeach; ?>
    </ul>
    <ul>
        <li>
            <strong> Name: </strong> <?= $task['title']; ?>
        </li>
        <li>
            <strong> Due Date: </strong> <?= $task['due']; ?>
        </li>
        <li>
            <strong> Person Responsible: </strong> <?= $task['assigned_to']; ?>
        </li>
        <!-- Boolean 표현 1. 삼항 연산자 -->
        <li>
            <strong> Status: </strong> <?= $task['completed'] ? 'Complete' : 'Incomplete'; ?>
        </li>
        <!-- Boolean 표현 2. 일반 조건문(if/else) -->
        <li>
            <strong> Status: </strong>
            <?php
            if ($task['completed']) {
                echo 'Complete &#9989;';
            } else {
                echo 'Incomplete ';
            }
            ?>

        </li>
        <!-- 2-1. -->
        <li>
            <strong> Status: </strong>
            <?php if ($task['completed']) : ?>
                <span class="icon">&#9989;</span>
            <?php endif; ?>
        </li>
        <!-- 2.2. not 표현 -->
        <li>
            <strong> Status: </strong>
            <?php
                if (! $task['completed']){
                    echo 'Incomplete';
                }
            ?>
        </li>
    </ul>
</body>
</html>
```



#### 2. Function

```php
<?php

require 'function.php';
require 'index.view.php';

$animals = ['dog', 'cat'];

dd($animals);
// dd(['dog', 'cat']);

<?php

function dd($data) {
    echo '<pre>';
    die(var_dump($data));
    echo '</pre>';
}
```
