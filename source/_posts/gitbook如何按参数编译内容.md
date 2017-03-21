---
title: gitbook如何按参数编译内容
date: 2017-03-21 17:18:04
tags:
---

有时候在编写gitbook的时候，一个版本的内容需要按照编译参数给出不同的结果。比如内部版和外部版就会有一些差异，这个时候如何控制编译结果呢？
这里引入gitbook的一个参数功能。
1. 在`book.json`中定义对应的控制参数
```json
{
    "variables": {
        "version": "out"
    }
}
```

2. 在需要控制的内容块上使用条件判断
```
{% if book.version == "out" %}
 
{% else %}
 
{% endif %}
```

