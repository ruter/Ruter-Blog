---
title: 「API 翻译与应用」- Pixabay
categories: Translation
keywords: API, Translation
toc: true
date: 2017-07-02 17:14:50
description: 又到了挖坑的时候，新系列「API 翻译与应用」开坑啦
---

这是新开坑的系列——「API 翻译与应用」的第 `1` 篇文章。

这个系列的主要目的在于翻译一些国外的好用又好玩的 API 文档，并通过实例应用（如果我不挖坑不填的话）进行使用说明。另一方面，很多人可能在阅读英文文档的时候比较没耐心（实不相瞒我就是其中之一），会忽略掉一些比较重要的信息，例如使用限制等。

这个系列的第一篇文章我要献给  [Pixabay](https://pixabay.com/) ，一个资源丰富又好用的无版权图库，也是为数不多的可以良好地支持中文关键词检索的图库，相信经常找图片素材的小伙伴一定不会陌生。

套用官方  [FAQ](https://pixabay.com/zh/service/faq/)  中的回答来介绍一下这次的主角：

> Pixabay是什么？在Pixabay，你可以查找和分享免费的无版权图像，这里所有的图像都以知识共享组织(Creative Commons) 的公共财产CC0协议发布。

下面是 Pixabay 官方 API 文档的翻译，有些地方翻译不恰当恳请斧正，先谢过各位老铁了，如果想直接看英文文档的可以点击「 [传送门](https://pixabay.com/api/docs/) 」前往~

# Pixabay API 文档

欢迎来到 Pixabay 的 API 文档，我们的 API 是一个用于查找 Pixabay 上以 CC0 协议发布的图片和视频的 RESTful 接口。

如果你使用了本 API，请在搜索结果展示给你的用户时告诉他们这些图片和视频是从哪里来的。一个链接到 Pixabay 的链接是必要的，为此你可以使用我们的  [logo](https://pixabay.com/zh/service/about/#goodies) 作为图片超链接。这是我们对免费使用 API 所恳请的唯一回报。

![Pixabay's logo](https://pixabay.com/static/img/logo.svg)

API 返回的是 JSON 编码的对象，散列键和值都是大小写敏感的，字符编码为 UTF-8。散列键会以随机的顺序返回，并且新的键可能会在任意的时间被加入。在从结果中移除散列键或者添加必要的查询参数前，我们会尽最大的努力通知我们的用户，但是我们偶尔也可能会在没有通知用户的情形下做这些动作。Pixabay API 不作保证。

## 频率限制

默认情况下，每小时的请求上限是 5,000 次。每一次请求都会带上 API 的 key，但是不会包含你的 IP 地址。响应头会告诉你一切你需要知道的当前的频率限制状态：

| 头部信息名                 | 说明                    |
| --------------------- | --------------------- |
| X-RateLimit-Limit     | 用户在30分钟内被允许的最大请求次数    |
| X-RateLimit-Remaining | 在当前频率限制窗口下剩余的请求数      |
| X-rateLimit-Reset     | 距离重设当前频率限制窗口所剩时间（按秒计） |

**为了保证 Pixabay API 的速度，所有的请求都必须缓存24小时。**另外，本 API 是给真实的人类请求使用的，不要发送大量的自动查询请求。大规模下载是不允许的。

## 热链

返回的图片链接可以被用于临时性地展示搜索结果。但是，持久化的图片热链（直接将它们嵌入到你的应用中）是不允许的。如果你想要使用这些图片，请先将它们下载到你的服务器上。视频是可以被直接嵌入到你的应用中的，但我们还是推荐将它们存储到你的服务器上。

## 错误处理

如果出现了错误，一个包含 HTTP 错误状态码的响应会被返回。这个响应包含了关于问题的纯文本的描述。例如，一旦你超出了频率限制，你会收到一个含有信息“API 超出频率限制”的 HTTP 错误429（“请求过多”）。

## 搜索图片

> GET <https://pixabay.com/api/>

### 参数

| 哈希键            | 类型       | 说明                                       |
| -------------- | -------- | ---------------------------------------- |
| key (必需)       | *str*    | 登录后即可查看                                  |
| q              | *str*    | 一个 URL 编码的搜索词。如果忽略则返回全部图片。值不能超过100个字符。示例：”yellow+flower” |
| lang           | *str*    | 用[语言代码](https://zh.wikipedia.org/wiki/ISO_639-1)对应的语言进行搜索。接受的值：cs, da, de, en, es, fr, id, it, hu, nl, no, pl, pt, ro, sk, fi, sv, tr, vi, th, bg, ru, el, ja, ko, zh。默认值：”en” |
| id             | *str*    | 用于检索特定图片的 ID，哈希后的 ID 或一组由逗号分隔的值。在由逗号分隔的一组数中，普通的 ID 和哈希后的 ID 不能放置在一起。 |
| response_group | *str*    | 在高分辨率图像和图像细节之间选择进行检索。若选择细节，可以访问到的图片最高尺寸为 960 x 720 px。接受的值：”image_details”, “high_resolution” (需要许可)。默认值：”image_details” |
| image_type     | *str*    | 按图片类型过滤结果。接受的值：”all”, “photo”, “illustration”, “vector”。默认值：”all” |
| orientation    | *str*    | 按图片的方向过滤结果。接受的值：”all”, “horizontal”, “vertical”。默认值：”all” |
| category       | *str*    | 按分类过滤结果。接受的值： fashion, nature, backgrounds, science, education, people, feelings, religion, health, places, animals, industry, food, computer, sports, transportation, travel, buildings, business, music |
| min_width      | *int*    | 图片最小宽。默认值：”0”                            |
| min_height     | *int*    | 图片最小高。默认值：”0”                            |
| editors_choice | *bool*   | 选择曾获得过[编辑精选](https://pixabay.com/zh/editors_choice/)奖的图片。接受的值：”true”, “false”。默认值：”false” |
| safesearch     | *bool*   | 是否只返回适合全年龄的图片。接受的值：”true”, “false”。默认值：”false” |
| order          | *str*    | 结果的排序方式。接受的值：”popular”, “latest”。默认值：”popular” |
| page           | *int*    | 返回的搜索结果是分页的。使用这个参数选择页码。默认值：1             |
| per_page       | *int*    | 决定每页结果的数量。接受的值：3 - 200。默认值：20            |
| callback       | *string* | JSONP 的回调函数名                             |
| pretty         | *bool*   | JSON 格式化输出。该选项不应用于生产环境。接受的值：”true”, “false”。默认值：”false” |

### 示例 1：默认的图片搜索

检索关于 “yellow flowers” 的 web 格式照片，检索关键词`q`需要进行 URL 编码。

[https://pixabay.com/api/?key={这里是key}&q=yellow+flowers&image_type=photo&pretty=true](https://pixabay.com/api/?key=&q=yellow+flowers&image_type=photo&pretty=true)

**注：请自行替换链接中的key部分**

这个请求的响应内容如下：

```json
{
    "total": 4692,
    "totalHits": 500,
    "hits": [
        {
            "id": 195893,
            "pageURL": "https://pixabay.com/en/blossom-bloom-flower-yellow-close-195893/",
            "type": "photo",
            "tags": "blossom, bloom, flower",
            "previewURL": "https://static.pixabay.com/photo/2013/10/15/09/12/flower-195893_150.jpg"
            "previewWidth": 150,
            "previewHeight": 84,
            "webformatURL": "https://pixabay.com/get/35bbf209db8dc9f2fa36746403097ae226b796b9e13e39d2_640.jpg",
            "webformatWidth": 640,
            "webformatHeight": 360,
            "imageWidth": 4000,
            "imageHeight": 2250,
            "imageSize": 4731420,
            "views": 7671,
            "downloads": 6439,
            "favorites": 1,
            "likes": 5,
            "comments": 2,
            "user_id": 48777,
            "user": "Josch13",
            "userImageURL": "https://static.pixabay.com/user/2013/11/05/02-10-23-764_250x250.jpg",
        },
        {
            "id": 14724,
            ...
        },
        ...
    ]
}
```

| 响应键           | 说明                                       |
| ------------- | ---------------------------------------- |
| total         | 命中的结果总数。                                 |
| totalHits     | 通过 API 可以访问到的图片数量。API 默认限制每次查询返回的最大图片数为 500 张。 |
| id            | 用于更新过期图片的 URL 的唯一标识。                     |
| pageURL       | 在 Pixabay 上的源网页，提供了下载尺寸图片的链接和图片文件大小。     |
| previewURL    | 预览链接，最大宽或高为 150 px 的低分辨率图片。              |
| webformatURL  | 最大宽或高为 640 px 的中等尺寸图片。链接 24 小时内有效。**替换掉任意 webformatURL 上的 ‘_640’ 可以访问其他图片尺寸：**替换为 ‘_180’ 或 ‘_340’ 可以分别得到 180 或 340 px 的图片。替换为 ‘_960’ 以得到最大尺寸为 960 x 720 px 的图片。 |
| views         | 总访问量。                                    |
| downloads     | 总下载量。                                    |
| favorites     | 总收藏量。                                    |
| likes         | 总点赞量。                                    |
| comments      | 总评论数。                                    |
| user_id, user | 图片贡献者的用户 ID 和名字。个人资料链接：[ https://pixabay.com/users/{ USERNAME }-{ ID }/](http://ruterly.com/2017/07/02/Pixabay-API-translate-and-application/#) |
| userImageURL  | 用户头像链接（250 x 250 px）。                    |

### 示例 2：高分辨率图片搜索 (需要许可)

使用`response_group=high_resolution`检索高分辨率图片。图片的详细数据，例如标签和源网页链接，都是不会出现在响应中的。

[https://pixabay.com/api/?key={这里是key}&response_group=high_resolution&q=yellow+flower](https://pixabay.com/api/?key=&response_group=high_resolution&q=yellow+flower&pretty=true)

该请求的响应数据：

```Json
{
    "total": 4692
    "totalHits": 500,
    "hits": [
        {
            "id_hash": "98b12d28d77432a2",
            "type": "photo",
            "previewURL": "https://pixabay.com/get/ed6a9364aea79b54a6290d9388e0722ad543e89fd0a76647_150.jpg",
            "previewWidth": 150,
            "previewHeight": 84,
            "webformatURL": "https://pixabay.com/get/ed6a9364aea79b54a6290d9388e0722ad543e89fd0a76647_640.jpg",
            "webformatWidth": 640,
            "webformatHeight": 360,
            "largeImageURL": "https://pixabay.com/get/ed6a9364aea79b54a6290d9388e0722ad543e89fd0a76647_1280.jpg",
            "fullHDURL": "https://pixabay.com/get/ed6a9364aea79b54a6290d9388e0722ad543e89fd0a76647_1920.jpg",
            "imageWidth": 4000,
            "imageHeight": 2250,
            "imageSize": 4731420,
            "imageURL": "https://pixabay.com/get/ed6a9364aea79b54a6290d9388e0722ad543e89fd0a76647.jpg",
            "user_id": 48777,
            "user": "Josch13"
            "userImageURL": "https://static.pixabay.com/user/2013/11/05/02-10-23-764_250x250.jpg",
        },
        {
            "id_hash": "bb4d3acd9b2b4650",
            ...
        },
        ...
    ]
}
```

| 响应键           | 说明                                       |
| ------------- | ---------------------------------------- |
| id_hash       | 用于更新过期图片的 URL 的唯一标识。哈希 ID 还可以用来 [构建 Pixabay 页面的链接。](https://pixabay.com/api/docs/#id_hash_urls) |
| largeImageURL | 按比例缩放到宽或高为 1280 px 的图片。                  |
| fullHDURL     | 按比例缩放到最大宽或高为 1920 px 的全高清图片。             |
| imageURL      | 原图的链接。                                   |
| imageSize     | 完整图片的文件尺寸。                               |
| vectorURL     | 矢量资源的链接，如果没有则会被省略。                       |

*其他键的说明请参考示例 1*。

### 示例 3：用 ID 或 哈希 ID 检索图片

ID 和 哈希后的 ID 都可以用于刷新所选图片的数据。多个 ID 或 哈希 ID 可以用逗号分隔，但是，你不能将它们混用在一个请求内。

a) 用 ID 获取图片详细信息；响应参考示例 1。

[https://pixabay.com/api/?key={这里是key}&id=11574](https://pixabay.com/api/?key=&id=11574&pretty=true)

b) 用哈希 ID 获取高分辨率的图片；响应参考示例 2。

[https://pixabay.com/api/?key={这里是key}&response_group=high_resolution&id=bb4d3acd9b2b4650](https://pixabay.com/api/?key=&response_group=high_resolution&id=bb4d3acd9b2b4650&pretty=true)

### 示例 4：用哈希 ID 构建 Pixabay 的链接

哈希 ID 可以用于创建一个对应 Pixabay 图片的链接。未授权用户访问该页面时需要先填写验证码。URL 的组成：[https://pixabay.com/goto/{](https://pixabay.com/goto/%7B) ID_HASH }/

<https://pixabay.com/goto/bb4d3acd9b2b4650/>

## 搜索视频

> GET <https://pixabay.com/api/videos/>

### 参数

| 哈希键            | 类型       | 说明                                       |
| -------------- | -------- | ---------------------------------------- |
| key (必需)       | str      | 登录后即可查看                                  |
| q              | str      | 一个 URL 编码的搜索词。如果忽略则返回全部视频。值不能超过100个字符。示例：”yellow+flower” |
| lang           | str      | 用[语言代码](https://zh.wikipedia.org/wiki/ISO_639-1)对应的语言进行搜索。接受的值：cs, da, de, en, es, fr, id, it, hu, nl, no, pl, pt, ro, sk, fi, sv, tr, vi, th, bg, ru, el, ja, ko, zh。默认值：”en” |
| id             | str      | 用于检索特定视频的 ID 或一组由逗号分隔的值。                 |
| video_type     | str      | 按视频类型过滤结果。接受的值：”all”, “film”, “animation”。默认值：”all” |
| category       | str      | 按分类过滤结果。接受的值： fashion, nature, backgrounds, science, education, people, feelings, religion, health, places, animals, industry, food, computer, sports, transportation, travel, buildings, business, music |
| min_width      | int      | 视频最小宽。默认值：”0”                            |
| min_height     | int      | 视频最小高。默认值：”0”                            |
| editors_choice | bool     | 选择曾获得过[编辑精选](https://pixabay.com/zh/editors_choice/)奖的视频。接受的值：”true”, “false”。默认值：”false” |
| safesearch     | bool     | 是否只返回适合全年龄的视频。接受的值：”true”, “false”。默认值：”false” |
| order          | *str*    | 结果的排序方式。接受的值：”popular”, “latest”。默认值：”popular” |
| page           | *int*    | 返回的搜索结果是分页的。使用这个参数选择页码。默认值：1             |
| per_page       | *int*    | 决定每页结果的数量。接受的值：3 - 200。默认值：20            |
| callback       | *string* | JSONP 的回调函数名                             |
| pretty         | *bool*   | JSON 格式化输出。该选项不应用于生产环境。接受的值：”true”, “false”。默认值：”false” |

### 示例

检索关于 “yellow flowers” 的视频，检索关键词`q`需要进行 URL 编码。

[https://pixabay.com/api/videos/?key={这里是key}&q=yellow+flowers](https://pixabay.com/api/videos/?key=&q=yellow+flowers&pretty=true)

该请求的响应内容：

```Json
{
    "total": 42,
    "totalHits": 42,
    "hits": [
        {
            "id": 125,
            "pageURL": "https://pixabay.com/videos/id-125/",
            "type": "film",
            "tags": "flowers, yellow, blossom",
            "duration": 12,
            "picture_id": "529927645",
            "videos": {
                "large": {
                    "url": "https://player.vimeo.com/external/135736646.hd.mp4?s=ed02d71c92dd0df7d1110045e6eb65a6&profile_id=119",
                    "width": 1920,
                    "height": 1080,
                    "size": 6615235
                },
                "medium": {
                    "url": "https://player.vimeo.com/external/135736646.hd.mp4?s=ed02d71c92dd0df7d1110045e6eb65a6&profile_id=174",
                    "width": 1280,
                    "height": 720,
                    "size": 3562083
                },
                "small": {
                    "url": "https://player.vimeo.com/external/135736646.sd.mp4?s=db2924c48ef91f17fc05da74603d5f89&profile_id=165",
                    "width": 950,
                    "height": 540,
                    "size": 2030736
                },
                "tiny": {
                    "url": "https://player.vimeo.com/external/135736646.sd.mp4?s=db2924c48ef91f17fc05da74603d5f89&profile_id=164",
                    "width": 640,
                    "height": 360,
                    "size": 1030736
                }
            },
            "views": 169,
            "downloads": 66,
            "favorites": 7,
            "likes": 3,
            "comments": 2,
            "user_id": 1281706,
            "user": "CoverrFreeFootage",
            "userImageURL": "https://static.pixabay.com/user/2015/10/16/09-28-45-303_250x250.png"
        },
        {
            "id": 14724,
            ...
        },
        ...
    ]
}
```

| 响应键           | 说明                                       |
| ------------- | ---------------------------------------- |
| total         | 命中的结果总数。                                 |
| totalHits     | 通过 API 可以访问到的视频数量。API 默认限制每次查询返回的最大视频数为 500 个。 |
| id            | 视频的唯一标识。                                 |
| pageURL       | 在 Pixabay 上的源网页。                         |
| picture_id    | 这个值可以用于检索视频的静态预览图片，有多种尺寸： [https://i.vimeocdn.com/video/{](https://i.vimeocdn.com/video/%7B) PICTURE*ID }\*{ SIZE }.jpg 可用的尺寸有：100x75, 200x150, 295x166, 640x360, 960x540, 1920x1080。示例： <https://i.vimeocdn.com/video/529927645_295x166.jpg> |
| videos        | 一组不同尺寸的视频流：”large” 通常分辨率有 1920x1080。如果大的视频不可用，一个空的 URL 和大小 0 将会被返回。”medium” 的分辨率为 1280x720，所有 Pixabay 的视频都有这个尺寸。”small” 一般分辨率为 960x540，旧的视频分辨率为 640x360。这个尺寸所有的视频都有。”tiny” 的分辨率为 640x360，旧的视频分辨率为 480x270。同样的所有视频都有这个尺寸。给任意的视频流链接加上 GET 参数 “download=1” 将会变成下载。 |
| views         | 总访问量。                                    |
| downloads     | 总下载量。                                    |
| favorites     | 总收藏量。                                    |
| likes         | 总点赞量。                                    |
| comments      | 总评论数。                                    |
| user_id, user | 贡献者的用户 ID 和名字。个人资料链接：[ https://pixabay.com/users/{ USERNAME }-{ ID }/](http://ruterly.com/2017/07/02/Pixabay-API-translate-and-application/#) |
| userImageURL  | 用户头像链接（250 x 250 px）。                    |

## JavaScript 示例

```javascript
var API_KEY = '{这里是key}';
var URL = "https://pixabay.com/api/?key="+API_KEY+"&q="+encodeURIComponent('red roses');
$.getJSON(URL, function(data){
    if (parseInt(data.totalHits) > 0)
        $.each(data.hits, function(i, hit){ console.log(hit.pageURL); });
    else
        console.log('No hits');
});
```

## 支持

如果你对 API 有任何疑问请[联系我们](https://pixabay.com/zh/service/contact/)。

## 开发者福利

下载免费的工具，jQuery 和 vanilla 的 JavaScript 插件以及其他有用的软件 - 由 Pixabay 开发。

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** http://ruterly.com/2017/07/02/Pixabay-API-Translation/
