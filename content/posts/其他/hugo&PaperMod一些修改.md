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
## 1、添加toc.html文件
在项目目录layouts/partials下添加toc.html文件
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
## 2、添加blank.css文件
在项目目录assets/css/extended下添加blank.css文件，内容如下：
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
## 3、single.html添加内容
在single.html添加
~~~
  {{- if (.Param "ShowToc") }}
  {{- partial "toc.html" . }}
  {{- end }}
~~~
# 设置相关推荐
## 1、
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
## 2、
在 layouts/_default/single.html 文件中添加
~~~
    <!-- Related -->
    <div class="pagination__title">
      <span class="pagination__title-h">{{ .Site.Params.relatedTitle | default "Related"}}</span>
      <hr />
    </div>

    {{ partial "related.html" . }}
    <!-- /Related -->
~~~

# 修改页脚
## 1、
将主题的./themes/hugo-PaperMod/layouts/partials/footer.html文件拷贝到hugo项目的根目录的./layouts/partials/ 文件夹中去，然后将
\<footer class="footer"\> \</footer\>
标签中的内容替换为如下内容：
~~~
<footer class="footer">
    <font color='#808695'> COPYRIGHT © 2018-2024 阿甘博客 </font> <a href="https://icp.gov.moe/?keyword=20230078"
        target="_blank" rel="noopener noreferrer nofollow">
        <font color='#808695'>萌ICP备20230078号 </font>
    </a></br>
    <font color='#808695'>本站由 <a href="https://app.cloudcone.com/?ref=10464" rel="noopener noreferrer nofollow"
            target="_blank">Cloudcone</a> 提供计算服务, 由 <a href="https://www.cloudflare.com/"
            rel="noopener noreferrer nofollow" target="_blank">Cloudflare</a> 提供全站加速服务。</font>
    <br>
    <span id="span" style="color:#808695"></span>
    <script type="text/javascript">
        function runtime() {
            // 初始时间，日/月/年 时:分:秒
            X = new Date("12/07/2018 12:50:18");
            Y = new Date();
            T = (Y.getTime() - X.getTime());
            M = 24 * 60 * 60 * 1000;
            a = T / M;
            A = Math.floor(a);
            b = (a - A) * 24;
            B = Math.floor(b);
            c = (b - B) * 60;
            C = Math.floor((b - B) * 60);
            D = Math.floor((c - C) * 60);
            //信息写入到DIV中
            span.innerHTML = "本站已运行: " + A + "天" + B + "小时" + C + "分" + D + "秒"
        }
        setInterval(runtime, 1000);
    </script>
    </br>
    <script type="text/javascript" src="/js/jquery-3.6.0.min.js"></script>
    <!--引入JQuery，如果别处引入过滤，可省略-->
    <font color='#808695'>当前CloudFlare节点: <span id="cdn">unknown</span></font>
    <!--在适当的地方放入需要显示CDN节点的信息-->
    <script>
        getCDNinfo = function () {
            $.ajax({
                url: "/cdn-cgi/trace",
                success: function (data, status) {
                    let areas = "Antananarivo, Madagascar - (TNR);Cape Town, South Africa - (CPT);Casablanca, Morocco - (CMN);Dar Es Salaam, Tanzania - (DAR);Djibouti City, Djibouti - (JIB);Durban, South Africa - (DUR);Johannesburg, South Africa - (JNB);Kigali, Rwanda - (KGL);Lagos, Nigeria - (LOS);Luanda, Angola - (LAD);Maputo, MZ - (MPM);Mombasa, Kenya - (MBA);Port Louis, Mauritius - (MRU);Réunion, France - (RUN);Bangalore, India - (BLR);Bangkok, Thailand - (BKK);Bandar Seri Begawan, Brunei - (BWN);Cebu, Philippines - (CEB);Chengdu, China - (CTU);Chennai, India - (MAA);Chittagong, Bangladesh - (CGP);Chongqing, China - (CKG);Colombo, Sri Lanka - (CMB);Dhaka, Bangladesh - (DAC);Dongguan, China - (SZX);Foshan, China - (FUO);Fuzhou, China - (FOC);Guangzhou, China - (CAN);Hangzhou, China - (HGH);Hanoi, Vietnam - (HAN);Hengyang, China - (HNY);Ho Chi Minh City, Vietnam - (SGN);Hong Kong - (HKG);Hyderabad, India - (HYD);Islamabad, Pakistan - (ISB);Jakarta, Indonesia - (CGK);Jinan, China - (TNA);Karachi, Pakistan - (KHI);Kathmandu, Nepal - (KTM);Kolkata, India - (CCU);Kuala Lumpur, Malaysia - (KUL);Lahore, Pakistan - (LHE);Langfang, China - (NAY);Luoyang, China - (LYA);Macau - (MFM);Malé, Maldives - (MLE);Manila, Philippines - (MNL);Mumbai, India - (BOM);Nagpur, India - (NAG);Nanning, China - (NNG);New Delhi, India - (DEL);Osaka, Japan - (KIX);Phnom Penh, Cambodia - (PNH);Qingdao, China - (TAO);Seoul, South Korea - (ICN);Shanghai, China - (SHA);Shenyang, China - (SHE);Shijiazhuang, China - (SJW);Singapore, Singapore - (SIN);Suzhou, China - (SZV);Taipei - (TPE);Thimphu, Bhutan - (PBH);Tianjin, China - (TSN);Tokyo, Japan - (NRT);Ulaanbaatar, Mongolia - (ULN);Vientiane, Laos - (VTE);Wuhan, China - (WUH);Wuxi, China - (WUX);Xi'an, China - (XIY);Yerevan, Armenia - (EVN);Zhengzhou, China - (CGO);Zuzhou, China - (CSX);Amsterdam, Netherlands - (AMS);Athens, Greece - (ATH);Barcelona, Spain - (BCN);Belgrade, Serbia - (BEG);Berlin, Germany - (TXL);Brussels, Belgium - (BRU);Bucharest, Romania - (OTP);Budapest, Hungary - (BUD);Chișinău, Moldova - (KIV);Copenhagen, Denmark - (CPH);Cork, Ireland -  (ORK);Dublin, Ireland - (DUB);Düsseldorf, Germany - (DUS);Edinburgh, United Kingdom - (EDI);Frankfurt, Germany - (FRA);Geneva, Switzerland - (GVA);Gothenburg, Sweden - (GOT);Hamburg, Germany - (HAM);Helsinki, Finland - (HEL);Istanbul, Turkey - (IST);Kyiv, Ukraine - (KBP);Lisbon, Portugal - (LIS);London, United Kingdom - (LHR);Luxembourg City, Luxembourg - (LUX);Madrid, Spain - (MAD);Manchester, United Kingdom - (MAN);Marseille, France - (MRS);Milan, Italy - (MXP);Moscow, Russia - (DME);Munich, Germany - (MUC);Nicosia, Cyprus - (LCA);Oslo, Norway - (OSL);Paris, France - (CDG);Prague, Czech Republic - (PRG);Reykjavík, Iceland - (KEF);Riga, Latvia - (RIX);Rome, Italy - (FCO);Saint Petersburg, Russia - (LED);Sofia, Bulgaria - (SOF);Stockholm, Sweden - (ARN);Tallinn, Estonia - (TLL);Thessaloniki, Greece - (SKG);Vienna, Austria - (VIE);Vilnius, Lithuania - (VNO);Warsaw, Poland - (WAW);Zagreb, Croatia - (ZAG);Zürich, Switzerland - (ZRH);Arica, Chile - (ARI);Asunción, Paraguay - (ASU);Bogotá, Colombia - (BOG);Buenos Aires, Argentina - (EZE);Curitiba, Brazil - (CWB);Fortaleza, Brazil - (FOR);Guatemala City, Guatemala - (GUA);Lima, Peru - (LIM);Medellín, Colombia - (MDE);Panama City, Panama - (PTY);Porto Alegre, Brazil - (POA);Quito, Ecuador - (UIO);Rio de Janeiro, Brazil - (GIG);São Paulo, Brazil - (GRU);Santiago, Chile - (SCL);Willemstad, Curaçao - (CUR);St. George's, Grenada - (GND);Amman, Jordan - (AMM);Baghdad, Iraq - (BGW);Baku, Azerbaijan - (GYD);Beirut, Lebanon - (BEY);Doha, Qatar - (DOH);Dubai, United Arab Emirates - (DXB);Kuwait City, Kuwait - (KWI);Manama, Bahrain - (BAH);Muscat, Oman - (MCT);Ramallah - (ZDM);Riyadh, Saudi Arabia - (RUH);Tel Aviv, Israel - (TLV);Ashburn, VA, United States - (IAD);Atlanta, GA, United States - (ATL);Boston, MA, United States - (BOS);Buffalo, NY, United States - (BUF);Calgary, AB, Canada - (YYC);Charlotte, NC, United States - (CLT);Chicago, IL, United States - (ORD);Columbus, OH, United States - (CMH);Dallas, TX, United States - (DFW);Denver, CO, United States - (DEN);Detroit, MI, United States - (DTW);Honolulu, HI, United States - (HNL);Houston, TX, United States - (IAH);Indianapolis, IN, United States - (IND);Jacksonville, FL, United States - (JAX);Kansas City, MO, United States - (MCI);Las Vegas, NV, United States - (LAS);Los Angeles, CA, United States - (LAX);McAllen, TX, United States - (MFE);Memphis, TN, United States - (MEM);Mexico City, Mexico - (MEX);Miami, FL, United States - (MIA);Minneapolis, MN, United States - (MSP);Montgomery, AL, United States - (MGM);Montréal, QC, Canada - (YUL);Nashville, TN, United States - (BNA);Newark, NJ, United States - (EWR);Norfolk, VA, United States - (ORF);Omaha, NE, United States - (OMA);Philadelphia, United States - (PHL);Phoenix, AZ, United States - (PHX);Pittsburgh, PA, United States - (PIT);Port-Au-Prince, Haiti - (PAP);Portland, OR, United States - (PDX);Queretaro, MX, Mexico - (QRO);Richmond, Virginia - (RIC);Sacramento, CA, United States - (SMF);Salt Lake City, UT, United States - (SLC);San Diego, CA, United States - (SAN);San Jose, CA, United States - (SJC);Saskatoon, SK, Canada - (YXE);Seattle, WA, United States - (SEA);St. Louis, MO, United States - (STL);Tampa, FL, United States - (TPA);Toronto, ON, Canada - (YYZ);Vancouver, BC, Canada - (YVR);Tallahassee, FL, United States - (TLH);Winnipeg, MB, Canada - (YWG);Adelaide, SA, Australia - (ADL);Auckland, New Zealand - (AKL);Brisbane, QLD, Australia - (BNE);Melbourne, VIC, Australia - (MEL);Noumea, New caledonia - (NOU);Perth, WA, Australia - (PER);Sydney, NSW, Australia - (SYD)".split(";");
                    let area = data.split("colo=")[1].split("\n")[0];
                    for (var i = 0; i < areas.length; i++) {
                        if (areas[i].indexOf(area) != -1) {
                            document.getElementById("cdn").innerHTML = areas[i];
                            break;
                        }
                    }
                }
            })
        }
        $(document).ready(function () {
            getCDNinfo();
            //页面加载完毕就获取CDN信息
        });
    </script>
    <br/>
    <p><font color='#808695'>芝兰生于幽谷，不以无人而不芳；君子修道立德，不以穷困而变节。</font></p>
    <div id="cc-myssl-id" style="text-align: center;display:inline;">
        <div title="MySSL安全签章" id="myssl_seal"
            onclick="window.open('https://myssl.com/seal/detail?domain=sharpgan.com','MySSL安全签章','height=800,width=470,top=0,right=0,toolbar=no,menubar=no,scrollbars=no,resizable=no,location=no,status=no')"
            style="text-align: center;display: inline-block"><img src="/img/seal.png" alt="myssl.com-MySSL安全签章"
                style="width: 100px; height: auto; cursor: pointer"></div>
    </div>
    <div id="written-by-human-id" style="text-align: center;display:inline;">
        <div title="written by human" id="written-by-human" style="text-align: center;display: inline-block"><a
                href="https://notbyai.fyi/" rel="noopener noreferrer nofollow" target="_blank"><img
                    src="/img/Written-By-Human-Not-By-AI-Badge-white.png" alt="written by human badge"
                    style="width: 100px; height: auto; cursor: pointer"></a></div>
    </div>
</footer>

~~~

# 修改主题原色
## 1、
在中添加assets/css/extended/theme-vars-override.css 
添加
~~~
:root {
    --theme: #fff;
    --entry: #cfcfff;
    --primary: rgba(0, 0, 106, 0.88);
    --secondary: rgba(0, 0, 80, 0.78);
    --tertiary: rgba(0, 0, 106, 0.16);
    --content: rgba(0, 0, 60, 0.88);
    --hljs-bg: #1c1d21;
    --code-bg: #f5f5f5;
    --border: #eee;
}

.dark {
    --theme: #101c7a;
    --entry: #202062;
    --primary: rgba(235, 235, 255, 0.96);
    --secondary: rgba(235, 235, 255, 0.66);
    --tertiary: rgba(1, 1, 5, 0.32);
    --content: rgba(235, 235, 255, 0.82);
    --hljs-bg: #2e2e33;
    --code-bg: #37383e;
    --border: #446;
}
~~~

# 修改换页-页码
## 1、修改list.html文件
将文件内的模板文件修改为{{ partial "pagination.html" . }}   来引用,创建pagination.html文件  路径为layouts\partials\pagination.html    添加以下代码
~~~
    <!-- 换页 -->
{{- $paginator := .Paginate .Pages }} 

<!-- 分页容器，居中 -->
<div class="text-center">
    <div class="pagination">

        <!-- 页码 -->
        {{ if (le $paginator.TotalPages 5) }}
            {{ $current_num := $paginator.PageNumber }}
            {{ range $i, $pager := $paginator.Pagers }}
                {{ if (eq $current_num $pager.PageNumber) }}
                    <div class="page-item active">{{ $pager.PageNumber }}</div>
                {{ else }}
                    <a class="page-link" href="{{ $pager.URL }}">
                        <div class="page-item">{{ $pager.PageNumber }}</div>
                    </a>
                {{ end }}
            {{ end }}
        {{ else }}
            {{ $first_page_num := 1 }}
            {{ $second_page_num := 2 }}
            {{ $last_page_num := $paginator.TotalPages }}
            {{ $second_last_page_num := (add $paginator.TotalPages -1) }}
            {{ $third_last_page_num := (add $paginator.TotalPages -2) }}
            {{ $current_num := $paginator.PageNumber }}

            <!-- 第 1 页 -->
            {{ if (eq $current_num $first_page_num) }}
                <div class="page-item active">{{ $current_num }}</div>
            {{ else }}
                <a class="page-link" href="{{ $paginator.First.URL }}">
                    <div class="page-item">1</div>
                </a>
            {{ end }}

            <!-- 第 2 页 -->
            {{ if (eq $current_num $second_page_num) }}
                <div class="page-item active">{{ $current_num }}</div>
            {{ else if (le $current_num 3)}}
                <a class="page-link" href="{{ $paginator.First.Next.URL }}">
                    <div class="page-item">{{ $second_page_num }}</div>
                </a>
            {{ else }}
                <div class="page-item">...</div>
            {{ end }}

            <!-- 第 3 页 -->
            {{ if (le $current_num $second_page_num)}}
                <a class="page-link" href="{{ $paginator.First.Next.Next.URL }}">
                    <div class="page-item">3</div>
                </a>
            {{ else if (ge $current_num $second_last_page_num) }}
                <a class="page-link" href="{{ $paginator.Last.Prev.Prev.URL }}">
                    <div class="page-item">{{ $third_last_page_num }}</div>
                </a>
            {{ else }}
                <div class="page-item active">{{ $current_num }}</div>
            {{ end }}

            <!-- 第 4 页 -->
            {{ if (eq $current_num $second_last_page_num) }}
                <div class="page-item active">{{ $current_num }}</div>
            {{ else if (ge $current_num $third_last_page_num) }}
                <a class="page-link" href="{{ $paginator.Last.Prev.URL }}">
                    <div class="page-item">{{ $second_last_page_num }}</div>
                </a>
            {{ else }}
                <div class="page-item">...</div>
            {{ end }}

            <!-- 第 5 页 -->
            {{ if (eq $current_num $last_page_num) }}
                <div class="page-item active">{{ $current_num }}</div>
            {{ else }}
                <a class="page-link" href="{{ $paginator.Last.URL }}">
                    <div class="page-item">{{ $last_page_num }}</div>
                </a>
            {{ end }}
        {{ end }}

        <!-- 下一页 -->
        {{ if $paginator.HasNext }}
            <a class="page-link" href="{{ $paginator.Next.URL }}">
                <div class="page-item"> ></div>
            </a>
        {{ else }}
            <div class="page-item disabled"> ></div>
        {{ end }}
        

              

    </div>
</div>

~~~


## 添加css 
~~~
控制换页页码

/* 确保分页容器本身是居中的 */
.pagination {
    display: flex; /* 使用flex布局来排列元素 */
    justify-content: center; /* 将分页内容水平居中 */
    align-items: center; /* 将分页内容垂直居中 */
    gap: 10px; /* 页码之间的间距，可以根据需要调整 */
    margin: 20px 0; /* 分页容器上下的间距 */
}

/* 页面项 (每个页码) */
.pagination .page-item {
    display: inline-block; /* 设置为内联块元素，允许设置宽高 */
    padding: 5px 10px; /* 为每个页码添加内边距 */
    margin: 0; /* 去掉额外的边距 */
    cursor: pointer; /* 设置鼠标悬停时显示为可点击状态 */
    font-size: 32px; /* 设置页码文字的字体大小 */
}

/* 分页链接 */
.pagination .page-link {
    text-decoration: none; /* 去掉链接的下划线 */
    color: #007bff; /* 设置分页链接的字体颜色 */
    padding: 5px 10px; /* 为分页链接添加内边距 */
    font-size: 16px; /* 设置分页链接的字体大小 */
}

/* 活跃页码 */
.pagination .active {
    background-color: #007bff; /* 设置当前页的背景色 */
    color: white; /* 设置当前页的文字颜色 */
    border-radius: 5px; /* 设置圆角效果 */
}

/* 禁用页码 */
.pagination .disabled {
    color: #ccc; /* 设置禁用页码的文字颜色 */
    cursor: not-allowed; /* 设置禁用状态下的鼠标样式 */
}

~~~


 
