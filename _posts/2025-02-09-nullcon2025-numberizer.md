---
layout: post
title: "[NullCon2025] Numberizer"
categories: [SWING, Writeup]
tags: [CTF, PHP, Vulnerability]
last_modified_at: 2025-02-09
---

```php
<?php
ini_set("error_reporting", 0);

if(isset($_GET['source'])) {
    highlight_file(__FILE__);
}

include "flag.php";

$MAX_NUMS = 5;

if(isset($_POST['numbers']) && is_array($_POST['numbers'])) {

    $numbers = array();
    $sum = 0;
    for($i = 0; $i < $MAX_NUMS; $i++) {
        if(!isset($_POST['numbers'][$i]) || strlen($_POST['numbers'][$i]) > 4 || !is_numeric($_POST['numbers'][$i])) {
            continue;
        }
        $the_number = intval($_POST['numbers'][$i]);
        if($the_number < 0) {
            continue;
        }
        $numbers[] = $the_number;
    }
    $sum = intval(array_sum($numbers));

    if($sum < 0) {
        echo "You win a flag: $FLAG";
    } else {
        echo "You win nothing with number $sum ! :-(";
    }
}
?>
<html>
    <head>
        <title>Numberizer</title>
    </head>
    <body>
        <h1>Numberizer</h1>
        <form action="/" method="post">
            <label for="numbers">Give me at most 10 numbers to sum!</label><br>
            <?php
            for($i = 0; $i < $MAX_NUMS; $i++) {
                echo '<input type="text" name="numbers[]"><br>';
            }
            ?>
            <button type="submit">Submit</button>
        </form>
        <p>To view the source code, <a href="/?source">click here.</a>
    </body>
</html>
```

일단 $sum을 음수로 만들어줘야 하는데 컴퓨터에서는 int의 한계 범위가 있기 때문에 엄청 큰 양수를 만들어주면 된다. 

php에서 엄청 큰 양수를 만들기 위해 어떤 것을 할 수 있는지 조건문에 있는 함수 취약점을 찾다가.. 

E를 입력하면 지수로 인식된다는 글을 찾았다.  
[https://github.com/php/php-src/issues/9311](https://github.com/php/php-src/issues/9311)

is_numeric()은 E를 지수로 인식한다(여담이지만 소수도 numeric에 포함된다. 그래서 처음에 이걸로 푸는 줄 알았음)