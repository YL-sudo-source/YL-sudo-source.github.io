---
title: "hugo&PaperMod一些修改"
date: "2024-11-20"
lastmod: "2024-11-24"
tags: 
    - 教程
    - PaperMod
---

# 给文章添加最后更新时间
在 PaperMod 的 中加上三行：post_meta.html
~~~
{{- if gt .Lastmod .Date -}}
{{- $scratch.Add "meta" (slice (printf "<span title='%s'>(updated: %s)</span>" (.Lastmod) (.Lastmod | time.Format (default "January 2, 2006" .Site.Params.DateFormat)))) }}
{{- end }}
~~~

在 Hugo 的 Front Matter 下手动加上 属性即可：lastmod

~~~
title: 折腾 Hugo & PaperMod 主题
slug: hugo-papermod-config
author: Dvel
date: 2022-01-11T15:15:38+08:00
lastmod: 2023-02-14T00:40:08+08:00
~~~

