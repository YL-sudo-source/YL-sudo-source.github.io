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
1、在项目目录layouts/partials下添加toc.html文件
~~~
{{- $headers := findRE "<h[1-6].*?>(.|\n])+?</h[1-6]>" .Content -}}
{{- $has_headers := ge (len $headers) 1 -}}
{{- if $has_headers -}}
<aside id="toc-container" class="toc-container wide">
    <div class="toc">
        <details {{if (.Param "TocOpen") }} open{{ end }}>
            <summary accesskey="c" title="(Alt + C)">
                <span class="details">{{- i18n "toc" | default "Table of Contents" }}</span>
            </summary>

            <div class="inner">
                {{- $largest := 6 -}}
                {{- range $headers -}}
                {{- $headerLevel := index (findRE "[1-6]" . 1) 0 -}}
                {{- $headerLevel := len (seq $headerLevel) -}}
                {{- if lt $headerLevel $largest -}}
                {{- $largest = $headerLevel -}}
                {{- end -}}
                {{- end -}}

                {{- $firstHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers 0) 1) 0)) -}}

                {{- $.Scratch.Set "bareul" slice -}}
                <ul>
                    {{- range seq (sub $firstHeaderLevel $largest) -}}
                    <ul>
                        {{- $.Scratch.Add "bareul" (sub (add $largest .) 1) -}}
                        {{- end -}}
                        {{- range $i, $header := $headers -}}
                        {{- $headerLevel := index (findRE "[1-6]" . 1) 0 -}}
                        {{- $headerLevel := len (seq $headerLevel) -}}

                        {{/* get id="xyz" */}}
                        {{- $id := index (findRE "(id=\"(.*?)\")" $header 9) 0 }}

                        {{- /* strip id="" to leave xyz, no way to get regex capturing groups in hugo */ -}}
                        {{- $cleanedID := replace (replace $id "id=\"" "") "\"" "" }}
                        {{- $header := replaceRE "<h[1-6].*?>((.|\n])+?)</h[1-6]>" "$1" $header -}}

                        {{- if ne $i 0 -}}
                        {{- $prevHeaderLevel := index (findRE "[1-6]" (index $headers (sub $i 1)) 1) 0 -}}
                        {{- $prevHeaderLevel := len (seq $prevHeaderLevel) -}}
                        {{- if gt $headerLevel $prevHeaderLevel -}}
                        {{- range seq $prevHeaderLevel (sub $headerLevel 1) -}}
                        <ul>
                            {{/* the first should not be recorded */}}
                            {{- if ne $prevHeaderLevel . -}}
                            {{- $.Scratch.Add "bareul" . -}}
                            {{- end -}}
                            {{- end -}}
                            {{- else -}}
                            </li>
                            {{- if lt $headerLevel $prevHeaderLevel -}}
                            {{- range seq (sub $prevHeaderLevel 1) -1 $headerLevel -}}
                            {{- if in ($.Scratch.Get "bareul") . -}}
                        </ul>
                        {{/* manually do pop item */}}
                        {{- $tmp := $.Scratch.Get "bareul" -}}
                        {{- $.Scratch.Delete "bareul" -}}
                        {{- $.Scratch.Set "bareul" slice}}
                        {{- range seq (sub (len $tmp) 1) -}}
                        {{- $.Scratch.Add "bareul" (index $tmp (sub . 1)) -}}
                        {{- end -}}
                        {{- else -}}
                    </ul>
                    </li>
                    {{- end -}}
                    {{- end -}}
                    {{- end -}}
                    {{- end }}
                    <li>
                        <a href="#{{- $cleanedID -}}" aria-label="{{- $header | plainify -}}">{{- $header | safeHTML -}}</a>
                        {{- else }}
                    <li>
                        <a href="#{{- $cleanedID -}}" aria-label="{{- $header | plainify -}}">{{- $header | safeHTML -}}</a>
                        {{- end -}}
                        {{- end -}}
                        <!-- {{- $firstHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers 0) 1) 0)) -}} -->
                        {{- $firstHeaderLevel := $largest }}
                        {{- $lastHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers (sub (len $headers) 1)) 1) 0)) }}
                    </li>
                    {{- range seq (sub $lastHeaderLevel $firstHeaderLevel) -}}
                    {{- if in ($.Scratch.Get "bareul") (add . $firstHeaderLevel) }}
                </ul>
                {{- else }}
                </ul>
                </li>
                {{- end -}}
                {{- end }}
                </ul>
            </div>
        </details>
    </div>
</aside>
<script>
    let activeElement;
    let elements;
    window.addEventListener('DOMContentLoaded', function (event) {
        checkTocPosition();

        elements = document.querySelectorAll('h1[id],h2[id],h3[id],h4[id],h5[id],h6[id]');
         // Make the first header active
         activeElement = elements[0];
         const id = encodeURI(activeElement.getAttribute('id')).toLowerCase();
         document.querySelector(`.inner ul li a[href="#${id}"]`).classList.add('active');
     }, false);

    window.addEventListener('resize', function(event) {
        checkTocPosition();
    }, false);

    window.addEventListener('scroll', () => {
        // Check if there is an object in the top half of the screen or keep the last item active
        activeElement = Array.from(elements).find((element) => {
            if ((getOffsetTop(element) - window.pageYOffset) > 0 && 
                (getOffsetTop(element) - window.pageYOffset) < window.innerHeight/2) {
                return element;
            }
        }) || activeElement

        elements.forEach(element => {
             const id = encodeURI(element.getAttribute('id')).toLowerCase();
             if (element === activeElement){
                 document.querySelector(`.inner ul li a[href="#${id}"]`).classList.add('active');
             } else {
                 document.querySelector(`.inner ul li a[href="#${id}"]`).classList.remove('active');
             }
         })
     }, false);

    const main = parseInt(getComputedStyle(document.body).getPropertyValue('--article-width'), 10);
    const toc = parseInt(getComputedStyle(document.body).getPropertyValue('--toc-width'), 10);
    const gap = parseInt(getComputedStyle(document.body).getPropertyValue('--gap'), 10);

    function checkTocPosition() {
        const width = document.body.scrollWidth;

        if (width - main - (toc * 2) - (gap * 4) > 0) {
            document.getElementById("toc-container").classList.add("wide");
        } else {
            document.getElementById("toc-container").classList.remove("wide");
        }
    }

    function getOffsetTop(element) {
        if (!element.getClientRects().length) {
            return 0;
        }
        let rect = element.getBoundingClientRect();
        let win = element.ownerDocument.defaultView;
        return rect.top + win.pageYOffset;   
    }
</script>
{{- end }}

~~~
2、在项目目录assets/css/extended下添加blank.css文件，内容如下：
~~~
:root {
    --nav-width: 1380px;
    --article-width: 650px;
    --toc-width: 300px;
}

.toc {
    margin: 0 2px 40px 2px;
    border: 1px solid var(--border);
    background: var(--entry);
    border-radius: var(--radius);
    padding: 0.4em;
}

.toc-container.wide {
    position: absolute;
    height: 100%;
    border-right: 1px solid var(--border);
    left: calc((var(--toc-width) + var(--gap)) * -1);
    top: calc(var(--gap) * 2);
    width: var(--toc-width);
}

.wide .toc {
    position: sticky;
    top: var(--gap);
    border: unset;
    background: unset;
    border-radius: unset;
    width: 100%;
    margin: 0 2px 40px 2px;
}

.toc details summary {
    cursor: zoom-in;
    margin-inline-start: 20px;
    padding: 12px 0;
}

.toc details[open] summary {
    font-weight: 500;
}

.toc-container.wide .toc .inner {
    margin: 0;
}

.active {
    font-size: 110%;
    font-weight: 600;
}

.toc ul {
    list-style-type: circle;
}

.toc .inner {
    margin: 0 0 0 20px;
    padding: 0px 15px 15px 20px;
    font-size: 16px;

    /*目录显示高度*/
    max-height: 83vh;
    overflow-y: auto;
}

.toc .inner::-webkit-scrollbar-thumb {  /*滚动条*/
    background: var(--border);
    border: 7px solid var(--theme);
    border-radius: var(--radius);
}

.toc li ul {
    margin-inline-start: calc(var(--gap) * 0.5);
    list-style-type: none;
}

.toc li {
    list-style: none;
    font-size: 0.95rem;
    padding-bottom: 5px;
}

.toc li a:hover {
    color: var(--secondary);
}

~~~
# 相关推荐
1、创建一个 layouts/partials/related.html 文件填写一下代码
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
2、在 layouts/_default/single.html 文件中添加
~~~
    <!-- Related -->
    <div class="pagination__title">
      <span class="pagination__title-h">{{ .Site.Params.relatedTitle | default "Related"}}</span>
      <hr />
    </div>

    {{ partial "related.html" . }}
    <!-- /Related -->
~~~
 
