
[Cursor太贵？分享三个免费AI编程方案+海量编程技巧【如何看待AI编程】\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1b5AeeGEFc/?spm_id_from=333.337.search-card.all.click&vd_source=83485b71343f442522d28357f4bb93eb)




[Site Unreachable](https://zhuanlan.zhihu.com/p/16508727483)


### 1. `CTRL/CMD + L` 打开对话框


### . `@Files` 注记，传递指定代码文件的上下文

当你在对话框输入 `@Files` 注记时，Cursor 会自动弹出对你代码仓库的检索列表，你可以输入你想要导入上下文的文件名，而后按下确认键，相应的文件里的内容便会届时自动注入到上下文中




### 2. `@Code` 注记，传递指定代码块的上下文

`Code` 注记提供更精确的代码片段，`@` 注记的使用都大同小异，会弹出相应的检索框，你输入关键词后在索引列表中选择相应的代码块即可。

代码块的识别是由你开发环境的 LSP 决定的，大多数情况下都是精确的

### 3. `@Docs` 注记，从函数或库的官方文档里获取上下文

`@Docs` 注记能够从函数或库的官方文档里获取上下文。目前，它只能从可访问的在线文档里获取上下文。因此，你自己写的类似于 [JSDoc](https://zhida.zhihu.com/search?content_id=252323812&content_type=Article&match_order=1&q=JSDoc&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDUwMjUxMTIsInEiOiJKU0RvYyIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjI1MjMyMzgxMiwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.dGBxnvEFkMhWq_xwN_vHiHxSZ4pqMTysGNGqA11t84Q&zhida_source=entity) 之类的文档信息除非你能整一个线上地址，否则是没用的~我个人觉得这个功能不是很泛用。

### 4. `@Web` 注记，从搜索引擎的搜索内容获取上下文

`@Web` 注记类似于一种方法，它会默认将你的提问先向搜索引擎进行搜索，然后从搜索结果里提取上下文喂给 LLM。但因为 Cursor 官方没公开透明具体的实现法子，它自个也没调好，实际上使用效果忽好忽差的。

如果你遇到问题想偷懒不打开网页搜报错或是大模型自身的回答无法解决问题，你可以直接使用这个注记。


### 5. `@Folders` 注记，传递文件目录信息的上下文

`@Folders` 注记能够提供文件目录的相关信息，如果你遇到什么路径问题，可以考虑使用这个注记向大模型寻求解决方法。

### 6. `@Chat` 注记，只能在文件内的代码生成窗口里使用的注记

`@Chat` 注记只能在文件内的代码生成窗口（`CTRL + K` 打开的窗口）里使用，它能够将你右边打开的对话窗口里的对话内容作为上下文传递给大模型。


### 7. `@Definitions` 注记，只能在文件内的代码生成窗口里使用的注记

和 `@Chat` 注记一样，`@Definitions` 注记只能在文件内的代码生成窗口里使用。它会将你光标停留的那一行代码里涉及到的变量、类型的相关定义作为上下文传递给大模型，类似于 `@Code` 注记。


### 8. `@Git` 注记，只能在对话窗里使用

对话窗指的是通过 `CTRL + L` 与 `CTRL + I` 打开的对话窗口。`@Git` 注记能够将你当前的 [Git 仓库](https://zhida.zhihu.com/search?content_id=252323812&content_type=Article&match_order=1&q=Git+%E4%BB%93%E5%BA%93&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDUwMjUxMTIsInEiOiJHaXQg5LuT5bqTIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjUyMzIzODEyLCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.I-laAR5AOFPd3ndpallYaUNezcb7ohYvkhDFGb4bFJY&zhida_source=entity)的 commit 历史作为上下文传递给大模型。

感觉比较适合在代码协作的时候查战犯清算的时候使用。
















