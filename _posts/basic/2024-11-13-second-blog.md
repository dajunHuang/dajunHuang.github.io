---
layout: posts
title: 中文 second blog
description: 中文 Today I needed to quickly parse a pipe delimited data file in order to get a unique list of values from one of the columns.
date: 2024-11-13 20:40:00 +0800
category:
- basic
tags:
- Quick-Post
- test_tag
permalink: /basic/second-blog
published: false
image: /assets/pic/notes/2024-11-23-dan-wo-tao-yan/image1.png
---

## 二级标题

### 三级标题

中文

Due to a plugin called `jekyll-titles-from-headings` which is supported by GitHub Pages by default. The above header (in the markdown file) will be automatically used as the pages title.

If the file does not start with a header, then the post title will be derived from the filename.

This is a sample blog post. You can talk about all sorts of fun things here.

---

### This is a header

#### Some T-SQL Code

```tsql
SELECT This, [Is], A, Code, Block -- Using SSMS style syntax highlighting
    , REVERSE('abc')
FROM dbo.SomeTable s
    CROSS JOIN dbo.OtherTable o;
```

#### Some PowerShell Code

```powershell
Write-Host "This is a powershell Code block";

# There are many other languages you can use, but the style has to be loaded second

ForEach ($thing in $things) {
    Write-Output "It highlights it using the GitHub style"
}
```