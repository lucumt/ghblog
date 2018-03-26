+++
author = "飞狐"
categories = ["Web编程"]
tags = ["JavaScript","EasyUI"]
date = "2017-08-06T00:06:36+08:00"
description = "Blog of Rosen Lu"
keywords = ["JavaScript","EasyUI"]
title = "解决EasyUI DataGrid中的行在列冻结时无法对齐的问题"

+++

相对于传统的用HTML中TABLE实现的表格，利用 **[EasyUI](https://www.jeasyui.com)** 中的 **[DataGrid](https://www.jeasyui.com/demo/main/index.php?plugin=DataGrid)** 实现的表格具有很多优点，如可以 *对列宽进行拖动调整、列冻结、行冻结、自定义格式化* 等功能，故而在Web开发中得到了广泛的应用。最近自己在使用DataGrid的列冻结功能时遇到了由于某些单元格中的内容较多导致该行无法对齐的问题，由于当前在EasyUI官网中无法找到该问题的解决方案，自己研究DataGrid的实现原理后，找到了变通的解决方案，故先记录下。

<!--more-->
## EasyUI DataGrid正常情况下的列冻结
下图为一个常见的DataGrid使用示例，该图展示了我们在使用EasyUI DataGrid时经常会遇到的一个问题：**由于某个列的长度很长导致表格出现滚动条**  
!["没有冻结列时正常显示的DataGrid"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-no-frozen-column1.png "没有冻结列时正常显示的DataGrid")  
对应的代码为  
```html
<thead>
   <tr>
     <th data-options="field:'name'"><b>书名</b></th>
     <th data-options="field:'price',align:'center',width:70"><b>价格</b></th>
     <th data-options="field:'pubdate',align:'center',width:90"><b>出版日期</b></th>
     <th data-options="field:'description',width:800"><b>简要介绍</b></th>
   </tr>
</thead>
``` 
得益于EasyUI DataGrid强大的功能，当表格中列的宽度太长时，它会自动加上横向滚动条，避免像传统的HTML TABLE表格在内容过多时会自己挤成一团，通过拖动滚动条，我们可以很方便的查看表格中各列的内容。

但有时候我们会运到另外一个问题，拖动滚动条时前面的某些列就不见了，如本例中的书名，在某些情况下可能会对我们的使用带来不便。  
!["拖动滚动条时无法看见前面的列"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-no-frozen-column2.png "拖动滚动条时无法看见前面的列")    
此时DataGrid的列冻结功能就可以派上用场了，只需要将需要固定的列冻结即可，在本例中我想把书名列冻结，在需要修改代码如下  
  
```html
 <thead frozen="true">
   <tr>
     <th data-options="field:'name'"><b>书名</b></th>
   </tr>
 </thead>
 <thead>
   <tr>
     <th data-options="field:'price',align:'center',width:70"><b>价格</b></th>
     <th data-options="field:'pubdate',align:'center',width:90"><b>出版日期</b></th>
     <th data-options="field:'description',width:800"><b>简要介绍</b></th>
   </tr>
 </thead>
``` 
其对应的运行效果如下：  
!["冻结列之后的DataGrid显示效果"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-frozen-column1.png "冻结列之后的DataGrid显示效果")

## EasyUI DataGrid非正常情形下的列冻结效果  
大多数情况下这种列冻结都能满足我们的需求，但上述冻结列正常显示有一个前提：**表格中每一行各列的高度一致**，若表格中某些行中存在列高超出DataGrid正常高度的情形(25px)，在进行列冻结时就会出现冻结行和非冻结行无法对齐的问题。

下图为一个列超出正常高度的DataGrid显示效果，从图中可以看出由于描述信息中的文字较多，导致正常的高度都比DataGrid默认的高度要很多，直观的显示就是不同行的高度不一致。    
!["列高超出正常高度时的DataGrid"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-no-frozen-column3.png "列高超出正常高度时的DataGrid")  
此时若对该表格进列冻结，同样会出现出现如前所述的冻结效果，但 **如果我们在一开始就将描述信息这列隐藏掉，之后通过点击等方式让该列显示，则会出现冻结行和非冻结行无法对齐的情况**，如下图所示     
!["列高超出正常高度时的DataGrid"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-frozen-column-not-align-row.png "列高超出正常高度时的DataGrid")   
对应的关键代码如下，也可以点击[jsfiddle.net/wch9rnr2/8/](https://jsfiddle.net/wch9rnr2/8/)查看完整的代码
```javascript
<button type="button" onclick="toggleDescription()">隐藏或显示描述信息</button>
function toggleDescription(){
	var colOptions = $("#dg").datagrid("getColumnOption","description");
	var isHidden = !!colOptions.hidden;
  if(isHidden){
     $("#dg").datagrid("showColumn","description");
  }else{
     $("#dg").datagrid("hideColumn","description");
  }
}
``` 
从上图可以看出此时书名列和其余列已经无法对齐，严重印象了我们的使用效果。

## 解决方案

要想解决该问题，首先需要找出该问题产生的根源，利用Chrome或其它浏览器调试工具可知，当有部分列冻结时，DataGrid表格被分成了两个不同的表格，一个是冻结的表格，一个是未冻结的表格。进一步分析发现导致无法对齐的问题根源为 **由于两个表格中同一行的高度不同，导致实际显示时看起来表格行没有对齐**。  
!["DataGrid列冻结时的源代码"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-frozen-source-element.png "DataGrid列冻结时的源代码")    
找到问题根源后，要解决该问题，我们只需要让表格中每一行高度保持一致即可，具体来说就是 **要在EasyUI DataGrid渲染表格之前将同一行的每一个单元格的高度保持一致即可**。问题转化为寻找适当的方法切入点来修改单元格高度，即对单元格的高度进行校验。

为了修正单元格的高度，我们需要首先找出特定行已冻结单元格和未冻结单元格的高度，而要找出它们的高度必须要等EasyUI DataGrid渲染完毕之后才能获取到其实际高度。本列中由于点击按钮时会对最后一列进行隐藏或显示，所以我们可以在`toggleDescription`方法的最后执行相应的校验逻辑。

进一步调试分析后发现，对于DataGrid中的某一个数据行而言，冻结列的单元格高度都一样，非冻结列的单元格高度也一样，**为了保持整行对齐，我们只需要高度较小的单元格的高度设置为高度较高的单元格的高度即可。考虑到有些影响列，同时为了简化实现，可以将及解决方案从设置单元格的高度更改为设置特定行在冻结和非冻结部分的行高即可**。至于如何设置EasyUI DataGrid中数据行的高度，请参见EasyUI作者**stworthy**大神在[datagrid dynamically set / reset row height](http://www.jeasyui.com/forum/index.php?topic=4951.0)中的回复，在该问题中，**stworthy**给出的解决方案如下：
```javascript
var dg = $('#dg');
dg.datagrid('options').rowHeight = 40;
for(var i=0; i<dg.datagrid('getRows').length; i++){
    dg.datagrid('refreshRow', i);
}
```
基于此，我们可以将`toggleDescription`方法改进如下，完整的代码请参见[jsfiddle.net/wch9rnr2/9/](https://jsfiddle.net/wch9rnr2/9/)
```
function toggleDescription(){
  var dg = $("#dg");
	var colOptions = dg.datagrid("getColumnOption","description");
	var isHidden = !!colOptions.hidden;
  if(isHidden){
     dg.datagrid("showColumn","description");
  }else{
     dg.datagrid("hideColumn","description");
  }
 
 var dgOptions = dg.datagrid("options");
 var rows = dg.datagrid("getRows");
 var row = null;
 var tr = null;
 var height1 =0;
 var height2 =0;
 for(var i in rows){
    row = rows[i];
	  tr = dgOptions.finder.getTr(dg[0],i);
	  height1 = $(tr[0]).height();//冻结行的高度
	  height2 = $(tr[1]).height();//非冻结行的高度
	  if((isHidden&&height2>height1)||(!isHidden&&height1>height2)){
         //冻结部分在显示时的高度取较大的那个
         $(tr[0]).css("height",height2+"px");
	  }
	}
}
```
此时当我们点击按钮来显示描述信息时，DataGrid中的每一行都已经对齐，如下图所示，至此问题获得解决！   
!["重新校验后的DataGrid冻结列显示效果"](/blog_img/easyui-datagrid-row-not-align-when-column-frozen/datagrid-frozen-column2.png "重新校验后的DataGrid冻结列显示效果")

在实际使用中，我发现有时候上述JavaScript校验代码还是不能正常工作，其原因为在执行校验代码时，EasyUI DataGrid还没有完全渲染完毕，此时可以利用**[setTimeout](https://www.w3schools.com/JSREF/met_win_setTimeout.asp)**函数来延后校验代码的执行，修改后的代码如下：  
```javascript
function toggleDescription(){
  var dg = $("#dg");
	var colOptions = dg.datagrid("getColumnOption","description");
	var isHidden = !!colOptions.hidden;
  if(isHidden){
     dg.datagrid("showColumn","description");
  }else{
     dg.datagrid("hideColumn","description");
  }
 
 var dgOptions = dg.datagrid("options");
 var rows = dg.datagrid("getRows");
 var row = null;
 var tr = null;
 var height1 =0;
 var height2 =0;
 setTimeout(function(){
   for(var i in rows){
      row = rows[i];
      tr = dgOptions.finder.getTr(dg[0],i);
      height1 = $(tr[0]).height();//冻结行的高度
      height2 = $(tr[1]).height();//非冻结行的高度
      if((isHidden&&height2>height1)||(!isHidden&&height1>height2)){
           //冻结部分在显示时的高度取较大的那个
           $(tr[0]).css("height",height2+"px");
      }
    } 
 },1000);//延后一秒执行
}
```
考虑到实际使用中EasyUI DataGrid的渲染时间无法确定，用**[setTimeout](https://www.w3schools.com/JSREF/met_win_setTimeout.asp)**并非最优解，希望EasyUI官方后续能为该问题提供更合理的解决方案!