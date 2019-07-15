---
layout: single
author_profile: true
title: "umi项目更改favicon"
date: 2019-06-22 10:30:53
# toc: true
tags:
  - umijs
categories:
  - umijs
---

1. 在项目根路径下新建public目录（如果这个目录不存在的话）
2. 把favicon.ico放到public目录下(文件名修改为你的文件名)
3. 修改pages/docment.ejs, 如果没有这个文件， 请新建一个, 在head标签内部添加link标签， 示例如下
```
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>RiderPie</title>
  <link rel="icon" type="image/x-icon" href="/favicon.ico" />
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

