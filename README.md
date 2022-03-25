# Docsify使用文档

> ## <font color=green>Node.js 安装配置</font>

`n latest`

查看版本

- `node -v`
- `npm -v`

> ## <font color=green>全局安装 docsify-cli 工具</font>

`npm i docsify-cli -g`

> ## <font color=green>项目初始化</font>

`docsify init ./Docsify`

初始化成功后，在Docsify目录下会有几个文件
- index.html    入口文件
- README.md     会做为主页内容渲染
- .nojekyll     用于阻止 GitHub Pages 忽略掉下划线开头的文件
直接编辑 README.md 就能更新文档内容，也可以添加更多页面

> ## <font color=green>本地运行docsify创建项目</font>

`docsify serve Docsify`
```shell
erniudeMBP:Documents erniu$ docsify serve Docsify

Serving /Users/erniu/Documents/Docsify now.
Listening at http://localhost:3000
```

> ## <font color=green>基础配置文件介绍</font>

|文件作用|文件|
|-------|----|
|基础配置项（入口文件|index.html|
|封面配置文件|_coverpage.md|
|侧边栏配置文件|_sidebar.md|
|导航栏配置文件|_navbar.md|
|主页内容渲染文件|README.md|
|浏览器图标|favicon.ico|

> ## <font color=green>基础配置项（index.html）</font>

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Docsify-Guide</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="description" content="Description">
    <meta name="viewport"
        content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <!-- 设置浏览器图标 -->
    <link rel="icon" href="/favicon.ico" type="image/x-icon" />
    <link rel="shortcut icon" href="/favicon.ico" type="image/x-icon" />
    <!-- 默认主题 -->
    <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify/lib/themes/vue.css">
</head>

<body>
    <!-- 定义加载时候的动作 -->
    <div id="app">加载中...</div>
    <script>
        window.$docsify = {
            // 项目名称
            name: 'Docsify-Guide',
            // 仓库地址，点击右上角的Github章鱼猫头像会跳转到此地址
            repo: 'https://github.com/YSGStudyHards',
            // 侧边栏支持，默认加载的是项目根目录下的_sidebar.md文件
            loadSidebar: true,
            // 导航栏支持，默认加载的是项目根目录下的_navbar.md文件
            loadNavbar: true,
            // 封面支持，默认加载的是项目根目录下的_coverpage.md文件
            coverpage: true,
            // 最大支持渲染的标题层级
            maxLevel: 5,
            // 自定义侧边栏后默认不会再生成目录，设置生成目录的最大层级（建议配置为2-4）
            subMaxLevel: 4,
            // 小屏设备下合并导航栏到侧边栏
            mergeNavbar: true,
        }
    </script>
    <script>
        // 搜索配置(url：https://docsify.js.org/#/zh-cn/plugins?id=%e5%85%a8%e6%96%87%e6%90%9c%e7%b4%a2-search)
        window.$docsify = {
            search: {
                maxAge: 86400000,// 过期时间，单位毫秒，默认一天
                paths: auto,// 注意：仅适用于 paths: 'auto' 模式
                placeholder: '搜索',
                // 支持本地化
                placeholder: {
                    '/zh-cn/': '搜索',
                    '/': 'Type to search'
                },
                noData: '找不到结果',
                depth: 4,
                hideOtherSidebarContent: false,
                namespace: 'Docsify-Guide',
            }
        }
    </script>
    <!-- docsify的js依赖 -->
    <script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
    <!-- emoji表情支持 -->
    <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/emoji.min.js"></script>
    <!-- 图片放大缩小支持 -->
    <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/zoom-image.min.js"></script>
    <!-- 搜索功能支持 -->
    <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
    <!--在所有的代码块上添加一个简单的Click to copy按钮来允许用户从你的文档中轻易地复制代码-->
    <script src="//cdn.jsdelivr.net/npm/docsify-copy-code/dist/docsify-copy-code.min.js"></script>
</body>

</html>
```

> ## <font color=green>封面配置文件（\_coverpage.md)</font>

[Docsify官网封面配置教程](https://docsify.js.org/#/zh-cn/cover)

> ### <font color=green>index.html</font>
```html
<!-- index.html -->

<script>
  window.$docsify = {
    coverpage: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

> ### <font color=green>\_coverpage.md</font>

```html
<!-- _coverpage.md -->

# Docsify使用指南 

> 💪Docsify使用指南，使用Typora+Docsify打造最强、最轻量级的个人&团队文档。

 简单、轻便 (压缩后 ~21kB)
- 无需生成 html 文件
- 众多主题


[开始使用 Let Go](/README.md)
```

> ## <font color=green>侧边栏配置文件（\_sidebar.md)</font>

[Docsify官网侧边栏配置教程](https://docsify.js.org/#/zh-cn/more-pages?id=%e5%ae%9a%e5%88%b6%e4%be%a7%e8%be%b9%e6%a0%8f)

> ### <font color=green>index.html</font>

```html
<!-- index.html -->

<script>
  window.$docsify = {
    loadSidebar: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

```
//自定义侧边栏后默认不会再生成目录，设置生成目录的最大层级，建议配置为1或者2
subMaxLevel: 2
```

> ### <font color=green>\_sidebar.md</font>

```html
<!-- _sidebar.md -->

* Typora+Docsify使用指南
  * [Docsify使用指南](/ProjectDocs/Docsify使用指南.md) <!--注意这里是相对路径-->
  * [Typora+Docsify快速入门](/ProjectDocs/Typora+Docsify快速入门.md)
* Docsify部署
  * [Docsify部署教程](/ProjectDocs/Docsify部署教程.md)
```

> ## <font color=green>导航栏配置文件（\_navbar.md)</font>

[Docsify官网导航栏配置教程](https://docsify.js.org/#/zh-cn/custom-navbar?id=%e9%85%8d%e7%bd%ae%e6%96%87%e4%bb%b6)

> ### <font color=green>index.html</font>

```html
<!-- index.html -->

<script>
  window.$docsify = {
    loadNavbar: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

> ### <font color=green>\_navbar.md</font>

```html
<!-- _navbar.md -->

* 链接到我
  * [博客园地址](https://www.cnblogs.com/Can-daydayup/)
  * [Github地址](https://github.com/YSGStudyHards)
  * [知乎地址](https://www.zhihu.com/people/ysgdaydayup)
  * [掘金地址](https://juejin.cn/user/2770425031690333/posts)
  * [Gitee地址](https://gitee.com/ysgdaydayup)


* 友情链接
  * [Docsify](https://docsify.js.org/#/)
  * [博客园](https://www.cnblogs.com/)
```

> ## <font color=green>全文搜索 - Search</font>

[全文搜索 - Search](https://docsify.js.org/#/zh-cn/plugins?id=%E5%85%A8%E6%96%87%E6%90%9C%E7%B4%A2-search)

> ## <font color=green>Docsify主题切换</font>

切换主体只需要在根目录的index.html切换对应的主体CSS文件即可

详情参考：[https://docsify.js.org/#/zh-cn/themes](https://docsify.js.org/#/zh-cn/themes)

> ## <font color=green>Docsify详细部署教程</font>

[Docsify部署教程](https://github.com/YSGStudyHards/Docsify-Guide/blob/main/ProjectDocs/Docsify%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B.md)



