# AjaxSearch
Servlet+Ajax实现搜索框智能提示
## 简介
搜索框相信大家都不陌生，几乎每天都会在各类网站进行着搜索。有没有注意到，很多的搜索功能，当输入内容时，下面会出现提示。这类提示就叫做搜索框的智能提示，本次就为大家介绍如何使用Servlet和Ajax来实现。主要介绍实现原理和代码的前后台实现过程。

## 项目分析

[scode type="green"]
实现语言：java
实现方式：Ajax异步传输
[/scode]

案例：比如百度的搜索框智能提示

![ajax](http://www.geekerstar.com/usr/uploads/2018/04/2611403624.png)

理论分析：
1、在搜索框输入关键字。
2、浏览器将关键字`异步`发送给服务器。
3、服务器经过处理，将相应的数据以JSon（xml）格式返回给客户端。
4、客户端接收到服务器的响应数据，解析之后使用JS操作DOM显示数据。

演示流程图分析：

![过程分析](http://www.geekerstar.com/usr/uploads/2018/04/2533403524.png)

[scode type="yellow"]
重点内容：
1、数据交互采用ajax方式。
2、JavaScript解析数据动态展示。
[/scode]


## 页面开发
### HTML部分
>body内容

```html
<body>
<div id="mydiv">
    <!--输入框-->
    <input type="text" size="50" id="keyword" onkeyup="getMoreContents()"
           onblur="keywordBlur()" onfocus="getMoreContents()"/>
    <input type="button" value="百度一下" width="50px">
    <!--内容展示区域-->
    <div id="popdiv">
        <table id="content_table" bgcolor="#FFFAFA" border="0" cellspacing="0" cellpadding="0">
            <tbody id="content_table_body">
            <%--动态查询出来的数据,显示在此--%>

            </tbody>
        </table>

    </div>

</div>

</body>
```

### CSS部分

>样式代码

```css
<style type="text/css">
        #mydiv {
            position: absolute;
            left: 50%;
            top: 50%;
            margin-left: -200px;
            margin-top: -50px;
        }

        .mouseOver {
            background: #708090;
            color: #FFFAFA;
        }

        .mouseOut {
            background: #FFFAFA;
            color: #000000;
        }
    </style>
```

### JavaScript部分
> 获取用户输入内容的关联信息的函数

```javascript
function getMoreContents() {
            //首先获取用户的输入
            var content = document.getElementById("keyword");
            if (content.value == "") {
                clearContent();
                return;
            }
            //然后给服务器发送用户输入的内容,因为采用ajax异步发送数据,
            //所以使用XmlHttp对象
            xmlHttp = creatXMLHttp();
            //给服务器发送数据
            var url = "search?keyword=" + escape(content.value);
            xmlHttp.open("GET", url, true);
            //xmlHttp绑定回调方法，这个回调方法会在xmlHttp状态改变的时候被调用
            //xmlHttp 状态0-4,我们只关心4(complete)，完成后再调用回调方法才有意义
            xmlHttp.onreadystatechange = callback;
            xmlHttp.send(null);
}
```

> 获取XmlHttp对象

```javascript
function creatXMLHttp() {
            //对于大多数浏览器都适用
            var xmlHttp;
            if (window.XMLHttpRequest) {
                xmlHttp = new XMLHttpRequest();
            }
            //要考虑浏览器的兼容性
            if (window.ActiveXObject) {
                xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
                if (!xmlHttp) {
                    xmlHttp = new ActiveXObject("Msxml2.XMLHTTP");
                }
            }
            return xmlHttp;
}
```

> 回调函数

```javascript
function callback() {
            //4代表成功
            if (xmlHttp.readyState == 4) {
                //200代表服务器响应成功
                if (xmlHttp.status == 200) {
                    //交互成功 获得响应的数据 是文本格式
                    var result = xmlHttp.responseText;
                    //解析数据
                    var json = eval("(" + result + ")");
                    //获取数据后动态显示 展示输入框下面
                    setContent(json);

                }
            }
}
```

> 设置关联数据展示

```javascript
function setContent(contents) {
            clearContent();
            setLocation();
            //获取关联数据的长度,以此来确定生成的<tr>
            var size = contents.length;
            //设置内容
            for (var i = 0; i < size; i++) {
                var nextNode = contents[i];//代表的是JSon数据的第i个元素
                var tr = document.createElement("tr");
                var td = document.createElement("td");
                td.setAttribute("border", "0");
                td.setAttribute("bgcolor", "#FFFAFA");
                td.onmouseover = function () {
                    this.className = 'mouseOver';
                };
                td.onmouseout = function () {
                    this.className = 'mouseOut';
                };
                td.onmousedown=function(){
                    //当鼠标点击一个关联数据的时候,被选中的数据 自动填充到输入框里面
                    document.getElementById("keyword").value=this.innerHTML;
                    //清除div边框
                    document.getElementById("popDiv").style.border="none";

                };
                var text = document.createTextNode(nextNode);
                td.appendChild(text);
                tr.appendChild(td);
                document.getElementById("content_table_body").appendChild(tr);
            }


}
```
### 前后台程序联调

> 清空之前的数据

```javascript
function clearContent() {
            var contentTableBody = document.getElementById("content_table_body");
            var size = contentTableBody.childNodes.length;
            for (var i = size - 1; i >= 0; i--) {
                contentTableBody.removeChild(contentTableBody.childNodes[i]);
            }
            document.getElementById("popdiv").style.border = "none";
}
```

> 输入框失去焦点 清空

```javascript
function keywordBlur() {
     clearContent();
}
```

> 设置显示关联信息

```javascript
function setLocation() {
            //关联信息的显示位置
            var content = document.getElementById("keyword");
            var width = content.offsetWidth;//输入框的宽度
            var left = content["offsetLeft"];//距离左边框的距离
            var top = content["offsetTop"] + content.offsetHeight;//距离顶部
            //获取显示数据div
            var popdiv = document.getElementById("popdiv");
            popdiv.style.border = "black 1px solid";
            popdiv.style.left = left + "px";
            popdiv.style.top = top + "px";
            popdiv.style.width = width + "px";
            document.getElementById("content_table").style.width = width + "px";

}
```

### 编写SearchServlet.java
```java

import net.sf.json.JSONArray;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.*;


public class SearchServlet extends HttpServlet {
    static List<String> datas = new ArrayList<String>();
    //模拟数据
    static {

        datas.add("ajax");
        datas.add("ajax post");
        datas.add("becky");
        datas.add("bill");
        datas.add("james");
        datas.add("jerry");
        datas.add("hao");

    }

    @Override
    protected  void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException,IOException{
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");
        System.out.print("123");
        //获取客户端数据
        String keyword = request.getParameter("keyword");
        //获取关键字
        List<String> listData = getData(keyword);
        //返回json格式
        response.getWriter().write(JSONArray.fromObject(listData).toString());


    }
    public List<String> getData(String keyword){
        List<String> list = new ArrayList<String>();
        for (String data:datas) {
            if(data.contains(keyword)){
                list.add(data);
            }
        }
        return  list;
    }

}

```

## 最终效果图
输入搜索信息下面会智能提示，点击提示信息会自动输入到搜索框中：

![智能提示图](http://www.geekerstar.com/usr/uploads/2018/04/2683632906.png)


## 项目地址

[geekerstar/AjaxSearch](https://github.com/geekerstar/AjaxSearch)

------------
[scode type="green"]如果您发现了文章有任何错误欢迎指正，有任何意见或建议，或者有疑问需要我提供帮助，也欢迎在下面留言，只需输入`昵称`+`邮箱`即可，`网站或博客`可选填。对于所有留言内容我会及时回复，非常期待与大家的交流！[/scode]


> 版权声明：本文（除特殊标注外）为原创文章，版权归 [Geekerstar](http://www.geekerstar.com) 所有。

> 本文链接：http://www.geekerstar.com/project/620.html

> 除了有特殊标注文章外欢迎转载，但请务必标明出处，格式如上，谢谢合作。

> 本作品采用 [知识共享署名-非商业性使用-相同方式共享 3.0 中国大陆许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 进行许可。
