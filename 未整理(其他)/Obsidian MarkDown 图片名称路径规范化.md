![](https://zhuanlan.zhihu.com/p/30892308149)  

[](https://www.zhihu.com/)

- [首页](https://www.zhihu.com/)
- [
    
    知乎直答
    
    焕新
    
    
    
    ](https://www.zhihu.com/zhida)
- [知乎知学堂](https://www.zhihu.com/education/learning)
- [等你来答](https://www.zhihu.com/question/waiting)

​

提问

​

80

消息

​

99+

私信

[​

创作中心

](https://www.zhihu.com/creator)

![点击打开杨明源的主页](https://pic1.zhimg.com/v2-6c7a496a521ba6ce943bcfbba72b643b_l.jpg?source=32738c0c&needBackground=1)

[](https://www.zhihu.com/)

Obsidian MarkDown 图片名称路径规范化

首发于[Makrdown 笔记经验分享](https://www.zhihu.com/column/c_1880284673852289867)

![点击打开杨明源的主页](https://pic1.zhimg.com/v2-6c7a496a521ba6ce943bcfbba72b643b_l.jpg?source=32738c0c&needBackground=1)

Failed to fetch

![Obsidian MarkDown 图片名称路径规范化](https://picx.zhimg.com/70/v2-311d92db26e74a3c4e854f0b169b43eb_1440w.image?source=172ae18b&biz_tag=Post)

# Obsidian MarkDown 图片名称路径规范化

[![字符序列](https://picx.zhimg.com/v2-b8cfe2713504259a46e6c4b5d82ae701_l.jpg?source=32738c0c&needBackground=1)](https://www.zhihu.com/people/zi-fu-xu-lie-7)

[字符序列](https://www.zhihu.com/people/zi-fu-xu-lie-7)

AFM交流群868224642，一起抱团取暖！

​关注

[

收录于 · Makrdown 笔记经验分享

](https://www.zhihu.com/column/c_1880284673852289867)

14 人赞同了该文章

​

目录

收起

目录

引言：Obsidian 图片管理存在两个问题

1. 无法自由的定义图片名称和路径

2. wiki 链接不具有普适性，用别的笔记软件不能正常显示图片

一、两个问题的解决思路

1. 命名和路径问题：怎么统一规范，路径怎么设置？

2. 普适性问题：图片链接转换前后对比

二、相关插件设置

⭐Custom Attachment location 解决问题 1

⭐Consistent Attachments and Links 解决问题 2

三、单个笔记 图片整理流程

四、批量笔记 图片整理流程

其他插件

Clearing Unused Images

Image Converter

Find orphaned files and broken links

paste image rename

其他链接

什么是相对路径？

## 目录

- 引言：Obsidian 图片管理存在两个问题

- 1. 无法自由的定义图片名称和路径
- 2. wiki 链接不具有普适性，用别的笔记软件不能正常显示图片

- 一、两个问题的解决思路

- 1. 命名和路径问题：怎么统一规范，路径怎么设置？
- 2. 普适性问题：图片链接转换前后对比

- 二、相关插件设置

- ⭐Custom Attachment location 解决问题 1
- ⭐Consistent Attachments and Links 解决问题 2

- 三、单个笔记 图片整理流程
- 四、批量笔记 图片整理流程
- 其他插件

- Clearing Unused Images
- Image Converter
- Find orphaned files and broken links
- paste image rename

- 其他链接

- 什么是相对路径？

## 引言：Obsidian 图片管理存在两个问题

### 1. 无法自由的定义图片名称和路径

Obsidian 默认附件存放在同一个文件夹里，这意味着你的库会逐渐变成一个整体，时间越长越难以将部分笔记分离出来。

比如，本科、硕士毕业了，或者某一项长期工作完成了，想把从整个库中把一部分相关笔记备份起来，比如 “硕士期间”，你可以直接将这些笔记 markdown 文件剪切走，但是对应的图片怎么办呢？整个库的图片都是混在一起的。

### 2. wiki 链接不具有普适性，用别的笔记软件不能正常显示图片

Obsidian 默认的图片链接是wiki链接，`![[比如这样]]`，这种格式用别的软件是打不开的。

这两点问题让我难受了许久，一直不敢在markdown笔记中添加图片，生怕污染了笔记库，经过多次尝试，终于找到了合适的解决方案。

这篇笔记将介绍两个插件来解决这两个痛点问题，实现图片自由！！

## 一、两个问题的解决思路

### 1. 命名和路径问题：怎么统一规范，路径怎么设置？

图片命名和存放路径应该统一有规律

**图片位置与笔记位置相对一致，且随笔记的位置改变而改变。**  
**图片名称与笔记名称相对一致，且随笔记的名称改变而改变。**

常见的图片文件夹有两种

1. 图片统一放在库里的某个文件夹下  
    这种方式清晰明了，但是如果想导出或归档部分笔记，很难从附件库中筛选出对应的图片。  
    
2. 图片跟随笔记放在相对路径的对应文件夹下 `./assets/${filename}`  
    

![](https://pic2.zhimg.com/v2-4c967779d01a995c5f62ee6545c344ef_1440w.jpg)

示例

这是一个例子，test 1 和 test 2 是两篇不同的笔记，笔记和assets是平级关系，笔记的图片分别在 assets 文件夹下的test 1 和 test 2 内。不仅文件夹命名有规律，图片命名也有规律。  
这种方式乍一看会稍微有些乱，但是每个文件夹下的图片都是和笔记对应好的，归档或导出时不需要大动干戈。

### 2. 普适性问题：图片链接转换前后对比

**笔记里的图片要用别的软件也能正常打开，不要在格式上对 obsidian 有依赖性。**（文档内应该是最基础的 markdown 相对路径链接，而不是wiki链接）

这是用 Consistent Attachments and Links 转换 wiki 链接前后的对比。先给大家看一下效果图，我们的目标就是让笔记的格式更有普适性，不依赖某个特定的软件。

我们可以对比一下obsidian wiki 链接和标准markdown相对路径链接

![](https://picx.zhimg.com/v2-21d7ee5390273e234378e68b92ec3089_1440w.jpg)

obsidian wiki 链接和标准markdown相对路径链接

---

在obsidian中都可以正常打开，如下图所示：

![](https://pic4.zhimg.com/v2-e20b42674c26506fb5187e98004baf21_1440w.jpg)

obsidian 打开

---

如果用 MarkText 打开这个笔记，可以看到 wiki 链接是无法正常加载的，如下图所示：

![](https://pic3.zhimg.com/v2-77d34e1cbed6e913ac973682a4130ea8_1440w.jpg)

MarkText 打开

## 二、相关插件设置

两个插件某种程度上来说是互补的，两者结合，yyds！

### ⭐Custom Attachment location 解决问题 1

49k下载，好用

**核心功能：规范化附件的名字和路径，自定义图片名字，自动转移图片到对应文件夹**

缺点：默认wiki链接且不能转换链接格式

([GitHub - RainCat1998/obsidian-custom-attachment-location: Customize attachment location with variables($filename, $data, etc) like typora.](https://link.zhihu.com/?target=https%3A//github.com/RainCat1998/obsidian-custom-attachment-location))

以下是个人设置

- Location for new attachments: `./assets/${filename}`  
    此处用默认的设置就好  
    
- Generated attachment filename: `${fileName}-${date:YYYYMMDDHHmmss}`  
    可以改成自己想要的格式  
    
- Should rename attachment folder: 打开。  
    如果笔记名字变了，存放对应图片的文件夹名字也会变。  
    
- Should rename attachment files: 打开。  
    如果笔记名字变了，对应图片的名字也会变。  
    
- Duplicate name separator:  
    用什么符号来分隔重复的图片名字，此处是空格  
    
- Should rename collected attachments：第一次批量修改时打开，后续建议关掉  
    
- Should delete orphan attachments: 打开  
    

### ⭐Consistent Attachments and Links 解决问题 2

**核心功能：将图片wiki链接转换成标准的 markdown 相对路径链接，让笔记更有普适性**

53k下载，好用，路径是按照 obsidian 默认来的，联用 Custom Attachment location 插件的路径可以覆盖默认路径。

可以清理空文件夹，将附件移动到对应的文件夹。

缺点：不能自定义附件名字。

([GitHub - dy-sh/obsidian-consistent-attachments-and-links: Obsidian plugin. Move note with attachments.](https://link.zhihu.com/?target=https%3A//github.com/dy-sh/obsidian-consistent-attachments-and-links))

## 三、单个笔记 图片整理流程

一共只需要用到两个插件：Custom Attachment location 和 Consistent Attachments and Links。

1. 先用 Custom Attachment location 插件设置图片的名字和储存位置；然后收集当前笔记的附件。  
Custom Attachment Location: Collect attachments in current note  
此时图片的命名和存放位置已经没问题了。效果如图所示：  

![](https://pic2.zhimg.com/v2-420b30e96e90ff0dc041fe97a78e04b5_1440w.jpg)

示例

  
2. 用 Consistent Attachments and Links 插件将 wiki 图片链接转换为标准的 markdown 链接  
Consistent Attachments and Links: Replace All Wiki Embeds with Markdown Embeds in Current note  
此时变成了这样：叹号+中括号+小括号（小括号内只包含了附件的名字）：  
`![test 1-20250317165307.png](test%201-20250317165307.png)`

3. 最后用 Consistent Attachments and Links 插件将链接转换为相对路径  
Consistent Attachments and Links: Convert All Embed Paths to Relative in Current Note  
最后变成了这样：叹号+中括号+很长的小括号（小括号内包含了附件的路径）：  
`![test 1-20250317165307.png](assets/test%201/test%201-20250317165307.png)`  

**恭喜你！结束了！你的笔记此时非常干净！**

## 四、批量笔记 图片整理流程

与单个文件的转换类似，建议先尝试整理单个笔记确保没问题了再批量整理，批量操作前记得备份。

依然只需要用到两个插件：Custom Attachment location 和 Consistent Attachments and Links。

1. 用 Custom Attachment location 插件设置图片的名字和储存位置；然后收集整个库的附件。  
    `Custom Attachment Location: Collect attachments in entire vault`  
    
2. 用 Consistent Attachments and Links 插件将 wiki 图片链接转换为标准的 markdown 链接  
    `Consistent Attachments and Links: Replace All Wiki Embeds with Markdown Embeds`  
    
3. 最后用 Consistent Attachments and Links 插件将链接转换为相对路径  
    `Consistent Attachments and Links: Convert All Embed Paths to Relative`  
    

## 其他插件

此处推荐一些其他相关插件，非必须

### Clearing Unused Images

96k下载，好用

清除没有被笔记链接的图片

[GitHub - ozntel/oz-clear-unused-images-obsidian: Obsidian plugin to clear the images that are not used in note files anymore](https://link.zhihu.com/?target=https%3A//github.com/ozntel/oz-clear-unused-images-obsidian)

### Image Converter

136k下载，不是很好用

图片大小转换，标注，批处理，笔记改名后附件不会自动改

[GitHub - xRyul/obsidian-image-converter: ⚡️ Convert, compress, resize, annotate, markup, draw, crop, rotate, flip, align images directly in Obsidian. Drag-resize, rename with variables, batch process. WEBP, JPG, PNG, HEIC, TIF.](https://link.zhihu.com/?target=https%3A//github.com/xryul/obsidian-image-converter)

---

folder 设置

location：custom

{notefolder}/assets/{notename}

---

name 设置

{notename}

---

link format 设置

markdown

### Find orphaned files and broken links

177k下载，不是很好用

[GitHub - Vinzent03/find-unlinked-files: Find files, which are nowhere linked, so they are maybe lost in your vault.](https://link.zhihu.com/?target=https%3A//github.com/Vinzent03/find-unlinked-files)

### paste image rename

71k下载

重命名图片的名字

[GitHub - reorx/obsidian-paste-image-rename: Renames pasted images and all the other attachments added to the vault](https://link.zhihu.com/?target=https%3A//github.com/reorx/obsidian-paste-image-rename)

## 其他链接

[MarkDown文件插入图片（绝对\相对路径\调整图像大小位置） - AomanHao - 博客园](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/AomanHao/p/17830051.html)

[浅谈Obsidian中的附件管理 - 经验分享 - Obsidian 中文论坛](https://link.zhihu.com/?target=https%3A//forum-zh.obsidian.md/t/topic/8497)

### 什么是相对路径？

[什么是相对路径？相对路径的具体写法和用法 - 司砚章 - 博客园](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/jgg54335/p/14787211.html)

送礼物

还没有人送礼物，鼓励一下作者吧

[所属专栏 · 2025-03-17 23:10 更新](https://zhuanlan.zhihu.com/c_1880284673852289867)

[![](https://picx.zhimg.com/v2-c5be1695771c4f9b442b5bde56e5e8e0_720w.jpg?source=172ae18b)

Makrdown 笔记经验分享

![](https://pic1.zhimg.com/v2-b8cfe2713504259a46e6c4b5d82ae701_l.jpg?source=172ae18b)

字符序列

2 篇内容 · 16 赞同



](https://zhuanlan.zhihu.com/c_1880284673852289867)订阅

[

最热内容 ·

Obsidian - Front matter



](https://zhuanlan.zhihu.com/c_1880284673852289867)

编辑于 2025-03-18 00:33・广东

[

笔记

](https://www.zhihu.com/topic/19554982)

[

Obsidian

](https://www.zhihu.com/topic/21349840)

[

Markdown

](https://www.zhihu.com/topic/19590742)

​赞同 14​​1 条评论

​分享

​喜欢​收藏​申请转载

​

![](https://pic1.zhimg.com/v2-6c7a496a521ba6ce943bcfbba72b643b_l.jpg?source=32738c0c&needBackground=1)

理性发言，友善互动

  

1 条评论

默认

最新

[![无言](https://picx.zhimg.com/v2-2df91c8d10882afab686407fac09678d_l.jpg?source=06d4cd63)](https://www.zhihu.com/people/6fed2d3bd5d313725643cc199aea956d)

[无言](https://www.zhihu.com/people/6fed2d3bd5d313725643cc199aea956d)

好用！！！！

05-28 · 浙江

​回复​喜欢

关于作者

[![字符序列](https://picx.zhimg.com/v2-b8cfe2713504259a46e6c4b5d82ae701_l.jpg?source=32738c0c&needBackground=1)](https://www.zhihu.com/people/zi-fu-xu-lie-7)

[字符序列](https://www.zhihu.com/people/zi-fu-xu-lie-7)

AFM交流群868224642，一起抱团取暖！

[

回答

**14**

](https://www.zhihu.com/people/zi-fu-xu-lie-7/answers)[

文章

**27**

](https://www.zhihu.com/people/zi-fu-xu-lie-7/posts)[

关注者

**433**

](https://www.zhihu.com/people/zi-fu-xu-lie-7/followers)

​关注​发私信

### 推荐阅读

[

![如何在 Obsidian 实践 P.A.R.A. 与卡片笔记法](https://picx.zhimg.com/v2-b3af6eafa05c572005ad8e5bb1a63611_250x0.jpg?source=172ae18b)

# 如何在 Obsidian 实践 P.A.R.A. 与卡片笔记法

Dluck发表于Dluck...



](https://zhuanlan.zhihu.com/p/553440175)[

![使用Quick Latex和Completr插件在Obsidian中实现快速编辑数学公式](https://picx.zhimg.com/v2-c83f7bb0424ee9f98fd623eab8727152_250x0.jpg?source=172ae18b)

# 使用Quick Latex和Completr插件在Obsidian中实现快速编辑数学公式

Blues



](https://zhuanlan.zhihu.com/p/695723899)[

![Obsidian胎教级教程：颠覆认知的笔记神器，这样用让你的效率翻倍！](https://picx.zhimg.com/v2-ffc40d541309fb4493256ad85f8d1ad6_250x0.jpg?source=172ae18b)

# Obsidian胎教级教程：颠覆认知的笔记神器，这样用让你的效率翻倍！

旷野



](https://zhuanlan.zhihu.com/p/26089182467)[

# Zettelkasten Obsidian：彻底改变你的笔记工作流程

在广阔的知识管理领域中，Zettelkasten 方法已成为一种改变游戏规则的捕捉、组织和连接想法的方法。这篇博文将深入探讨 Zettelkasten 方法，探索强大的 Obsidian 平台，并为您提供改变笔记…

持枪的兔子发表于用工具提升...



](https://zhuanlan.zhihu.com/p/20599133244)