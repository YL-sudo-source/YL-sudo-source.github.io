---
title: "hugo&PaperMod一些修改"
date: "2024-11-20"
lastmod: "2024-11-24"
tags: 
    - 教程
    - PaperMod
    - 活动
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

# 评论功能
代码来源[dvel](https://dvel.me/posts/hugo-papermod-config/)
修改comments.html
~~~
{{- /* Comments area start */ -}}
{{- /* to add comments read => https://gohugo.io/content-management/comments/ */ -}}

<link rel="stylesheet" href="https://unpkg.com/@waline/client@v2/dist/waline.css" />

<div id="waline"></div>
<script type="module">
    import { init } from 'https://unpkg.com/@waline/client@v2/dist/waline.mjs';
    const locale = {
        nick: '昵称',
        nickError: '请填写昵称',
        mail: '邮箱',
        mailError: '请填写正确的邮件地址',
        link: '网址',
        optional: '可选',
        placeholder: '仅填写昵称即可发表回复。\n填写邮箱可收到回复提醒。\n评论区支持 Markdown 语法及预览。\n',
        sofa: '来发评论吧~',
        submit: '提交',
        like: '喜欢',
        cancelLike: '取消喜欢',
        reply: '回复',
        cancelReply: '取消回复',
        comment: '评论',
        refresh: '刷新',
        more: '加载更多...',
        preview: '预览',
        emoji: '表情',
        uploadImage: '上传图片',
        seconds: '秒前',
        minutes: '分钟前',
        hours: '小时前',
        days: '天前',
        now: '刚刚',
        uploading: '正在上传',
        login: '管理',
        logout: '退出',
        admin: '博主',
        sticky: '置顶',
        word: '字',
        wordHint: '评论字数应在 $0 到 $1 字之间！\n当前字数：$2',
        anonymous: '匿名',
        level0: '潜水',
        level1: '冒泡',
        level2: '吐槽',
        level3: '活跃',
        level4: '话痨',
        level5: '传说',
        gif: '表情包',
        gifSearchPlaceholder: '搜索表情包',
        profile: '个人资料',
        approved: '通过',
        waiting: '待审核',
        spam: '垃圾',
        unsticky: '取消置顶',
        oldest: '按倒序',
        latest: '按正序',
        hottest: '按热度',
        reactionTitle: '你认为这篇文章怎么样？',
    };
    init({
        // options
        el: '#waline',
        serverURL: 'https://comment.dvel.me',
        locale,
        emoji: false,     // 表情
        search: false,    // GIF 表情包
        reaction: false,  // 文章反应
        requiredMeta: ['nick'],
        pageSize: 10,
        imageUploader: false,
        copyright: true,
        pageview: true,
        like: false,
    });
</script>

{{- /* Comments area end */ -}}
~~~
img\01\AS.jpg
# 侧边目录
# 相关推荐
创建一个 layouts/partials/related.html 文件填写一下代码
~~~
{{ $related := .Site.RegularPages.Related . | first 5 }}
{{ with $related }}
<h3>See Also</h3>
<ul>
 {{ range . }}
 <li><a href="{{ .RelPermalink }}">{{ .LinkTitle }}</a></li>
 {{ end }}
</ul>
{{ end }}
~~~

 
