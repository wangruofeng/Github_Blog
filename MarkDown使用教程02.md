## 图片
Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式： 行内式和参考式。

![Alt text](/path/to/img.jpg)

![Alt text](/path/to/img.jpg "Optional title")

> 详细叙述如下：
> 一个惊叹号 !
> 接着一个方括号，里面放上图片的替代文字
> 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上 选择性的 'title' 文字

参考式的图片语法则长得像这样：

![Alt text][id]

「id」是图片参考的名称，图片参考的定义方式则和连结参考一样：

[id]: url/to/image  "Optional title attribute"

## 其他
### 自动链接
<http://example.com/>

<address@example.com>

## 反斜杠
Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：

\   反斜线
`   反引号
*   星号
_   底线
{}  花括号
[]  方括号
()  括弧
#   井字号
+   加号
-   减号
.   英文句点
!   惊叹号

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>