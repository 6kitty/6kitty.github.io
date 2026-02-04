---
layout: post
title: "[NullCon2025] Paginator"
categories: [Web Hacking]
tags: [SQL Injection, CTF]
last_modified_at: 2025-02-09
---

```php
<?php
ini_set("error_reporting", 0);
ini_set("display_errors", 0);

if(isset($_GET['source'])) {
    highlight_file(__FILE__);
}

include "flag.php";

$db = new SQLite3('/tmp/db.db');
try {
  $db->exec("CREATE TABLE pages (id INTEGER PRIMARY KEY, title TEXT UNIQUE, content TEXT)");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Flag', '" . base64_encode($FLAG) . "')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 1', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 2', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 3', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 4', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 5', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 6', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 7', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 8', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 9', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 10', 'This is not a flag, but just a boring page.')");
} catch (Exception $e) {
  //var_dump($e);
}

if(isset($_GET['p']) && str_contains($_GET['p'], ",")) {
  [$min, $max] = explode(",", $_GET['p']);
  if(intval($min) <= 1 ) {
    die("This post is not accessible...");
  }
  try {
    $q = "SELECT * FROM pages WHERE id >= $min AND id <= $max";
    $result = $db->query($q);
    while ($row = $result->fetchArray(SQLITE3_ASSOC)) {
      echo $row['title'] . " (ID=" . $row['id'] . ") has content: \"" . $row['content'] . "\"<br>";
    }
  } catch (Exception $e) {
    echo "Try harder!";
  }
} else {
    echo "Try harder!";
}
?>
<html>
    <head>
        <title>Paginator</title>
    </head>
    <body>
        <h1>Paginator</h1>
        <a href="/?p=2,10">Show me pages 2-10</a>
        <p>To view the source code, <a href="/?source">click here.</a>
    </body>
</html>
```

db 쿼리를 보내는 것을 보아 높은 확률로 sql injection 아닐까 했다.

![Image](https://blog.kakaocdn.net/dna/b6yy7f/btsL7B8bAvx/AAAAAAAAAAAAAAAAAAAAALFQKrM8QUUTrUtaBHOBTGsciBxbrE4COV04_txhO8d3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hSHUqjLgAJ5Omk1Nbr6gtVEg1cM%3D)

ID를 1도 출력할 수 있게 or을 넣었다

![Image](https://blog.kakaocdn.net/dna/ba691x/btsL8kkFXpV/AAAAAAAAAAAAAAAAAAAAAAvRxDSTIKnbTRPmy1kKYnbZbO21NPvIy3o-_lThi0b_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=futcKQADlR3c2QjNJ1od5LzrJoU%3D)