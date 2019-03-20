---
title:     "Hugo集成Algolia搜索"  # 文章标题
subtitle:    "Hugo集成Algolia搜索"  # 文章标题
description: ""
date:         2018-04-10T18:21:17+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["blog","其他"]
url:    "/2018-04-10-集成algolia搜索"
categories:  ["other"]
showtoc: true
---
## 1.简介
[Algolia](https://www.algolia.com/)是为你的 APP 或者网站添加搜索的最佳方式。 开发人员可以使用 API 上传并同步希望搜索的数据，然后可以进行相关的配置，比如产品转化率等等。可以使用 InstantSearch 等前端框架进行自定义搜索，为用户创造最佳的搜索体验。

## 2.注册
前往官方网站[https://www.algolia.com/](https://www.algolia.com/) 使用 GitHub 或 Google 帐号登录。登录完成后根据提示信息填写一些基本的信息即可，注册完成后前往 [Dashboard](https://www.algolia.com/dashboard)，我们可以发现 Algolia 会默认给我们生成一个 app。

![dashboard index](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/04/005.png)

选择 Indices，添加一个新的索引，我们这里命名为`hugo`，创建成功后，我们可以看到提示中还没有任何记录。

![indices](
https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/04/004.png)
Algolia 为我们提供了三种方式来增加记录：手动添加、上传 json 文件、API。我们这里使用第三种方式来进行数据的添加。

## 3.插件
要使用 API 的方式来添加搜索的数据，我们可以自己根据 Algolia 提供的 [API 文档](https://www.algolia.com/doc/api-reference/)进行开发，这也是很容易的，为简单起见，我们这里使用一个`hugo-algolia`的插件来完成我们的数据同步工作。

> 要安装`hugo-aligolia`我们需要先确保我们已经安装了 npm 或者 yarn 包管理工具。

使用下面的命令安装即可：
```shell
$ npm install hugo-algolia -g
```

安装完成后，在我们 hugo 生产的静态页面的根目录下面新建一个`config.yaml`的文件(和`config.toml`同级)，然后在`config.yaml`文件中指定 `Algolia`相关的 API 数据。
```yaml
baseurl: "/"
DefaultContentLanguage: "zh-cn"
hasCJKLanguage: true
languageCode: "zh-cn"
title: "River's Site"
theme: "beautifulhugo"
metaDataFormat: "yaml"

algolia_search = true
algolia_appId = "3DH4V2B4JK"
algolia_indexName = "hugo"
algolia_apiKey = "31c446dxxxxxxxxxxxxxxxxxxxxxxxx"

```

> API 相关数据可以前往 dashboard 的 `API Keys`查看，注意上面的`key`是**Admin API Key**。

配置完成以后，在根目录下面执行下面的命令：
```shell
$ hugo-algolia -s
JSON index file was created in public/algolia.json
{ updatedAt: '2019-03-20T10:29:03.861Z', taskID: 4896848941 }
```
然后我们可以看到，上面命令执行完成后会在`public`目录下面生成一个`algolia.json`的文件。这个时候我们在 dashboard 中打开 Indices，可以看到已经有几十条数据了。

![Indices](
https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/04/003.png)

> 如果某篇文章不想被索引的话，我们只需要在文件的最前面设置 index 参数为 false 即可，`hugo-algolia`插件在索引的过程中会自动跳过它。

## 4.前端

现在我们将需要被搜索的文章数据已经成功提交到`Algolia`，接下来的事情就是前端页面的展示了。下面的操作对于不同的主题或许有不同的地方，请根据自己的实际情况进行相应的修改。我这里使用的是`white`主题，在`themes/white/layouts/partials`目录下面新增文件：（**search.html**）
  ```html
  <!-- Including InstantSearch.js library and styling -->
  <script src="https://cdn.jsdelivr.net/npm/instantsearch.js@2.6.0/dist/instantsearch.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.20.1/moment.min.js"></script>
  <link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.6.0/dist/instantsearch.min.css">
  <link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.6.0/dist/instantsearch-theme-algolia.min.css">

  <div id="search-searchbar"></div>
   <div class="post-list" id="search-hits">
   </div>

  <script>
  const search = instantsearch({
    appId: '{{ .Site.Params.algolia_appId }}',
    indexName: '{{ .Site.Params.algolia_indexName }}',
    apiKey: '{{ .Site.Params.algolia_apiKey }}'
  });

  const hitTemplate = function(hit) {
  /*  if (hit === null){
        return;
    }*/
    // debugger;
    let date = '';
    if (hit.date) {
      date = moment(hit.date).format('MMM D, YYYY');
    }
    let url = `${hit.url}#${hit.author}`;
    const title = hit._highlightResult.title.value;

    let breadcrumbs = '';
    if (hit._highlightResult.headings) {
      breadcrumbs = hit._highlightResult.headings.map(match => {
        return `<span class="post-breadcrumb">${match.value}</span>`
      }).join(' > ')
    }

    let description = "" ;
    if (hit._highlightResult.description){
        description = hit._highlightResult.description.value;
    }
    else{
        description = hit.summary;
    }


    return `
      <div class="post-item">
        <h3><a class="post-link" href="${url}">${title}</a></h3>
        <a href="${url}" class="post-breadcrumbs">${breadcrumbs}</a>
        <div class="post-snippet">${description}</div>
        <span class="post-meta">${date}</span>
      </div>
    `;
  }


  search.addWidget(
    instantsearch.widgets.searchBox({
      container: '#search-searchbar',
      placeholder: 'Search into posts...',
      poweredBy: true // This is required if you're on the free Community plan
    })
  );

  search.addWidget(
    instantsearch.widgets.hits({
      container: '#search-hits',
      templates: {
        item: hitTemplate
      }
    })
  );

  search.start();
  </script>

  <style>
  .ais-search-box {
    max-width: 100%;
    margin-bottom: 15px;
  }
  .post-item {
    margin-bottom: 30px;
  }
  .post-link .ais-Highlight {
    color: #111;
    font-style: normal;
    text-decoration: underline;
  }
  .post-breadcrumbs {
    color: #424242;
    display: block;
  }
  .post-breadcrumb {
    font-size: 18px;
    color: #424242;
  }
  .post-breadcrumb .ais-Highlight {
    font-weight: bold;
    font-style: normal;
  }
  .post-snippet .ais-Highlight {
    color: #2a7ae2;
    font-style: normal;
    font-weight: bold;
  }
  .post-snippet img {
    display: none;
  }
  </style>

  ```

注意上面 JS 代码：
  ```javascript
  <script>
  const search = instantsearch({
    appId: '{{ .Site.Params.algolia_appId }}',
    indexName: '{{ .Site.Params.algolia_indexName }}',
    apiKey: '{{ .Site.Params.algolia_apiKey }}'
  });
  ```

> - `algolia_search`的第一个参数为是否开启索引
> - `algolia_appId`的第二个参数为Application ID `3DH4V2B4JK`
> - `algolia_indexName`为我们创建的索引名称`hugo`，
> - `algolia_apiKey`为我们的Admin API Key`31c446dxxxxxxxxxxxxxxxxxxxxxxxx`。


然后我们只需要添加一个搜索入口即可，在`themes/white/layouts/partials/nav.html`文件最下面添加如下代码：
  ```html
  <!-- Navigation -->
  <nav class="navbar navbar-default navbar-custom navbar-fixed-top">
      <div class="container-fluid">
          <!-- Brand and toggle get grouped for better mobile display -->
          <div class="navbar-header page-scroll">
              <button type="button" class="navbar-toggle">
                  <span class="sr-only">Toggle navigation</span>
                  <span class="icon-bar"></span>
                  <span class="icon-bar"></span>
                  <span class="icon-bar"></span>
              </button>
              <a class="navbar-brand" href="{{ "/" | relLangURL }}">{{ .Site.Title }}</a>
          </div>

          <!-- Collect the nav links, forms, and other content for toggling -->
          <!-- Known Issue, found by Hux:
              <nav>'s height woule be hold on by its content.
              so, when navbar scale out, the <nav> will cover tags.
              also mask any touch event of tags, unfortunately.
          -->
          <div id="huxblog_navbar">
              <div class="navbar-collapse">
                  <ul class="nav navbar-nav navbar-right">
                      <li>
                          <a href="{{ "/" | relLangURL }}">Home</a>
                      </li>
                      {{ range $name, $taxonomy := .Site.Taxonomies.categories }}
                      <li>
                          <a href="{{ "categories/" | relLangURL }}{{ $name | urlize }}">{{ $name }}</a>
                      </li>
                      {{ end }}

  		    {{ range .Site.Params.addtional_menus }}
                          <li><a href="{{.href | relLangURL}}">{{.title}}</a></li>
                      {{ end }}

                      {{ if .Site.Params.algolia_search }}
  		    <li>
                          <a href="{{ "search" | relURL }}">SEARCH <img src="{{ "img/search.png" | relURL }}" height="15" style="cursor: pointer;" alt="Search"></a>
  		    </li>
                      {{ end }}
                  </ul>
              </div>
          </div>
          <!-- /.navbar-collapse -->
      </div>
      <!-- /.container -->
  </nav>
  <script>
      // Drop Bootstarp low-performance Navbar
      // Use customize navbar with high-quality material design animation
      // in high-perf jank-free CSS3 implementation
      var $body   = document.body;
      var $toggle = document.querySelector('.navbar-toggle');
      var $navbar = document.querySelector('#huxblog_navbar');
      var $collapse = document.querySelector('.navbar-collapse');

      $toggle.addEventListener('click', handleMagic)
      function handleMagic(e){
          if ($navbar.className.indexOf('in') > 0) {
          // CLOSE
              $navbar.className = " ";
              // wait until animation end.
              setTimeout(function(){
                  // prevent frequently toggle
                  if($navbar.className.indexOf('in') < 0) {
                      $collapse.style.height = "0px"
                  }
              },400)
          }else{
          // OPEN
              $collapse.style.height = "auto"
              $navbar.className += " in";
          }
      }
  </script>

  ```
其中最重要的代码是引入上面我们新建的`search.html`文件。css引用的别人的文件
```html
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.6.0/dist/instantsearch.min.css">
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.6.0/dist/instantsearch-theme-algolia.min.css">

```

## 5.搜索

上面的所有工作完成后，我们重新生成静态页面，更新网站数据。我们可以看到导航栏最右边已经有了一个搜索按钮了。

![search demo](
https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/04/006.png)

## 6.参考资料

* [https://github.com/zhaohuabing/hugo-theme-cleanwhite/](https://github.com/zhaohuabing/hugo-theme-cleanwhite/)
* [https://gohugo.io/tools/search/](https://gohugo.io/tools/search/)
* [https://www.npmjs.com/package/hugo-algolia](https://www.npmjs.com/package/hugo-algolia)
