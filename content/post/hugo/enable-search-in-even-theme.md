---
title: "给Hugo中的Even主题添加搜索功能"
date: 2023-11-30T10:35:48+08:00
lastmod: 2023-11-30T10:35:48+08:00
draft: false
keywords: ["hugo","search"]
description: "参考网络文档给Hugo中的Even主题添加搜索功能，以方便检索博客的内容"
tags: ["hugo"]
categories: ["个人博客"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

介绍如何在[**Hugo**](https://gohugo.io/)的[**Even**](https://github.com/olOwOlo/hugo-theme-even)主题中添加基于[**Fuse.js**](https://www.fusejs.io/)的搜索功能。

<!--more-->

  本文主要参考[**给hugo添加搜索功能**](https://sobaigu.com/hugo-set-featuer-search.html)实现并基于个人需求做了适当的改进，相关操作步骤如下：

## 输出索引文件

1. 在`config.toml`中添加如下内容，确保可输出`JSON`格式的数据

   ```toml
   [outputs]
     home = ["HTML", "RSS", "JSON"]
   ```

2. 在`layouts/_default`下面创建一个`index.json`并写入如下内容

   ```json
   {{- $.Scratch.Add "index" slice -}}
   {{- range .Site.RegularPages -}}
     {{- $.Scratch.Add "index" (dict "title" .Title "tags" .Params.tags "categories" .Params.categories "date" .Params.date "contents" .Plain "permalink" .Permalink) -}}
   {{- end -}}
   {{- $.Scratch.Get "index" | jsonify -}}
   ```

   此步骤执行完毕后可通过`http://IP:1313/index.json`查看输出的`JSON`文件，结果类似如下

   ![Hugo中输出index.json文件](/blog_img/hugo/enable-search-in-even-theme/hugo-index-json-output.png "Hugo中输出index.json文件")

   此步骤较为关键，后续的检索即是对`index.json`的内容进行检索，其中包含了`title`、`tag`等属性，作用如下

   |     属性     | 作用                                     |
   | :----------: | :--------------------------------------- |
   |   `title`    | 文章标题，关键字检索                     |
   |    `tags`    | 文章标签，用于关键字检索                 |
   | `categories` | 文章类别，用于关键字检索                 |
   |  `contents`  | 文章内容，用于关键字检索                 |
   | `permalink`  | 在搜索结果中打开对应的页面               |
   |    `date`    | 对搜索结果进行排序，新发布的排在前面[^1] |

## 创建索引页面

1. 在`content`目录下创建一个名为`search.md`的文件，需要添加`layout: "search"`配置来确保其内容是通过模板文件生成，同时在菜单部分添加如下配置，确保搜索功能能展示出来

   ```toml
   [[menu.main]]
   	name = "搜索"
   	weight = 50
   	identifier = "search"
   	url = "/search/"
   ```

2. 在`layouts/_default`下创建一个名为`search.html`的文件，并写入如下内容，确保前一个步骤中的模板文件引入能生效

   ```html
   {{ define "main" }}
   	<section>
   	  <div>
   		<form action="{{ "search" | absURL }}">
   		  <input id="search-query" name="s"/>
   		</form>
   		<div id="search-results">
   		 <h3 id="search-results-info">Matching pages</h3>
   		</div>
   	  </div>
   	</section>
   	<script id="search-result-template" type="text/x-js-template">
   	  <div id="summary-${key}" class="search_list">
   		<h4><a href="${link}">${title}</a></h4>
   		<p>${snippet}</p>
   		<small>
   			${ isset tags }<div><b>标签:</b> ${tags}</div>${ end }
   			${ isset categories }<div><b>类别:</b> ${categories}</div>${ end }
   			<span class="post-time"><b>时间:</b> ${date}</span>
   		</small>
   	  </div>
   	</script>
   {{ end }}
   ```

3. 在`layouts/partials/scripts.html`中添加如下代码，引入对应的`js`文件

   ```html
   <!-- search function -->
   {{ if eq (trim .Page.RelPermalink "/") "search"}}
   	<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
   	<script src="https://cdnjs.cloudflare.com/ajax/libs/fuse.js/3.2.0/fuse.min.js"></script>
   	<script src="https://cdnjs.cloudflare.com/ajax/libs/mark.js/8.11.1/jquery.mark.min.js"></script>
   	<script src="{{ "lib/search/search.js" | absURL }}"></script>
   {{- end }}
   ```

4. 在`static/lib/search`下创建一个名为`search.js`的文件，并写入如下内容，它是我们的搜索核心，至此整个搜索功能添加完毕

   ```javascript
   summaryInclude=60;
   var fuseOptions = {
     shouldSort: true,
     includeMatches: true,
     threshold: 0.0,
     tokenize:true,
     location: 0,
     distance: 100,
     maxPatternLength: 32,
     minMatchCharLength: 1,
     keys: [
       {name:"title",weight:0.8},
       {name:"contents",weight:0.5},
       {name:"tags",weight:0.3},
       {name:"categories",weight:0.3}
     ]
   };
   
   var searchQuery = param("s");
   if(searchQuery){
     $("#search-query").val(searchQuery);
     executeSearch(searchQuery);
   }else {
     $('#search-results').append("<p>请在上面输入一个词或词组</p>");
   }
   
   // sort by date
   function sortResult(a,b){
   	let time1 = new Date(b.item.date).getTime();
   	let time2 = new Date(a.item.date).getTime();
   	return time1 - time2;
   }
   
   function formatDate(date){
   	return date.split('T')[0];
   }
   
   function executeSearch(searchQuery){
     $.getJSON( "/index.json", function( data ) {
       var pages = data;
       var fuse = new Fuse(pages, fuseOptions);
       var result = fuse.search(searchQuery);
       if(result.length > 0){
   	  $('#search-results-info').html("共检索到" + result.length + "条记录").show();
   	  result.sort(sortResult);
         populateResults(result);
       }else{
         $('#search-results-info').html("没有搜索到结果!").show();
       }
     });
   }
   
   function populateResults(result){
     $.each(result,function(key,value){
       var contents= value.item.contents;
       var snippet = "";
       var snippetHighlights=[];
       var tags =[];
       if( fuseOptions.tokenize ){
         snippetHighlights.push(searchQuery);
       }else{
         $.each(value.matches,function(matchKey,mvalue){
           if(mvalue.key == "tags" || mvalue.key == "categories" ){
             snippetHighlights.push(mvalue.value);
           }else if(mvalue.key == "contents"){
             start = mvalue.indices[0][0]-summaryInclude>0?mvalue.indices[0][0]-summaryInclude:0;
             end = mvalue.indices[0][1]+summaryInclude<contents.length?mvalue.indices[0][1]+summaryInclude:contents.length;
             snippet += contents.substring(start,end);
             snippetHighlights.push(mvalue.value.substring(mvalue.indices[0][0],mvalue.indices[0][1]-mvalue.indices[0][0]+1));
           }
         });
       }
   
       if(snippet.length<1){
         snippet += contents.substring(0,summaryInclude*2);
       }
       //pull template from hugo templarte definition
       var templateDefinition = $('#search-result-template').html();
       //replace values
       var output = render(templateDefinition,{key:key,title:value.item.title,link:value.item.permalink,tags:value.item.tags,categories:value.item.categories,date:formatDate(value.item.date),snippet:snippet});
       $('#search-results').append(output);
   
       $.each(snippetHighlights,function(snipkey,snipvalue){
         $("#summary-"+key).mark(snipvalue);
       });
   
     });
   }
   
   function param(name) {
       return decodeURIComponent((location.search.split(name + '=')[1] || '').split('&')[0]).replace(/\+/g, ' ');
   }
   
   function render(templateString, data) {
     var conditionalMatches,conditionalPattern,copy;
     conditionalPattern = /\$\{\s*isset ([a-zA-Z]*) \s*\}(.*)\$\{\s*end\s*}/g;
     //since loop below depends on re.lastInxdex, we use a copy to capture any manipulations whilst inside the loop
     copy = templateString;
     while ((conditionalMatches = conditionalPattern.exec(templateString)) !== null) {
       if(data[conditionalMatches[1]]){
         //valid key, remove conditionals, leave contents.
         copy = copy.replace(conditionalMatches[0],conditionalMatches[2]);
       }else{
         //not valid, remove entire section
         copy = copy.replace(conditionalMatches[0],'');
       }
     }
     templateString = copy;
     //now any conditionals removed we can do simple substitution
     var key, find, re;
     for (key in data) {
       find = '\\$\\{\\s*' + key + '\\s*\\}';
       re = new RegExp(find, 'g');
       templateString = templateString.replace(re, data[key]);
     }
     return templateString;
   }
   ```

## 索引功能验证

1. 在页面右上角的一级菜单中有一个名为`搜索`的链接，点击进入后会展示类似如下界面

   ![Hugo中搜索界面](/blog_img/hugo/enable-search-in-even-theme/hugo-search-page.png "Hugo中搜索界面")

2. 在搜索框中输入对应的关键字并点击`Enter`键后，经过1-2s的等待，会出现类似如下的页面，展示相关的搜索结果

   ![Hugo中搜索结果展示界面](/blog_img/hugo/enable-search-in-even-theme/hugo-search-result.png "Hugo中搜索结果展示界面")

[^1]: 参考来源中的排序结果是无序的，不便于使用，故本文添加基于发布时间的排序，使得结果更直观。