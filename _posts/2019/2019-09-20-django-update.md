---
layout: single
author_profile: true
title: "django update() auto_now 不更新问题"
date: 2019-09-20 10:30:53
toc: true
tags:
  - python
  - django
categories:
  - python
---

### 概述
django update() auto_now 不更新问题

在django的model层给字段添加auto_now之后,再使用save()方法更新数据库时会自动更新当时的时间,

如果用django filter的update则是因为直接调用sql语句 不通过 model层, 所以不会自动更新带有auto_now的字段,

官方对此的解释为

```
What you consider a bug, others may consider a feature, e.g. usingupdate_fieldsto bypass updating fields withauto_now. In fact, I wouldn't expect auto_now fields to be updated if not present inupdate_fields.
```

这也是很有必要的, 有时我们只想对用户直接进行的操作记录更新的时间, 对系统自动的刚新则不予更新时间.

这时我们可以显示的更新该字段 比如 update(xxx=yyy, update_time=datetime.datime.now())
