---
title: Markdown简要规则
date: 2026-03-01 16:10:00
tags: Markdown
categories:
  - [笔记工具, Markdown]
---


# 一级标题 
\# 一级标题
## 二级标题 
\## 二级标题
### 三级标题 
\### 三级标题
#### 四级标题 
\#### 四级标题
##### 五级标题 
\##### 五级标题
###### 六级标题 
\###### 六级标题

---

*强调文本* _强调文本_ 
\*强调文本\* \_强调文本\_

**加粗文本** __加粗文本__ 
\*\*加粗文本\*\* \__加粗文本\__

==标记文本== 
\==标记文本\==

~~删除文本~~ 
\~~删除文本\~~

> 引用文本 

\> 引用文本

---

- 项目 `- 项目`
  * 项目 `* 项目`
    + 项目 `+ 项目`

1. 项目1 `1. 项目1`
2. 项目2 `2. 项目2`
3. 项目3 `3. 项目3`

- [ ] 计划任务 

\- [ ] 计划任务

- [x] 完成任务 

\- [x] 完成任务

---

链接: [百度link](www.baidu.com) 
`[百度link](www.baidu.com)`

---

内联代码片

```
// A code block
var foo = 'bar';
```


\``` 
// A code block
var foo = 'bar';
\```

```javascript
// An highlighted block
var foo = 'bar';
```
\```javascript
// An highlighted block
var foo = 'bar';
\```

---

项目     | Value
-------- | -----
电脑  | $1600
手机  | $12
导管  | $1

项目     \| Value
-------- \| -----
电脑  \| $1600
手机  \| $12
导管  \| $1

---

公式：

Gamma公式展示 $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$ 是通过 Euler integral

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

Gamma公式展示 \$\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N\$ 是通过 Euler integral

\$\$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
\$\$

---