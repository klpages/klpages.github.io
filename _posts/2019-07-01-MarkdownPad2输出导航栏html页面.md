---
layout:     post
title:      MarkdownPad2输出带有导航栏的html页面
subtitle:   
date:       2019-07-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Markdown
---
编写MD文档时为了方便查看有时需要输出带有导航栏的页面,废话不多说，下面是效果图：  
<img src="/img/article/md.jpg"/>

### 解决方案如下

- 设置HTML HEAD编辑器   
将如下代码配置到MarkdownPad 2里的工具-> 选项 ->高级 ->HTML HEAD编辑器 

``` javascript
<script src="http://rjgc.cn/jquery-3.3.1.min.js"></script>
<script type="text/javascript">
//是否显示导航栏
 var showNavBar = true;
 //是否展开导航栏
 var expandNavBar = true;

 $(document).ready(function(){
    var h1s = $("body").find("h1");
    var h2s = $("body").find("h2");
    var h3s = $("body").find("h3");
    var h4s = $("body").find("h4");
    var h5s = $("body").find("h5");
    var h6s = $("body").find("h6");

   var links = document.links;
   for (var i = 0; i < links.length; i++) {
    if (!links[i].target) {
        if (
            links[i].hostname !== window.location.hostname || 
            /\.(?!html?)([a-z]{0,3}|[a-zt]{0,4})$/.test(links[i].pathname)
        ) {
            links[i].target = '_blank';
            } 
        }
    }

    var headCounts = [h1s.length, h2s.length, h3s.length, h4s.length, h5s.length, h6s.length];
    var vH1Tag = null;
    var vH2Tag = null;
    for(var i = 0; i < headCounts.length; i++){
        if(headCounts[i] > 0){
            if(vH1Tag == null){
                vH1Tag = 'h' + (i + 1);
            }else{
                vH2Tag = 'h' + (i + 1);
            }
        }
    }
    if(vH1Tag == null){
        return;
    }

    $("body").prepend('<div class="BlogAnchor">' + 
        
        '<p class="html_header">' + 
            '<span></span>' + 
        '</p>' + 
        '<div class="AnchorContent" id="AnchorContent"> </div>' + 
    '</div>' );

var vH1Index = 0;
var vH2Index = 0;
var menuIndex = 0;
$("body").find("h1,h2,h3,h4,h5,h6").each(function(i,item){
var id = '';
// var name = '';
var tag = $(item).get(0).tagName.toLowerCase();
//不同标题,不同 className,不同字体大小
var className = '';
if (tag == vH1Tag) {
// id = name = ++vH1Index;
// name = id;
vH2Index = 0;
className = 'item_h1';
} else if (tag == vH2Tag) {
// id = vH1Index + '_' + ++vH2Index;
// name = vH1Index + '.' + vH2Index;
className = 'item_h2';
}
//修改 id 生成策略
id = "_" + menuIndex;
menuIndex++;

$(item).attr("id","wow"+id);
$(item).addClass("wow_head");
        $("#AnchorContent").css('max-height', ($(window).height() - 80) + 'px');
        $("#AnchorContent").append('<li><a class="nav_item '+className+' anchor-link" onclick="return false;" href="#" link="#wow'+id+'">'+""+""+$(this).text()+'</a></li>');
    });

    
    $(".anchor-link").click(function(){
        $("html,body").animate({scrollTop: $($(this).attr("link")).offset().top}, 500);
    });

    var headerNavs = $(".BlogAnchor li .nav_item");
    var headerTops = [];
    $(".wow_head").each(function(i, n){
        headerTops.push($(n).offset().top);
    });
    $(window).scroll(function(){
        var scrollTop = $(window).scrollTop();
        $.each(headerTops, function(i, n){
            var distance = n - scrollTop;
            if(distance >= 0){
                $(".BlogAnchor li .nav_item.current").removeClass('current');
                $(headerNavs[i]).addClass('current');
                return false;
            }
        });
    });

    if(!showNavBar){
        $('.BlogAnchor').hide();
    }
    if(!expandNavBar){
        $(this).html("目录▼");
        $(this).attr({"title":"展开"});
        $("#AnchorContent").hide();
    }
 });


 $(window).resize(function() {
  $("#AnchorContent").css('max-height', ($(window).height() - 80) + 'px');
});


//插入title的ico图标  
var ico_link = "<link rel=icon type=image/png sizes=32x32 href=html/ico.png>";
$("head").prepend(ico_link);
</script>
```


- 设置CSS  
将如下代码配置到MarkdownPad 2里的工具-> 选项->样式表->添加 


```css
/*markdown_box*/
    html{background: #FFFFFF; font-family:"Microsoft YaHei", "微软雅黑", Arial; font-weight: 400}
    body{background:#FFFFFF;font-size: 16px; color: #515254; line-height: 188%; margin:0 5% 0 17%; padding: 0 !important;}
    .html_header{display:block; height: 60px; line-height: 46px; border-bottom: 1px solid #E6ECF1; text-align: left !important;}
    .html_header span::after{content: "Title"; font-size: 17px; font-weight: bold;}
    .html_header span::before{content: "❖"; font-size: 19px; font-weight: bold; margin:0 8px 0 5px; color: #1284d9}

   
    img{border-radius:5px;margin:30px 0 46px !important;width:100%;max-width: 1100px; opacity: .6; transition: opacity .3s}
    img:hover{opacity: 1}


    .BlogAnchor {width: 210px !important}

    @media screen and (max-width: 1620px) {
        .BlogAnchor {width: 210px !important}
    }

    @media screen and (max-width: 1370px) {
        .BlogAnchor {width: 200px !important}
    }


    @media screen and (max-width: 820px) {
        .BlogAnchor {width: 180px !important}
        .html_header span::before{ margin-left: 0}
        .html_header span::after{ font-size: 15px;}
        h2{ font-size: 30px; }
        h3{ font-size: 24px; }
    }

    @media screen and (max-width: 640px) {
        .BlogAnchor {display: none;}
        h2{ font-size: 28px; }
        h3{ font-size: 22px; }
    }


/*scroll_style*/
.AnchorContent {
    overflow-y: auto !important;
    overflow-x: hidden !important;
    margin-right:0 !important
}

.AnchorContent::-webkit-scrollbar {
    width: 6px;
    height: 10px;
    background-color: rgba(0,0,0,0) !important;
    position: absolute;
    right: 10px
}
.AnchorContent::-webkit-scrollbar-track {
    -webkit-box-shadow: none;
    border-radius: 0;
    background-color: rgba(0,0,0,0) !important
}
.AnchorContent::-webkit-scrollbar-thumb {
    border-radius: 0px;
    -webkit-box-shadow: none;
    background-color: none;
    opacity: 0 !important;
    transition: all .3s
}

.AnchorContent:hover::-webkit-scrollbar-thumb {
    border-radius: 0px;
    -webkit-box-shadow: none;
    background-color: #d4d6d8;
    border-right:2px solid #fff;
    opacity:1;
}

.AnchorContent::-webkit-scrollbar-thumb:hover {
    background-color: #c0c2c4 !important
}


/*导航*/
    .BlogAnchor {
        background: #fff;
        padding: 10px 0 ;
        line-height: 180%;
        position: fixed;
        height: 100%;
        width: 200px;
        left: 0;
        top: 0;
        border-right: 1px solid #E6ECF1;

    }
    .BlogAnchor p {
        font-size: 20px;
        color: #444;
        margin: 0 16px;
        text-align: center;
    }
    .BlogAnchor .AnchorContent {
        padding: 5px 0px;
        overflow: auto;
    }
    .BlogAnchor li{
        text-indent: 1.5rem;
        font-size: 14px;
        list-style: none;
        padding: 0 0px;
    }
    .BlogAnchor li .nav_item{
        padding:3px 16px;

    }


    .BlogAnchor li .item_h1{
        margin-left: 0rem;
        padding-left:3px;
        font-weight: bold;
    }
    .BlogAnchor li .item_h2{
        margin-left: 0;
        font-size: 0.8rem;
        padding-left: 18px;
        color: #708090
    }
    .BlogAnchor li .nav_item.current{
        color: #087cd2;
        background-color: rgba(238,242,249,.8);
        border-left:4px solid #087cd2;
        text-indent: 20px;
        font-weight: bold;

    }
    #AnchorContentToggle {
        font-size: 13px;
        font-weight: normal;
        color: #FFF;
        display: inline-block;
        line-height: 20px;
        background: rgb(7,196,189);
        font-style: normal;
        padding: 1px 8px;
    }
    .BlogAnchor a:hover {
        color: #087cd2  ;
        background:rgba(238,242,249,.5);
    }
    .BlogAnchor a {
        text-decoration: none;
        color: #333;
        display: block;
        border-radius: 0px;
    }
    
/* BLOCKS
=============================================================================*/

p, blockquote, ul, ol, dl, table, pre {
  margin: 15px 0;
}

/* HEADERS
=============================================================================*/

h1, h2, h3, h4, h5, h6 {
  margin: 20px 0 10px;
  padding: 0;
  font-weight: bold;
  -webkit-font-smoothing: antialiased;
}

h1 tt, h1 code, h2 tt, h2 code, h3 tt, h3 code, h4 tt, h4 code, h5 tt, h5 code, h6 tt, h6 code {
  font-size: inherit;
}

h1 {
  font-size: 28px;
  color: #000;
}

h2 {
  font-size: 24px;
  border-bottom: 1px solid #ccc;
  color: #000;
}

h3 {
  font-size: 18px;
}

h4 {
  font-size: 16px;
}

h5 {
  font-size: 14px;
}

h6 {
  color: #777;
  font-size: 14px;
}

body>h2:first-child, body>h1:first-child, body>h1:first-child+h2, body>h3:first-child, body>h4:first-child, body>h5:first-child, body>h6:first-child {
  margin-top: 0;
  padding-top: 0;
}

a:first-child h1, a:first-child h2, a:first-child h3, a:first-child h4, a:first-child h5, a:first-child h6 {
  margin-top: 0;
  padding-top: 0;
}

h1+p, h2+p, h3+p, h4+p, h5+p, h6+p {
  margin-top: 10px;
}

/* LINKS
=============================================================================*/

a {
  color: #4183C4;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

/* LISTS
=============================================================================*/

ul, ol {
  padding-left: 30px;
}

ul li > :first-child, 
ol li > :first-child, 
ul li ul:first-of-type, 
ol li ol:first-of-type, 
ul li ol:first-of-type, 
ol li ul:first-of-type {
  margin-top: 0px;
}

ul ul, ul ol, ol ol, ol ul {
  margin-bottom: 0;
}

dl {
  padding: 0;
}

dl dt {
  font-size: 14px;
  font-weight: bold;
  font-style: italic;
  padding: 0;
  margin: 15px 0 5px;
}

dl dt:first-child {
  padding: 0;
}

dl dt>:first-child {
  margin-top: 0px;
}

dl dt>:last-child {
  margin-bottom: 0px;
}

dl dd {
  margin: 0 0 15px;
  padding: 0 15px;
}

dl dd>:first-child {
  margin-top: 0px;
}

dl dd>:last-child {
  margin-bottom: 0px;
}

/* CODE
=============================================================================*/

pre, code, tt {
  font-size: 12px;
  font-family: Consolas, "Liberation Mono", Courier, monospace;
  color:#F66
}

code, tt {
  margin: 0 0px;
  padding: 0px 0px;
  white-space: nowrap;
  border: 1px solid #eaeaea;
  background-color: #f8f8f8;
  border-radius: 3px;
}

pre>code {
  margin: 0;
  padding: 0;
  white-space: pre;
  border: none;
  background: transparent;
}

pre {
  background-color: #f8f8f8;
  border: 1px solid #ccc;
  font-size: 13px;
  line-height: 19px;
  overflow: auto;
  padding: 6px 10px;
  border-radius: 3px;
}

pre code, pre tt {
  background-color: transparent;
  border: none;
}

kbd {
    -moz-border-bottom-colors: none;
    -moz-border-left-colors: none;
    -moz-border-right-colors: none;
    -moz-border-top-colors: none;
    background-color: #DDDDDD;
    background-image: linear-gradient(#F1F1F1, #DDDDDD);
    background-repeat: repeat-x;
    border-color: #DDDDDD #CCCCCC #CCCCCC #DDDDDD;
    border-image: none;
    border-radius: 2px 2px 2px 2px;
    border-style: solid;
    border-width: 1px;
    font-family: "Helvetica Neue",Helvetica,Arial,sans-serif;
    line-height: 10px;
    padding: 1px 4px;
}

/* QUOTES
=============================================================================*/

blockquote{border-radius:0 4px 4px 0;color:#708090;border-left:3px solid #1284d9; background:rgba(238,242,249,.7); padding:8px 16px; font-size: 15px;}

blockquote>:first-child {
  margin-top: 0px;
}

blockquote>:last-child {
  margin-bottom: 0px;
}

/* HORIZONTAL RULES
=============================================================================*/

hr {
  clear: both;
  margin: 15px 0;
  height: 0px;
  overflow: hidden;
  border: none;
  background: transparent;
  border-bottom: 4px solid #ddd;
  padding: 0;
}
/* TABLE
=============================================================================*/
table {
  font-family: "Trebuchet MS", Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

table td, table th {
  border: 1px solid #ddd;
  padding: 8px;
}

table tr:nth-child(even){background-color: #f2f2f2;}

table tr:hover {background-color: #ddd;}

table th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #6495ED;
  color: white;
}
```

好了，现在你可以导出或者F6预览页面了。  
PS:本代码是基于朋友提供源码的基础上修改,在此感谢朋友的帮助，你懂的！
